---
title: 'Getting Started with .NET Test Automation'
date: 2024-11-15
permalink: /posts/2024/11/dotnet-test-automation/
tags:
  - automation
  - dotnet
  - selenium
  - testing
excerpt: 'A comprehensive guide to building your first test automation framework with .NET, Selenium, and SpecFlow.'
---

## Introduction

Test automation is essential for modern software development, enabling faster releases and higher quality. In this guide, I'll walk you through building a robust test automation framework using .NET, Selenium WebDriver, and SpecFlow.

## Why .NET for Test Automation?

.NET offers several advantages for test automation:

- **Strong typing** reduces runtime errors
- **Excellent IDE support** with Visual Studio and Rider
- **Rich ecosystem** of testing libraries
- **Cross-platform** with .NET Core
- **Great CI/CD integration** with Azure DevOps and GitHub Actions

## Setting Up Your Project

### Prerequisites

- .NET 6.0 or higher
- Visual Studio 2022 or JetBrains Rider
- Chrome or Edge browser

### Create the Project

```bash
dotnet new nunit -n MyTestFramework
cd MyTestFramework
dotnet add package Selenium.WebDriver
dotnet add package Selenium.WebDriver.ChromeDriver
dotnet add package SpecFlow
dotnet add package SpecFlow.NUnit
```

## Building the Framework

### 1. Page Object Model

The Page Object Model (POM) is a design pattern that creates an object repository for web elements.

```csharp
public class LoginPage
{
    private readonly IWebDriver _driver;
    
    // Locators
    private By UsernameInput => By.Id("username");
    private By PasswordInput => By.Id("password");
    private By LoginButton => By.CssSelector("button[type='submit']");
    
    public LoginPage(IWebDriver driver)
    {
        _driver = driver;
    }
    
    public void Login(string username, string password)
    {
        _driver.FindElement(UsernameInput).SendKeys(username);
        _driver.FindElement(PasswordInput).SendKeys(password);
        _driver.FindElement(LoginButton).Click();
    }
}
```

### 2. WebDriver Factory

Create a factory to manage WebDriver instances:

```csharp
public class WebDriverFactory
{
    public static IWebDriver CreateDriver(string browserType)
    {
        return browserType.ToLower() switch
        {
            "chrome" => new ChromeDriver(),
            "firefox" => new FirefoxDriver(),
            "edge" => new EdgeDriver(),
            _ => throw new ArgumentException($"Browser {browserType} not supported")
        };
    }
}
```

### 3. Base Test Class

```csharp
[TestFixture]
public class BaseTest
{
    protected IWebDriver Driver;
    
    [SetUp]
    public void Setup()
    {
        Driver = WebDriverFactory.CreateDriver("chrome");
        Driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);
        Driver.Manage().Window.Maximize();
    }
    
    [TearDown]
    public void TearDown()
    {
        Driver?.Quit();
    }
}
```

### 4. Writing Your First Test

```csharp
[TestFixture]
public class LoginTests : BaseTest
{
    [Test]
    public void SuccessfulLogin_ShouldNavigateToDashboard()
    {
        // Arrange
        var loginPage = new LoginPage(Driver);
        Driver.Navigate().GoToUrl("https://example.com/login");
        
        // Act
        loginPage.Login("testuser", "password123");
        
        // Assert
        Assert.That(Driver.Url, Does.Contain("/dashboard"));
    }
}
```

## Adding BDD with SpecFlow

### Feature File

```gherkin
Feature: User Login
  As a user
  I want to log into the application
  So that I can access my dashboard

Scenario: Successful login with valid credentials
  Given I am on the login page
  When I enter username "testuser" and password "password123"
  And I click the login button
  Then I should be redirected to the dashboard
```

### Step Definitions

```csharp
[Binding]
public class LoginSteps
{
    private readonly IWebDriver _driver;
    private readonly LoginPage _loginPage;
    
    public LoginSteps(IWebDriver driver)
    {
        _driver = driver;
        _loginPage = new LoginPage(_driver);
    }
    
    [Given(@"I am on the login page")]
    public void GivenIAmOnTheLoginPage()
    {
        _driver.Navigate().GoToUrl("https://example.com/login");
    }
    
    [When(@"I enter username ""(.*)"" and password ""(.*)""")]
    public void WhenIEnterCredentials(string username, string password)
    {
        _loginPage.Login(username, password);
    }
    
    [Then(@"I should be redirected to the dashboard")]
    public void ThenIShouldBeRedirectedToDashboard()
    {
        Assert.That(_driver.Url, Does.Contain("/dashboard"));
    }
}
```

## Best Practices

### 1. Use Explicit Waits

```csharp
var wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(10));
var element = wait.Until(d => d.FindElement(By.Id("dynamic-element")));
```

### 2. Implement Retry Logic

```csharp
public T RetryOnException<T>(Func<T> action, int maxRetries = 3)
{
    for (int i = 0; i < maxRetries; i++)
    {
        try
        {
            return action();
        }
        catch (Exception) when (i < maxRetries - 1)
        {
            Thread.Sleep(1000);
        }
    }
    throw new Exception("Max retries exceeded");
}
```

### 3. Take Screenshots on Failure

```csharp
[TearDown]
public void TearDown()
{
    if (TestContext.CurrentContext.Result.Outcome.Status == TestStatus.Failed)
    {
        var screenshot = ((ITakesScreenshot)Driver).GetScreenshot();
        screenshot.SaveAsFile($"screenshot_{TestContext.CurrentContext.Test.Name}.png");
    }
    Driver?.Quit();
}
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Test Automation

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '6.0.x'
      - name: Run Tests
        run: dotnet test --logger "console;verbosity=detailed"
```

## Conclusion

You now have a solid foundation for .NET test automation! Key takeaways:

- Use Page Object Model for maintainability
- Implement proper wait strategies
- Add BDD with SpecFlow for better collaboration
- Integrate with CI/CD from day one

In future posts, I'll cover advanced topics like parallel execution, reporting, and performance optimization.

## Resources

- [Selenium Documentation](https://www.selenium.dev/documentation/)
- [SpecFlow Documentation](https://docs.specflow.org/)
- [My Test Automation Framework Project](/portfolio/01-automation-framework/)

Happy testing! 🚀
