---
layout: post
title: Testable Web APIs in .Net Core, Part 1
---

Integration testing can be a pain. Beating your head against a mocking framework is probably one of the least rewarding aspects of programming. Configuring faked test environments and maintaining them is even worse. If you want to feel visceral terror, overwriting your prod DBs because you weren't paying attention to config can't be beat. Oh, all of your customers are now "Testo Test", billable at "555 Test Avenue"? There's a better way.

Core has a number of things going for it vis a vis easy testing: drop-in in-memory providers for EF Core, an in-memory webserver, and built-in dependency injection. Once everything is set up, with a few lines of code you can submit calls to your endpoints and collect a result from your contexts. You provide your faked contexts by swapping out a few dependencies and giving your faked server a test startup class. The entire affair is decidedly low maintenance. A fistful of MS packages gets you what you need.

For our purposes, let's assume we have something that emits JSON containing a timestamp, a product, a dollar figure, and an email. Maybe a POS system or online order system.

```json
{
  "now": "2018-01-31T16:20:00.748Z",
  "product": "bluetooth enabled toaster",
  "amount": 17.77,
  "email":"testo@test.test"
}
```

First, we'll log that information into a table in one db for auditing purposes. We'll check a second existing CRM DB to see if we have that email on file, and if we do, we'll write it to a table in that database. The folks in sales can begin drumming up hype on our upcoming line of  blockchain toasters. Spoiler: it's a GPU that you lay a slice of bread on.

First, grab the [.Net Core SDK](https://www.microsoft.com/net/download/). Let's spin up a solution and some projects. We'll stuff data and business logic in a single project for simplicity.

```bash
dotnet new sln -o Toastmaster
cd Toastmaster
dotnet new webapi -o src/Toastmaster
dotnet new classlib -o src/Toastmaster.Domain
dotnet new xunit -o src/Toastmaster.Tests
dotnet sln Toastmaster.sln add src/Toastmaster/Toastmaster.csproj
dotnet sln Toastmaster.sln add src/Toastmaster.Data/Toastmaster.Domain.csproj
dotnet sln Toastmaster.sln add src/Toastmaster.Tests/Toastmaster.Tests.csproj
```

