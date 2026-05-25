# UrbanLadder Test Automation — Hybrid Framework

A robust end-to-end test automation framework for [urbanladder.com](https://www.urbanladder.com), built with **Selenium WebDriver**, **TestNG**, and a **Hybrid architecture** that combines the Page Object Model, Keyword-Driven, and Data-Driven approaches.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Framework Architecture](#framework-architecture)
- [Test Flows](#test-flows)
- [Key Classes Explained](#key-classes-explained)
- [Configuration](#configuration)
- [Test Data](#test-data)
- [Reporting](#reporting)
- [Running the Tests](#running-the-tests)
- [Parallel Execution](#parallel-execution)
- [CI/CD Integration](cicd-integration)
- [Logging](#logging)
- [Known Issues & Fixes](known-issues--fixes)

---

## Overview

This project automates three end-to-end user journeys on the Urban Ladder e-commerce website:

1. **Bookshelf filtering** — homepage scroll, navigation, filter application (Storage Type = Open), and product data capture (top 3 with Deal Prices)
2. **Collections exploration** — New Arrivals → Oasis Collection → Living Room → Primary Material = Solid Wood filter → capture all product names into ArrayList
3. **Gift Card form validation** — form filling on woohoo.in, invalid email input, error message capture using getText(), and screenshot

The framework is designed for **cross-browser, parallel execution** on Chrome, Edge, and Firefox, and produces both **ExtentReports (HTML)** and **Allure Reports** for test visibility.

---

## Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Java | 11 / 17 / 21 | Programming language |
| Selenium WebDriver | 4.18.1 | Browser automation |
| WebDriverManager | 5.7.0 | CI/CD fallback for driver management |
| TestNG | 7.9.0 | Test runner, assertions, parallel execution, data providers |
| Apache POI | 5.2.5 | Read test data from Excel (.xlsx) |
| Log4j2 | 2.23.0 | Logging throughout the framework |
| ExtentReports | 5.1.1 | Rich HTML test execution reports |
| Allure TestNG | 2.25.0 | Interactive Allure test reports |
| Maven | 3.x | Build tool and dependency management |

---

## Project Structure

```
HackathonProject_UrbanLadderAutomation_TestTitans/
│
├── pom.xml                              # Maven dependencies and build config
├── config.properties                    # Environment config (URL, waits, paths)
├── testng.xml                           # Test suite config (parallel, browsers)
│
├── .github/workflows/
│   └── test-automation.yml              # GitHub Actions CI/CD pipeline
│
├── testdata/
│   ├── TestData.xlsx                    # Excel test data for Gift Card (Data-Driven)
│   └── Keywords.xlsx                    # Excel keywords for Keyword-Driven execution
│
├── screenshots/                         # Auto-saved after every test with timestamp
├── logs/                                # Log4j writes automation.log here
├── reports/
│   └── ExtentReport.html               # ExtentReports HTML output
├── allure-results/                      # Raw Allure JSON data
├── documents/                           # BRD, Test Plan, Test Strategy, RTM
│
└── src/
    └── test/
        ├── java/com/hackathonproject/
        │   ├── base/
        │   │   └── BaseTest.java            # Browser setup, ThreadLocal driver, lifecycle
        │   ├── keywords/
        │   │   ├── KeywordActions.java      # Reusable action methods (click, type, scroll)
        │   │   └── KeywordEngine.java       # Reads Keywords.xlsx and executes steps
        │   ├── pages/
        │   │   ├── HomePage.java            # Page Object for homepage navigation
        │   │   ├── BookshelvesPage.java     # Page Object for Bookshelves filter flow
        │   │   ├── CollectionsPage.java     # Page Object for New Arrivals flow
        │   │   └── GiftCardPage.java        # Page Object for Gift Card form (woohoo.in)
        │   ├── utils/
        │   │   ├── ConfigReader.java        # Reads config.properties
        │   │   ├── ExcelReader.java         # Reads Excel data via Apache POI
        │   │   ├── ExtentReportManager.java # Singleton ExtentReports manager
        │   │   └── ScreenshotUtil.java      # Screenshot capture with timestamps
        │   ├── listeners/
        │   │   └── TestListener.java        # ITestListener for reports + screenshots
        │   └── tests/
        │       ├── BookshelvesTest.java     # Flow 1: Bookshelves filter tests
        │       ├── CollectionsTest.java     # Flow 2: Collections navigation tests
        │       └── GiftCardTest.java        # Flow 3: Gift Card validation tests
        │
        └── resources/
            ├── config.properties            # Runtime configuration
            └── log4j2.xml                   # Logging configuration
```

---

## Framework Architecture

This is a **Hybrid Framework** combining three design patterns:

### 1. Page Object Model (POM)
Each page of the website has its own class in the `pages/` package. Locators (`@FindBy` annotations) and page-specific actions are encapsulated within that class. All elements are initialized via `PageFactory.initElements()`. Test classes interact with pages through these objects, not directly with the driver.

```
Test Class  →  Page Object  →  KeywordActions  →  Selenium WebDriver
```

### 2. Keyword-Driven Layer
Two components work together:

**KeywordActions.java** — provides a library of reusable, high-level browser interaction methods:
- `click(locator)` / `clickElement(element)` — waits for clickability then clicks
- `jsClick(locator)` / `jsClickElement(element)` — JavaScript click for overlapping elements
- `type(locator, text)` / `typeInElement(element, text)` — clears field and types text
- `getText(locator)` / `getTextFromElement(element)` — waits for visibility and returns trimmed text
- `hoverOver(element)` — mouse hover using Actions class
- `scrollDown(pixels)` — scrolls the page by pixel amount
- `scrollToElement(element)` — brings element into view
- `waitForVisible(locator)` / `waitForElementVisible(element)` — explicit visibility wait
- `isDisplayed(element)` — checks element visibility
- `navigateTo(url)` — navigates to a URL

**KeywordEngine.java** — reads `Keywords.xlsx` and executes each step:
- Reads TestCase, StepNo, Keyword, Locator, Data, Description from Excel
- Auto-detects XPath vs CSS vs ID from the locator string
- Calls the matching `KeywordActions` method for each keyword
- Supports: `navigate`, `click`, `jsclick`, `type`, `gettext`, `scrolldown`

### 3. Data-Driven Layer
`ExcelReader.java` reads test inputs from `testdata/TestData.xlsx` using Apache POI. The `@DataProvider` annotation in `GiftCardTest` feeds Excel data to the test method — each row = one test iteration with different invalid email formats.

---

## Test Flows

### Flow 1 — Bookshelves Filter (`BookshelvesTest`)

| Step | Action |
|---|---|
| 1 | Open urbanladder.com → scroll down → click Bookshelves in Shop by Category |
| 2 | Click Storage Type filter (`aria-label="Storage Type filter"`) → select Open Storage (`dropdown-menu-storage-type`) |
| 3 | Capture first 3 product names (h3 tags) and Deal Prices (not MRP) |
| 4 | Assert: product count ≥ 3, all Deal Prices < ₹15,000, no empty names |
| 5 | Handle duplicate prices (BR_002): if product 4 has same price as product 3, include it |

**Navigation path:** Home → Scroll → Bookshelves → Storage Type = Open → Capture Products

### Flow 2 — Collections Navigation (`CollectionsTest`)

| Step | Action |
|---|---|
| 1 | Click New Arrivals from secondary nav bar |
| 2 | Navigate to Oasis Living Room Collection (direct URL) |
| 3 | Click Primary Material filter (`aria-label="Primary Material filter"`) → select Solid Wood (`dropdown-menu-primary-material`) |
| 4 | Capture all product names into `ArrayList<String>` |
| 5 | Assert: list not empty, page title not empty, each name not empty |

**Navigation path:** Home → New Arrivals → Oasis Living Room → Solid Wood Filter → Capture All Products

### Flow 3 — Gift Card Form Validation (`GiftCardTest`)

| Step | Action |
|---|---|
| 1 | Navigate to urbanladder.woohoo.in Gift Card page |
| 2 | Select Happy Anniversary card design |
| 3 | Enter amount (1500) and quantity (1) |
| 4 | Fill sender details (from Excel) and receiver details with invalid email |
| 5 | Press Enter on receiver email field to submit |
| 6 | Capture error message using `getText()` |
| 7 | Assert: error message not empty, page didn't navigate away |
| 8 | Take screenshot |

**Navigation path:** woohoo.in → Anniversary Card → Fill Form → Invalid Email → Capture Error

**Data-Driven:** Runs twice with different invalid emails from `TestData.xlsx`:
- Row 1: `invalid@` (no domain after @)
- Row 2: `harry.com` (no @ symbol)

---

## Key Classes Explained

### `BaseTest.java`
The parent class for all test classes. Responsibilities:
- Manages `WebDriver` using `ThreadLocal<WebDriver>` — each parallel thread gets its own driver instance
- `@BeforeTest` — opens browser once for all flows in a `<test>` block
- `@AfterMethod` — takes screenshot + navigates back to homepage after every test
- `@AfterTest` — quits browser after all flows complete
- Uses `System.setProperty` for local driver paths, falls back to `WebDriverManager` for CI/CD
- Supports Chrome, Edge, and Firefox via `testng.xml` parameters

### `KeywordActions.java`
The keyword library. Every interaction with the browser goes through this class. All methods include:
- Explicit waits (`WebDriverWait`) before acting
- Two versions: `By` locator and `WebElement` (@FindBy)
- Try-catch with error logging and re-throw on failure
- The `WebDriverWait` timeout is read from `config.properties` (`explicitWait` property)

### `KeywordEngine.java`
Reads `Keywords.xlsx` and executes steps for a given test case name. Auto-resolves locator type:
- Starts with `//` → XPath
- Starts with `#` or `.` → CSS Selector
- Simple word → ID

### `TestListener.java`
Implements TestNG's `ITestListener` interface:
- `onStart` — initializes ExtentReports
- `onTestStart` — creates new ExtentTest entry (ThreadLocal for parallel safety)
- `onTestSuccess` — marks PASS + takes screenshot
- `onTestFailure` — marks FAIL + takes screenshot + logs error + attaches to report
- `onTestSkipped` — marks SKIP + takes screenshot
- `onFinish` — flushes report to HTML, logs pass/fail/skip summary

### `ScreenshotUtil.java`
Captures screenshots with timestamps (including milliseconds `_SSS` to avoid filename collisions during parallel execution). Uses `StandardCopyOption.REPLACE_EXISTING` as additional protection.

### `ConfigReader.java`
Reads all runtime settings from `config.properties`:
- Base URL, browser, timeouts
- All page URLs (bookshelves, new arrivals, oasis, living room, gift card)
- Driver paths (chrome, edge, firefox)
- Test data and keywords file paths

### `ExcelReader.java`
Apache POI-based Excel reader. Handles `STRING`, `NUMERIC`, and `BOOLEAN` cell types. Numeric values are cast to `long` to avoid floating-point suffixes (e.g., `15000.0` → `"15000"`).

---

## Configuration

All runtime settings are in `config.properties`. **Zero hardcoded values in Java files.**

```properties
# Base URL
baseURL=https://www.urbanladder.com
browser=chrome
implicitWait=10
explicitWait=15

# Page URLs
bookshelvesURL=https://www.urbanladder.com/collection/bookshelves?src=top_category_bookshelves
newArrivalsURL=https://www.urbanladder.com/collection/new-arrivals
oasisCollectionURL=https://www.urbanladder.com/page/explore-oasis-collection
livingRoomURL=https://www.urbanladder.com/collection/oasis-living-room-collection
giftCardURL=https://urbanladder.woohoo.in/en-gb/digital/urban-ladder-e-gift-card

# Driver Paths (local)
chromeDriverPath=C:\\path\\to\\chromedriver.exe
edgeDriverPath=C:\\path\\to\\msedgedriver.exe
firefoxDriverPath=C:\\path\\to\\geckodriver.exe

# Test Data
testDataPath=testdata/TestData.xlsx
keywordsPath=testdata/Keywords.xlsx
```

---

## Test Data

### TestData.xlsx — GiftCardData sheet (Data-Driven)

| SenderFirstName | SenderLastName | SenderEmail | SenderPhone | ReceiverFirstName | ReceiverLastName | ReceiverEmail | GiftMessage |
|---|---|---|---|---|---|---|---|
| James | Lee | jameslee@gmail.com | 7639081256 | Rose | Williams | invalid@ | Happy Anniversary! |
| Kate | Stone | katestone@gmail.com | 9567823471 | Harry | Kane | harry.com | Congratulations! |

### Keywords.xlsx — Keywords sheet (Keyword-Driven)

| TestCase | StepNo | Keyword | Locator | Data | Description |
|---|---|---|---|---|---|
| BookshelvesTest | 1 | navigate | | https://www.urbanladder.com/... | Open Bookshelves page |
| BookshelvesTest | 2 | jsClick | //div[@aria-label='Storage Type filter'] | | Click Storage Type |
| GiftCardTest | 1 | navigate | | https://urbanladder.woohoo.in/... | Open Gift Cards |
| GiftCardTest | 2 | type | #denomination | 1500 | Enter amount |

---

## Reporting

### ExtentReports (HTML)
Generated automatically by `TestListener.java` after every test run.
- **Output:** `reports/ExtentReport.html`
- **Includes:** Test steps, pass/fail status, failure stack traces, embedded screenshots

### Allure Reports
Configured via `allure-testng` dependency and `allure-maven` plugin.
- **Output:** `target/site/allure-maven-plugin/index.html`
- **Generate:** `mvn allure:report`

### TestNG Default Report
- **Output:** `test-output/index.html`

---

## Running the Tests

### Prerequisites
- Java 11+ installed and `JAVA_HOME` set
- Chrome / Edge / Firefox browser installed
- `config.properties` populated with correct driver paths

### Run all tests via TestNG (Eclipse)
Right-click `testng.xml` → **Run As → TestNG Suite**

### Run a single flow (Eclipse)
Right-click any test file → **Run As → TestNG Test**

### Run via Maven
```bash
mvn clean test
```

### Run a specific flow via Maven
```bash
mvn clean test -Dtest=BookshelvesTest
mvn clean test -Dtest=CollectionsTest
mvn clean test -Dtest=GiftCardTest
```

---

## Parallel Execution

`testng.xml` is configured for parallel execution across Chrome, Edge, and Firefox:

```xml
<suite name="Urban Ladder Test Suite" parallel="tests" thread-count="3">
    <test name="Chrome Tests">
        <parameter name="browser" value="chrome"/>
        ...
    </test>
    <test name="Edge Tests">
        <parameter name="browser" value="edge"/>
        ...
    </test>
    <test name="Firefox Tests">
        <parameter name="browser" value="firefox"/>
        ...
    </test>
</suite>
```

- `ThreadLocal<WebDriver>` in `BaseTest` ensures each thread has an isolated driver instance
- `ThreadLocal<ExtentTest>` in `ExtentReportManager` ensures each thread logs to its own report entry
- Each browser block runs all three flows → **12 total test executions** (4 methods × 3 browsers)

---

## CI/CD Integration

### Jenkins
- **Job Type:** Maven project
- **Source:** GitHub repository (auto-pulls latest code)
- **Goal:** `clean test -Dsurefire.suiteXmlFiles=testng-jenkins.xml -DforkCount=1`
- **Post-build:** Allure Report generation
- **Driver handling:** Local `System.setProperty` with `WebDriverManager` fallback

### GitHub Actions
- **Workflow file:** `.github/workflows/test-automation.yml`
- **Triggers:** Push to main, pull requests, manual dispatch
- **Runs on:** `windows-latest` with JDK 21 + Chrome

---

## Logging

Logging is handled by **Log4j2**, configured in `src/test/resources/log4j2.xml`.

```java
protected Logger log = LogManager.getLogger(this.getClass());
```

- **Console:** Real-time output during execution
- **File:** `logs/automation.log` — persistent log file
- **Pattern:** `%d{yyyy-MM-dd HH:mm:ss} [%t] %-5level %logger{36} - %msg%n`
- **Thread name** `[%t]` distinguishes parallel execution threads

Log levels used:
- `INFO` — successful actions, navigation steps, assertion results
- `WARN` — fallback actions (e.g., URL navigation when click fails)
- `ERROR` — test failures, element not found, driver errors

---

## Known Issues & Fixes

| Issue | Root Cause | Fix |
|---|---|---|
| Living Room click fails | `@FindBy` element goes stale after Oasis page load | Fallback to direct URL navigation |
| Prices show MRP instead of Deal Price | Page has 4 price elements per product (Deal label, Deal amount, MRP label, MRP amount) | Find "Deal Price" text elements and extract ₹ amount from them |
| Gift Card shows amount error instead of email error | Amount 500 below minimum on woohoo.in | Changed amount to 1500 |
| Screenshot FileAlreadyExistsException | TestListener + AfterMethod take screenshots same second | Added milliseconds `_SSS` to timestamp + `REPLACE_EXISTING` |
| Jenkins ThreadLocal driver null | Parallel threads don't share ThreadLocal | Use `testng-jenkins.xml` with `parallel="false"` + `-DforkCount=1` |
| Allure Maven plugin fails on JDK 21 | API incompatibility in v2.12.0 | Upgraded to v2.13.0, report generates before error |
| Storage Type / Primary Material filter not clicking | Site changed from h2 tags to div with aria-label | Use `aria-label` and dropdown `id` based locators |

---

## Team

**Team Name:** Test Titans — POD2

**Project:** Urban Ladder Test Automation Hackathon

**Date:** May 2026