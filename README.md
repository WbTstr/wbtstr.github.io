# WbTstr

<!-- start -->
## Overview
WbTstr is the successor of [WbTstr.Net](https://github.com/mirabeau-nl/WbTstr.Net). It's completely rewritten from scratch with extensibility in mind, using the latest C# / .NET features.
Users of WbTstr are provided with an intuitive API that can be used to completely automate all facets of automated browser-based functional testing, without having to deal with the nitty-gritty details of Selenium.

Noteworthy functionalities include:

- only uses native instructions to control browser instances;
- un-opinoinated towards choice of assertion framework;
- access to HTML of individual page elements;
- read/write access to cookies and execution of custom JavaScript;
- native support for HTTP Basic authentication;
- configurable WebDriver scope (instance per fixture, or per test).

Head over to the [guide](/guide.html) to get started.
<!-- end -->


<!-- start -->
## Contributors
WbTstr is an open-source initaitive from [Mirabeau](https://www.mirabeau.nl/en).

We have the following contributors:

* [@onnovalkering](https://github.com/onnovalkering) 
* [@lazytesting](https://github.com/lazytesting)
* [@rickvanschalkwijk](https://github.com/rickvanschalkwijk)
<!-- end -->

<!-- start -->
## Guide

### Install
WbTstr is available from NuGet. You can either use the NuGet GUI within Visual Studio, or execute the following commands in the Package Manager Console, to install WbTstr.

```powershell
Install-Package WbTstr
Install-Package WbTstr.Drivers.Chrome
```

Be sure to also install at least one driver, we'll assume Chrome for the remainder of this guide.

### Adding fixtures
Below is shown a basic WbTstr fixture. Note that, although WbTstr is un-opinoinated towards choice of assertion framework, we're using NUnit for this guide.

```csharp   
namespace WbTstr.Examples 
{
    [TestFixture]
    [WebDriverConfig(WebDriverType.Chrome)]
    public class Fixture : WbTstrFixture
    {
        [TestCase]
        public void Test()
        {
            I.Open("http://www.example.org")
        }
    }  
}
```

With the `WebDriverConfig` attribute we can specify the type of WebDriver we want to use for this fixture. 
The `I` property inherited from the `WbTstrFixture` class can be used to issue instructions to the WebDriver.

### Configuration

For more advanced configurations, we can add more arguments to the `WebDriverConfig` attribute, as shown below.
The `WebDriverScope` defines the scope of the WebDriver. In this case a new browser instance will be started for each test (the default scope is `WebDriverScope.Fixture`).

```csharp   
namespace WbTstr.Examples 
{
    [TestFixture]
    [WebDriverConfig(WebDriverType.Chrome, WebDriverScope.Test, preset: "preset1")]
    public class Fixture : WbTstrFixture
    {
        [TestCase]
        public void Test()
        {
            I.Open("http://www.example.org")
        }
    }  
}
```

When a configuration preset is provided, additional WebDriver-specific configuration will be loaded from _WbTstr.config_ (make sure to set the `Copy to Output Directory` property to `Copy always` for this file). It is, for example, possible to configure a proxy and/or pass arguments to the WebDriver executable:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <webDriverConfig name="preset1">
    <proxy>
      <httpProxy>http://user:pass@proxy.example.org:1234</httpProxy>
    </proxy>
    <arguments>
      <argument key="start-maximized" />
    </arguments>
  </webDriverConfig>
</configuration>
```

Note that configuration presets can be reused among fixtures. For a complete overview of the configuration options, please consult the [API reference](/api.html).

### Capture elements
WbTstr supports capturing page elements, through CSS selectors, in two ways: `return` or `out` values. This is demonstrated by the two tests below.

```csharp   
[TestCase]
public void ReturnValue()
{
    // Arrange
    string selector = "#element";

    // Act
    I.Open("http://www.example.org")
    var element = I.Find(selector)
    I.Click(element);

    // Assert
    Assert.AreEqual(selector, element.Selector);
}

[TestCase]
public void OutValue()
{
    // Arrange
    string selector = "#element";

    // Act
    I.Open("http://www.example.org")
        .Find(selector, out var element)
        .Click(element);

    // Assert
    Assert.AreEqual(selector, element.Selector);
}
```

Even though the two tests are equivalent, the use of `out` values integrates better with the fluent API of WbTstr. This is the recommended way of capturing elements. 

The same holds for to capturing [page states](/api.html#i-capturepage).
<!-- end -->


<!-- start -->
## API

### Fluent
The WbTstr API is a fluent interface, based on [FluentAutomation](https://github.com/stirno/FluentAutomation). This means that successive instructions can be chained, to create tests that are easy to follow (illustrated below).

```csharp
I.Open("https://www.example.org")
    .Click("#search-icon")
    .Enter("Apples").In("#search-box")
    .Type(Keys.Enter)
    .Find("#result:first-child", var out topResult)
    .Click(topResult);
```

### Instructions
The WbTstr API provides 22 distinct instructions. Selectors must be specified as [CSS selectors](https://www.w3.org/TR/css3-selectors/).

#### I.Append
Append text to an input or textarea.

```csharp
I.Append("Apples").In("#element");

I.Append(12).In("#element");

I.Append("Apples").In(element);
```

#### I.Authenticate
Performs a HTTP basic authentication. First waits until an alert window is present, and then enters the username and password.

```csharp
I.Authenticate("username", "password");

I.Authenticate("username", "password", timeout);
```

#### I.Click
Perform a single click on an element.

```csharp
I.Click("#element");

I.Click(element);
```

#### I.CapturePage
Captures the current state of the page.

```csharp
var page = I.CapturePage()

I.CapturePage(out var page);
```

#### I.DeleteCookie
Deletes a cookie.

```csharp
I.DeleteCookie("name");
```

#### I.DoubleClick
Perform a double click on an element.

```csharp
I.DoubleClick("#element");

I.DoubleClick(element);
```

#### I.Drag
Perform a drag interaction from a point A to a point B. Points can be either elements or a window coordinate (coordinates start at the left-upper).

```csharp
I.Drag("#pointA").To("#pointB");
I.Drag("#pointA").To(pointB);
I.Drag("#pointA").To(100, 100);

I.Drag(pointA).To("#pointB");
I.Drag(pointA).To(pointB);
I.Drag(pointA).To(100, 100);

I.Drag(100, 100).To("#pointB");
I.Drag(100, 100).To(pointB);
I.Drag(100, 00).To(100, 100);
```

#### I.Enter
Enter text in an input or textarea. This will clear the target first.

```csharp
I.Enter("Apples").In("#element");

I.Enter(12).In("#element");

I.Enter("Apples").In(element);
```

#### I.ExecuteJs
Executes a JS script. Supported return types are: IElement, string, long and bool.

```csharp
var body = I.ExecuteJs<IElement>("return window.document.body");

var tagName = I.ExecuteJs<string>("return window.document.body.tagName");

long childElementCount = I.ExecuteJs<long>("return window.document.body.childElementCount");

bool HasAttributes = I.ExecuteJs<bool>("return window.document.body.hasAttributes()");
```

#### I.Find
Capture a single element.

```csharp
var element = I.Find("#element");

I.Find("#element", out var element);
```

#### I.FindMultiple
Capture multiple element.

```csharp
var elements = I.FindMultiple("#elements");

I.FindMultiple("#elements", out var elements);
```

#### I.Focus
Focus on an element.

```csharp
I.Focus("#element");

I.Focus(element);
```

#### I.Hover
Hover on an element.

```csharp
I.Hover("#element");

I.Hover(element);
```

#### I.Open
Navigate to a specific URL.

```csharp
I.Open("https://www.example.org");

I.Open(new Uri("https://www.example.org"));
```

#### I.ResizeWindow
Resizes the window

```csharp
I.ResizeWindow(1920, 1080);
```

#### I.RightClick
Perform a right/context click on an element.

```csharp
I.RightClick("#element");

I.RightClick(element);
```

#### I.MaximizeWindow
Maximizes the window

```csharp
I.MaximizeWindow();
```

#### I.SetCookie
Add a cookie with the domain of the currently loaded website. Deletes cookies with the same name first.

```csharp
I.SetCookie("name", "value");

I.SetCookie(CookieFactory.Create("name", "value"));
```

#### I.TakeScreenshot
Takes a screenshot and saves it to disk (as BMP, JPEG or PNG based on file extension).

```csharp
I.TakeScreenshot("screenshot.png");

I.TakeScreenshot("screenshot.png", "%TEMP%");
```

#### I.Type
Type a text, without specifiying a target.

```csharp
I.Type("Apples");

I.Type(Keys.Enter);
```

#### I.Wait
Wait for a specific period of time.

```csharp
I.Wait(5000);

I.Wait(seconds: 5);

I.Wait(TimeSpan.FromSeconds(5));
```

#### I.WaitUntil
Wait until a predicate returns true.

```csharp
I.WaitUntil(() => I.Find("#element").Displayed);

I.WaitUntil(() => I.Find("#element").Text == "Apple", interval, timeout)
```

<!-- end -->