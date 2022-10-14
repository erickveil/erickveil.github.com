---
layout: post
title: How to Create a CRUD Application in ASP.NET 6 Using the Model First Approach
date: 22-10-13
categories: C#, ASP.NET, ASP, .NET, CRUD, Model First, Model, MVC
---

I've been working on CRUD applications since the defacto standard was the LAMP stack and you had to program [prepared statements](https://www.php.net/manual/en/mysqli.quickstart.prepared-statements.php) in PHP. With C#, ASP.NET takes all the fun out of it so you are less likely to mess it up. It's entirely possible to write up a simple CRUD application these days without knowing a lick of SQL. In my opinion, that's good. Anything to get us away from bypassing the compiler by writing code inside a string. If we could also retire unreadable regex right next to the goto statement, that would be perfect.

# First You Need a Model

You have to start somewhere. In the bad-old-days we started with the database, using query language to create the tables and columns. With [Code First](https://learn.microsoft.com/en-us/ef/ef6/modeling/code-first/workflows/new-database) you start... well, you start with the code. The Model specifically. You know what a Model is, because you've been using [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) as a basic priciple in all of your applications, right? Right?

Your model is the code that represents your data in your program. It's just a class that holds values that you've querried from the database. And in this case, it's the Code in "Code First".

## Here's the steps:

- Right-click the Models folder
	- Add > Class
	- Name it something descriptive, like what your table name would be.
- Set public fields - these will translate to database columns
- Use the `vartype?` keyword on your fields:
	- `public vartype? name` - The `?` makes things like int and string nullable.
- Decorate with [data annotations](https://learn.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/models-data/validation-with-the-data-annotation-validators-cs) to set each field as keys, data types, etc.
	- using [System.ComponentModel.DataAnnotations;](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations?view=net-7.0)

# Create the database table from the model automatically with view and controller

Now, it *is* technically possible to do this manually, but why would you? Well, you'd do it at least once so that you know what's going on. We get to use the NuGet script `Add-Migration` to do the work for us. We get this because the manual way is pretty much the same every time you do it. And if you yourself do it 100 times, you're going to write a script that does it for you instead. You do this because like any really good programmer, [you're lazy](https://en.wikipedia.org/wiki/Larry_Wall). But you don't have to, because it's already been done. Still, do the work, then come back.

## Install the Console Script

- Tools > NuGet Package Manager > Package Manager Console
	Install-Package Microsoft.EntityFrameworkCore.Design
	Install-Package Microsoft.EntityFrameworkCore.SqlServer

## Let Visual Studio Set up Scaffolding For You

You don't even have to write **all** the code first. Just the model. With that, you can let Visual Studio write the rest of the code. 

- Right click Controllers folder
	- Add > New Scaffold Item
	- MVC Controller with views, using Entity Framework
		- Select your Model class
		- Data context class > "+"
			- MyModel.Data.MyModelContext
	- Automatically creates "scaffolding"
		- Inserts required package references in the MyModel.csproj project file.
		- Registers the database context in the Program.cs file.
			- Adds Dependency Injection (builder.Services... etc)
				- Look for ``<MyModelContext>``
				- See it passed as an argument in the new controller's 
		- Adds a database connection string to the appsettings.json file.			
		- Creates A movies controller: Controllers/MyController.cs
		- Creates Razor view files for Create, Delete, Details, Edit, and Index pages: 
		- Views/Movies/*.cshtml
		- Creates A database context class: Data/MyContext.cs

## Use Add-Migration to Sync Code With Database

The code is now done. Now is the part that comes second: Making the code talk to the database:

Open up a console from here:

Tools > NuGet Package Manager > Package Manager Console

Enter the following two commands:

```

	Add-Migration InitialCreate
	Update-Database

```

# Conclusion

That's all there is! At this point, you have a model, a view, and a controller. You've even got the database table all set up. You barely had to type any code. Why did you even go to school to learn about programming?

