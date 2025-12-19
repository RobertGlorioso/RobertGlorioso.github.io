<!---
title: "Setting up Selenium and Visual Studio"
date: 2025-12-19
description: "Create app and test in the same environment using Visual Studio and Selenium browser automation"
tags : 
  - C#
  - Visual Studio
  - QA
  - Software Development
  - Selenium
--->

## Setting up Selenium and Visual Studio

1. In Visual Studio set up the project you want to test with Selenium. It should be a web based project so choose the appropriate template for creating a page that Selenium can access and create the test project using one of the testing frameworks. In this post I am using MSTest it is for testing a Blazor Web App.

![new project](/assets/img/set_up_sel/new_project.png)

2. Build some funtionality into the web app - add buttons, set up sql server to retrieve data, etc.
3. In solution explorer, Add a dependency to the test project by right clicking the top level item and hitting "Add > Project Reference" and then select your web app.

![add dependency](/assets/img/set_up_sel/ref_manager.png)

4. Install selenium and move the chromedriver and/or geckodriver executables into an accessible environment path
5. Use the test explorer function on Visual Studio to start running tests IDE
6. If there is no tests showing up, there might be an issue with Visual Studio test explorer, either an update is necessary or it may be necessary to start VS in admin mode.

## Code examples

```html
<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p id="count" role="status">Current Count: @currentCount</p>

<button id="add10" class="btn btn-primary" @onclick="IncrementCount">Click me add 10</button>
<button id="addone" class="btn btn-primary" @onclick="() => currentCount++">Click me add one</button>


```

```csharp
@page "/counter"
@rendermode InteractiveServer

<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p id="count" role="status">Current Count: @currentCount</p>

<button id="add10" class="btn btn-primary" @onclick="IncrementCount">Click me add 10</button>
<button id="addone" class="btn btn-primary" @onclick="() => currentCount++">Click me add one</button>

@code {
    private int currentCount = 0;
    
    [Parameter]
    public int IncrementAmount { get; set; } = 1;

    private void IncrementCount()
    {
        currentCount += IncrementAmount;
    }
}
```