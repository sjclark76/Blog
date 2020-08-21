---
layout: post
title: ".NET Testing with Docker Databases and Bard.db"
date: 2020-08-21
image: /images/posts/rust-containers.jpg
tags: [.NET Core, Docker, Testing, Bard.Db]
---

## Past Approaches

In the past, writing tests that required a database was always hard and over the years I've employed a number of different approaches.

- **Shared Test Database**. In the early days we would sometimes use a shared database for testing which led to mixed results. On the one hand it was good as you were usually testing against the same database you would be using in Production which would yield accurate test results. But on the other it was bad because it was usually difficult to manage the data in the database required to execute your tests and schema management could be problematic with multiple developers/branches using the same database.

<!--more-->

- **Mocking the Data Access Layer** With the introduction of design patterns such as the [repository pattern](https://martinfowler.com/eaaCatalog/repository.html) it was possible completely ignore the database and just fake it. This made your tests (which were now more unit than integration tests) much easier to write and fast to run. It also had the advantage of making the tests very portable, such that they could be run anywhere with very little effort. However the main problem with these style of tests is that you're really just testing mock database an not ther real thing. Too many times in my career have I seen all my tests passing only for my code to fail when run against a real database.

* **In Memory Database** In recent years EF Core has provided the ability to be able to specify the use of an in memory database and I have seen it's usage become quite popular. However tempting this is it still has the same problems as mocking the data access layer. In my experience it is too easy write a linq query that runs perfectly well against an in memory database but that cannot be evaluated into real SQL.

- **Lightweight Databases** What do I mean by lightweight databases? Small databases such as SQL Server Express localDB. In the past I've had a fair bit of success writing tests against these types of databases. Slower than in memory databases but close enough to the real thing to provide the reasurance that my code was going to work into production.

## Current Approach - Docker

So over the last couple of years with the rise of containers and [Docker](https://www.docker.com/) every new project I have worked on has used a container based database to for testing.

The benefits are that it is usually fairly inexpensive to spin up and throw away a container. Can usually be used in build pipelines and are a very good test of your code as you can often use exactly the same version of database that you would be using in production.

However calling Docker-Compose commands from .NET code can be a little fiddly. To get round the docker-compose problem we have been using the [Docker.DotNet](https://github.com/dotnet/Docker.DotNet) project to drive the Docker API to pull down our docker database images and start and stop our containers.

This still requires a little effort though. To make life a easier I've encapsulated some of this code in a small Nuget package called [Bard.Db](https://docs.bard.net.nz/db/) which makes spinning up and down a database a breeze.

## Example Creating a SQL Server instance.

Create a new instance of Bard.Db.MsSqlDatabase passing in the database name, SA Password, port number, tag version.

```c#
var db = new MsSqlDatabase(
           databaseName: "BardDB_SQL_2017",
           saPassword: "Password1",
           portNumber: "1066",
           tagName: "2017-latest");

```

### Starting up the Database

```c#
var result = db.StartDatabase();

```

This may take a little time the first time as Docker needs to download the image in the background.

### Stopping the Database

To stop the database simply call.

```c#
db.StopDatabase();
```

## Conclusion

Currently the libray only supports MSSQL and PostgreSQL as they are the databases that I uses most in my day to day life. But if there is interest it would be easy enough to support more databases going forward.
