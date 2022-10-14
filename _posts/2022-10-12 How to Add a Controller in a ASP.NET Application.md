---
layout: post
title: How To Add a Controller in a C# ASP.NET 6.0 Web Application
date: 22-10-14
categories: C#, ASP.NET, ASP, .NET, CRUD, MVC
---

Sometimes you need an extra controller beyond what the [Code First](https://erickveil.github.io/c%23,/asp.net,/asp,/.net,/crud,/model/first,/model,/mvc/2022/10/13/How-to-Create-a-CRUD-Application-in-ASP.NET-6-Using-the-Code-First-Approach.html) automatic approach provided for you. ASP.NET automates that process for you as well! Just follow these steps:

- Right-click Controllers folder
- Add > Controller
	- Empty for a blank
	- Read/write action for some default methods (controller methods == "actions")
	- Or add a View at the same time.
- Actions
	- Can define any Action Method in the controller, not just limited to CRUD.
	- Return Values:
		- Can return a string, which would print in the browser at the address: 
			- https:///localhost/MyController/MyAction
			- URL Routing defined in Program.cs
		- Usually return IActionResult
			- return View(); to display the corresponding View page.
	- Index Action is default if no action is named in the GET URL.
	- Can give any action parameters, their names will be matched with GET values (.../MyAction?paramA=Hello&paramB=World)
		- The ID parameter is automatically matched from the URL, according to the routing:
			- https://localhost/MyController/MyAction/ID
- Assign values to ViewData[] to pass variable values from the Controller to the View.