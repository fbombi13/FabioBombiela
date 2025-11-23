---
title: "Novice: Modern .NET 10 Test Automation Framework"
excerpt: "A robust, modular test automation framework built with .NET 10, Selenium 4, and NUnit. Features structured logging with Serilog, environment-specific configurations, and a clean architecture design."
collection: portfolio
date: 2024-11-23
header:
  teaser: /images/portfolio/dotnet-taf-teaser.png
---

**Novice** is a state-of-the-art test automation framework designed to demonstrate the power and flexibility of modern .NET for QA automation. Built on the bleeding edge **.NET 10**, it integrates industry-standard tools like **Selenium WebDriver**, **NUnit**, and **Serilog** into a cohesive, maintainable solution.

## 🚀 Key Features

*   **Bleeding Edge Tech Stack**: Leverages **.NET 10** and **C# 12+** features for high performance and cleaner code.
*   **Modular Architecture**: strict separation of concerns:
    *   **Core**: Infrastructure, Logging, Configuration.
    *   **Business**: Page Objects, Business Logic, UI Components.
    *   **Tests**: NUnit test cases (Unit, Integration, E2E).
*   **Robust Configuration System**: Uses `Microsoft.Extensions.Configuration` to support environment-specific settings (`appsettings.Test.json`, `appsettings.Development.json`), allowing seamless switching between local, dev, and CI/CD environments.
*   **Structured Logging**: Integrated **Serilog** configuration that outputs to both Console and File with different verbosity levels based on the environment.
*   **Cross-Browser Support**: Pre-configured for **Edge**, **Chrome**, and **Firefox** via WebDriver.
*   **CI/CD Ready**: Designed to be easily integrated into pipelines with headless execution modes and XML test results.

## 🛠️ Technical Stack

| Component | Technology | Version |
|-----------|------------|---------|
| **Language** | C# / .NET | 10.0 |
| **Web Driver** | Selenium | 4.38.0 |
| **Test Runner** | NUnit | 4.3.2 |
| **Logging** | Serilog | 4.3.0 |
| **Config** | MS Extensions | 10.0.0 |

## 🏗️ Architecture Overview

The framework follows a clean, layered architecture to ensure scalability:

```csharp
Novice/
├── Core/                    # The "Engine"
│   ├── LoggerConfig.cs      # Serilog factory
│   └── ConfigurationHelper.cs # Singleton config access
├── Business/                # The "Model"
│   ├── Pages/               # Page Object Models
│   └── Components/          # Reusable UI widgets
├── Tests/                   # The "Execution"
│   └── Functional/          # E2E Test Scenarios
└── appsettings.json         # Global Config
```

## 💻 Code Example

Here's how a typical test looks in **Novice**, demonstrating the clean syntax and built-in logging:

```csharp
[Test]
public void UserCanLoginSuccessfully()
{
    _logger.Information("Starting Login Test");

    // Arrange
    var loginPage = new LoginPage(_driver);
    
    // Act
    loginPage.NavigateTo();
    loginPage.Login("user@example.com", "securePassword");

    // Assert
    Assert.That(loginPage.IsDashboardVisible(), Is.True, "Dashboard should be visible after login");
    
    _logger.Information("Login Test Completed Successfully");
}
```

## 🔗 Project Links

*   **Source Code**: [View on GitHub](https://github.com/fbombi13/AutomationAcademy-Dotnet-TAF)
*   **Documentation**: [Read the Docs](https://github.com/fbombi13/AutomationAcademy-Dotnet-TAF/blob/main/README.md)
