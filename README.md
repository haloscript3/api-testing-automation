# API Automation Framework - Learning Roadmap & Production-Grade Implementation

> **A Comprehensive Guide for Java/Selenium Developers Transitioning to API Testing**

[![Java](https://img.shields.io/badge/Java-11%2F17-orange.svg)](https://openjdk.java.net/)
[![RestAssured](https://img.shields.io/badge/RestAssured-5.x-green.svg)](https://rest-assured.io/)
[![Cucumber](https://img.shields.io/badge/Cucumber-BDD-brightgreen.svg)](https://cucumber.io/)
[![TestNG](https://img.shields.io/badge/TestNG-7.x-yellow.svg)](https://testng.org/)
[![Allure](https://img.shields.io/badge/Allure-Report-blue.svg)](https://docs.qameta.io/allure/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red.svg)](https://www.jenkins.io/)

---

## Table of Contents

- [Overview](#overview)
- [Target Application](#target-application)
- [Tech Stack](#tech-stack)
- [Phase 1: Architecture & Design Pattern](#phase-1-architecture--design-pattern)
- [Phase 2: The Code (Mental Model)](#phase-2-the-code-mental-model)
- [Phase 3: CI/CD Pipeline (AWS/Linux)](#phase-3-cicd-pipeline-awslinux)
- [Phase 4: Learning Roadmap](#phase-4-learning-roadmap)
- [Quick Start](#quick-start)
- [Contributing](#contributing)

---

## Overview

This repository contains a **production-grade API Automation Framework** designed for developers who have mastered Java and Selenium but are new to API Testing. The framework follows industry best practices and implements a **Hybrid Framework** approach combining Cucumber (BDD), TestNG, and Rest Assured.

### Key Features
- **BDD Approach** - Cucumber for readable, business-friendly test scenarios
- **Robust Assertions** - TestNG with SoftAsserts for comprehensive validation
- **Clean Architecture** - Singleton Pattern, Builder Pattern, DRY Principle
- **POJO-Based Data Handling** - Jackson Databind + Lombok
- **Beautiful Reports** - Allure Reports integrated with Jenkins
- **CI/CD Ready** - Declarative Jenkins Pipeline for AWS Linux

---

## Target Application

**Restful Booker API** - A simple hotel booking API for practicing API testing.

| Endpoint | Description |
|----------|-------------|
| `POST /auth` | Create authentication token |
| `GET /booking` | Get all booking IDs |
| `GET /booking/{id}` | Get specific booking |
| `POST /booking` | Create new booking |
| `PUT /booking/{id}` | Update booking |
| `PATCH /booking/{id}` | Partial update booking |
| `DELETE /booking/{id}` | Delete booking |

**API Documentation:** [https://restful-booker.herokuapp.com/apidoc/index.html](https://restful-booker.herokuapp.com/apidoc/index.html)

---

## Tech Stack

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| **Language** | Java | 11 / 17 | Core programming language |
| **API Library** | Rest Assured | 5.x | HTTP client for API requests |
| **Test Runner** | TestNG | 7.x | Test execution & assertions |
| **BDD Framework** | Cucumber | 7.x | Gherkin-based test scenarios |
| **Serialization** | Jackson Databind | 2.x | JSON to POJO conversion |
| **Code Generator** | Lombok | 1.18.x | Reduce boilerplate code |
| **Reporting** | Allure Report | 2.x | Rich test reports |
| **CI/CD** | Jenkins | Latest | Pipeline automation |
| **Build Tool** | Maven | 3.x | Dependency management |

---

## Phase 1: Architecture & Design Pattern

### Folder Structure (Hybrid Framework)

```
api-test/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── api/
│   │               ├── config/
│   │               │   ├── ConfigReader.java          # Singleton Pattern
│   │               │   └── EndpointConfig.java        # API endpoints constants
│   │               ├── pojo/
│   │               │   ├── Booking.java               # Booking POJO with Lombok
│   │               │   ├── BookingDates.java          # Nested dates POJO
│   │               │   ├── BookingResponse.java       # Response wrapper
│   │               │   └── AuthRequest.java           # Auth credentials POJO
│   │               ├── utils/
│   │               │   ├── RequestSpecBuilder.java    # REST Assured specs
│   │               │   ├── TokenManager.java          # Token handling
│   │               │   └── DataGenerator.java         # Test data factory
│   │               └── clients/
│   │                   ├── AuthClient.java            # Auth API methods
│   │                   └── BookingClient.java         # Booking API methods
│   │
│   └── test/
│       ├── java/
│       │   └── com/
│       │       └── api/
│       │           ├── runner/
│       │           │   └── TestRunner.java            # Cucumber-TestNG runner
│       │           ├── stepdefs/
│       │           │   ├── BookingStepDefs.java       # Booking step definitions
│       │           │   ├── AuthStepDefs.java          # Auth step definitions
│       │           │   └── Hooks.java                 # Before/After hooks
│       │           └── base/
│       │               └── BaseTest.java              # RequestSpecBuilder setup
│       │
│       └── resources/
│           ├── features/
│           │   ├── auth.feature                       # Auth scenarios
│           │   └── booking.feature                    # Booking scenarios
│           ├── config.properties                      # Environment configs
│           ├── testng.xml                             # TestNG suite config
│           └── allure.properties                      # Allure configuration
│
├── pom.xml                                            # Maven dependencies
├── Jenkinsfile                                        # CI/CD pipeline
├── README.md                                          # This file
└── .gitignore                                         # Git ignore rules
```

### Singleton Design Pattern - ConfigReader

The **Singleton Pattern** ensures only one instance of the configuration reader exists throughout the test execution, providing:
- Thread-safe configuration access
- Lazy initialization
- Memory efficiency
- Consistent configuration state

```java
/**
 * Singleton Configuration Reader
 * Loads properties once and provides global access
 */
public class ConfigReader {
    
    private static ConfigReader instance;
    private Properties properties;
    
    // Private constructor prevents external instantiation
    private ConfigReader() {
        loadProperties();
    }
    
    // Double-checked locking for thread safety
    public static ConfigReader getInstance() {
        if (instance == null) {
            synchronized (ConfigReader.class) {
                if (instance == null) {
                    instance = new ConfigReader();
                }
            }
        }
        return instance;
    }
    
    private void loadProperties() {
        properties = new Properties();
        try (InputStream input = getClass().getClassLoader()
                .getResourceAsStream("config.properties")) {
            properties.load(input);
        } catch (IOException e) {
            throw new RuntimeException("Failed to load config.properties", e);
        }
    }
    
    public String getProperty(String key) {
        return properties.getProperty(key);
    }
    
    public String getBaseUrl() {
        return getProperty("base.url");
    }
}
```

### RequestSpecBuilder in BaseTest (DRY Principle)

The **BaseTest** class centralizes REST Assured configuration to avoid code repetition:

```java
/**
 * BaseTest - Centralized REST Assured Configuration
 * Implements DRY principle with RequestSpecBuilder
 */
public class BaseTest {
    
    protected static RequestSpecification requestSpec;
    protected static ResponseSpecification responseSpec;
    
    @BeforeClass
    public void setupSpecifications() {
        // Request Specification - Applied to all requests
        requestSpec = new RequestSpecBuilder()
            .setBaseUri(ConfigReader.getInstance().getBaseUrl())
            .setContentType(ContentType.JSON)
            .addHeader("Accept", "application/json")
            .addFilter(new AllureRestAssured())  // Allure integration
            .log(LogDetail.ALL)
            .build();
        
        // Response Specification - Common validations
        responseSpec = new ResponseSpecBuilder()
            .expectContentType(ContentType.JSON)
            .log(LogDetail.ALL)
            .build();
    }
    
    protected RequestSpecification getAuthenticatedSpec(String token) {
        return given()
            .spec(requestSpec)
            .header("Cookie", "token=" + token);
    }
}
```

---

## Phase 2: The Code (Mental Model)

### Maven Dependencies (pom.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.api</groupId>
    <artifactId>api-automation-framework</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- Dependency Versions -->
        <rest-assured.version>5.4.0</rest-assured.version>
        <testng.version>7.9.0</testng.version>
        <cucumber.version>7.15.0</cucumber.version>
        <jackson.version>2.16.1</jackson.version>
        <lombok.version>1.18.30</lombok.version>
        <allure.version>2.25.0</allure.version>
        <allure-maven.version>2.12.0</allure-maven.version>
    </properties>
    
    <dependencies>
        <!-- REST Assured -->
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>${rest-assured.version}</version>
        </dependency>
        
        <!-- TestNG -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>${testng.version}</version>
        </dependency>
        
        <!-- Cucumber -->
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>${cucumber.version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-testng</artifactId>
            <version>${cucumber.version}</version>
        </dependency>
        
        <!-- Jackson Databind -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        
        <!-- Allure -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>${allure.version}</version>
        </dependency>
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-rest-assured</artifactId>
            <version>${allure.version}</version>
        </dependency>
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-cucumber7-jvm</artifactId>
            <version>${allure.version}</version>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <!-- Maven Surefire Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
                <configuration>
                    <suiteXmlFiles>
                        <suiteXmlFile>src/test/resources/testng.xml</suiteXmlFile>
                    </suiteXmlFiles>
                    <argLine>
                        -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/1.9.21/aspectjweaver-1.9.21.jar"
                    </argLine>
                </configuration>
            </plugin>
            
            <!-- Allure Maven Plugin -->
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>${allure-maven.version}</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### POJO Classes with Lombok

#### Booking.java
```java
package com.api.pojo;

import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.*;

/**
 * Booking POJO - Represents a hotel booking
 * Uses Lombok to eliminate boilerplate code
 */
@Data                           // Generates getters, setters, toString, equals, hashCode
@Builder                        // Enables builder pattern: Booking.builder()...build()
@NoArgsConstructor              // Generates no-args constructor (required for Jackson)
@AllArgsConstructor             // Generates all-args constructor
@JsonInclude(JsonInclude.Include.NON_NULL)  // Exclude null fields from JSON
public class Booking {
    
    @JsonProperty("firstname")
    private String firstName;
    
    @JsonProperty("lastname")
    private String lastName;
    
    @JsonProperty("totalprice")
    private Integer totalPrice;
    
    @JsonProperty("depositpaid")
    private Boolean depositPaid;
    
    @JsonProperty("bookingdates")
    private BookingDates bookingDates;
    
    @JsonProperty("additionalneeds")
    private String additionalNeeds;
}
```

#### BookingDates.java
```java
package com.api.pojo;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.*;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class BookingDates {
    
    @JsonProperty("checkin")
    private String checkIn;
    
    @JsonProperty("checkout")
    private String checkOut;
}
```

#### BookingResponse.java
```java
package com.api.pojo;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

@Data
public class BookingResponse {
    
    @JsonProperty("bookingid")
    private Integer bookingId;
    
    @JsonProperty("booking")
    private Booking booking;
}
```

### BookingStepDefs.java - Step Definitions

```java
package com.api.stepdefs;

import com.api.base.BaseTest;
import com.api.pojo.*;
import com.api.utils.TokenManager;
import io.cucumber.java.en.*;
import io.restassured.response.Response;
import org.testng.asserts.SoftAssert;

import static io.restassured.RestAssured.*;

/**
 * Step Definitions for Booking Feature
 * Demonstrates POST (Create) and GET (Verify) operations
 */
public class BookingStepDefs extends BaseTest {
    
    private Response response;
    private BookingResponse bookingResponse;
    private Booking testBooking;
    private SoftAssert softAssert;
    
    @Given("I have valid booking details")
    public void createValidBookingDetails() {
        BookingDates dates = BookingDates.builder()
            .checkIn("2024-01-15")
            .checkOut("2024-01-20")
            .build();
        
        testBooking = Booking.builder()
            .firstName("James")
            .lastName("Brown")
            .totalPrice(150)
            .depositPaid(true)
            .bookingDates(dates)
            .additionalNeeds("Breakfast")
            .build();
    }
    
    @When("I send a POST request to create a booking")
    public void createBooking() {
        response = given()
            .spec(requestSpec)
            .body(testBooking)
        .when()
            .post("/booking")
        .then()
            .spec(responseSpec)
            .statusCode(200)
            .extract()
            .response();
        
        bookingResponse = response.as(BookingResponse.class);
    }
    
    @Then("the booking should be created successfully")
    public void verifyBookingCreated() {
        softAssert = new SoftAssert();
        
        softAssert.assertNotNull(bookingResponse.getBookingId(), 
            "Booking ID should not be null");
        softAssert.assertEquals(bookingResponse.getBooking().getFirstName(), 
            testBooking.getFirstName(), "First name mismatch");
        softAssert.assertEquals(bookingResponse.getBooking().getLastName(), 
            testBooking.getLastName(), "Last name mismatch");
        softAssert.assertEquals(bookingResponse.getBooking().getTotalPrice(), 
            testBooking.getTotalPrice(), "Total price mismatch");
        
        softAssert.assertAll();
    }
    
    @When("I send a GET request for the created booking")
    public void getBookingById() {
        Integer bookingId = bookingResponse.getBookingId();
        
        response = given()
            .spec(requestSpec)
        .when()
            .get("/booking/" + bookingId)
        .then()
            .spec(responseSpec)
            .statusCode(200)
            .extract()
            .response();
    }
    
    @Then("I should receive the correct booking details")
    public void verifyRetrievedBooking() {
        Booking retrievedBooking = response.as(Booking.class);
        
        softAssert = new SoftAssert();
        softAssert.assertEquals(retrievedBooking.getFirstName(), 
            testBooking.getFirstName());
        softAssert.assertEquals(retrievedBooking.getLastName(), 
            testBooking.getLastName());
        softAssert.assertEquals(retrievedBooking.getTotalPrice(), 
            testBooking.getTotalPrice());
        softAssert.assertAll();
    }
}
```

### Token Extraction & Management

```java
package com.api.utils;

import com.api.config.ConfigReader;
import io.restassured.response.Response;
import static io.restassured.RestAssured.*;

/**
 * TokenManager - Handles Auth Token lifecycle
 * Extracts token from Auth endpoint and provides to subsequent requests
 */
public class TokenManager {
    
    private static String token;
    
    /**
     * Get authentication token (lazy initialization)
     * Token is cached and reused across tests
     */
    public static String getToken() {
        if (token == null) {
            token = generateToken();
        }
        return token;
    }
    
    /**
     * Force token refresh
     */
    public static void refreshToken() {
        token = generateToken();
    }
    
    /**
     * Generate new token from Auth endpoint
     */
    private static String generateToken() {
        ConfigReader config = ConfigReader.getInstance();
        
        String authPayload = """
            {
                "username": "%s",
                "password": "%s"
            }
            """.formatted(
                config.getProperty("auth.username"),
                config.getProperty("auth.password")
            );
        
        Response response = given()
            .baseUri(config.getBaseUrl())
            .contentType("application/json")
            .body(authPayload)
        .when()
            .post("/auth")
        .then()
            .statusCode(200)
            .extract()
            .response();
        
        // Extract token from response using JsonPath
        return response.jsonPath().getString("token");
    }
    
    /**
     * Use token in authenticated requests
     * Example usage in step definitions
     */
    public static Response authenticatedDelete(String endpoint) {
        return given()
            .baseUri(ConfigReader.getInstance().getBaseUrl())
            .contentType("application/json")
            .header("Cookie", "token=" + getToken())
        .when()
            .delete(endpoint);
    }
}
```

---

## Phase 3: CI/CD Pipeline (AWS/Linux)

### Jenkinsfile (Declarative Pipeline)

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
        jdk 'JDK-17'
    }
    
    environment {
        ALLURE_RESULTS = 'target/allure-results'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building project...'
                sh 'mvn clean compile -DskipTests'
            }
        }
        
        stage('Test Execution') {
            steps {
                echo 'Running API Tests...'
                sh '''
                    mvn test \
                        -Dtest.environment=staging \
                        -Dcucumber.filter.tags="@smoke or @regression" \
                        -Dsurefire.suiteXmlFiles=src/test/resources/testng.xml
                '''
            }
            post {
                always {
                    echo 'Archiving test results...'
                    junit allowEmptyResults: true, 
                          testResults: '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Generate Allure Report') {
            steps {
                echo 'Generating Allure Report...'
                allure([
                    includeProperties: false,
                    jdk: '',
                    properties: [],
                    reportBuildPolicy: 'ALWAYS',
                    results: [[path: "${ALLURE_RESULTS}"]]
                ])
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
            // Optional: Send notification
            // emailext body: 'Build failed', subject: 'API Tests Failed', to: 'team@company.com'
        }
    }
}
```

### Shell Commands for Test Execution

```bash
# Basic test execution
mvn clean test

# Run with specific TestNG suite
mvn clean test -DsuiteXmlFile=src/test/resources/testng.xml

# Run with Cucumber tags
mvn clean test -Dcucumber.filter.tags="@smoke"

# Run with environment selection
mvn clean test -Denv=staging

# Full command with Allure report generation
mvn clean test allure:serve

# Generate Allure report without opening browser
mvn clean test allure:report
```

---

## Phase 4: Learning Roadmap

### 4-Week Mastery Plan

```
+-----------------------------------------------------------------------------+
|                        4-WEEK API TESTING MASTERY PLAN                      |
+-----------------------------------------------------------------------------+

WEEK 1: REST Assured Fundamentals
|-- Day 1-2: HTTP Concepts & REST API Basics
|   |-- HTTP Methods (GET, POST, PUT, PATCH, DELETE)
|   |-- Status Codes (2xx, 4xx, 5xx)
|   |-- Headers, Query Params, Path Params
|   +-- JSON Structure & JSONPath
|
|-- Day 3-4: Rest Assured Setup & First Tests
|   |-- Maven project setup
|   |-- given().when().then() syntax
|   |-- Request/Response logging
|   +-- Postman to Rest Assured conversion
|
+-- Day 5-7: REST Assured Advanced
    |-- RequestSpecification & ResponseSpecification
    |-- JsonPath & XmlPath extraction
    |-- Serialization/Deserialization basics
    +-- Practice with Restful Booker API

WEEK 2: Framework Building Blocks
|-- Day 8-9: POJO & Jackson Databind
|   |-- Create POJOs for API payloads
|   |-- @JsonProperty annotations
|   |-- Nested objects handling
|   +-- Lombok integration (@Data, @Builder)
|
|-- Day 10-11: Design Patterns
|   |-- Singleton for ConfigReader
|   |-- Builder pattern with Lombok
|   |-- Factory pattern for test data
|   +-- Page Object Model equivalent for APIs
|
+-- Day 12-14: TestNG Deep Dive
    |-- Assertions (Hard vs Soft)
    |-- @DataProvider for parameterization
    |-- TestNG XML configuration
    +-- Parallel execution setup

WEEK 3: BDD & Framework Integration
|-- Day 15-16: Cucumber Basics
|   |-- Gherkin syntax (Given/When/Then)
|   |-- Feature files structure
|   |-- Step Definitions
|   +-- Hooks (@Before, @After)
|
|-- Day 17-18: Cucumber + TestNG + Rest Assured
|   |-- TestRunner configuration
|   |-- Sharing state between steps (PicoContainer)
|   |-- Tags for test organization
|   +-- Scenario Outline with Examples
|
+-- Day 19-21: Framework Completion
    |-- Implement full booking CRUD tests
    |-- Token management implementation
    |-- Test data management
    +-- Error handling & logging

WEEK 4: CI/CD & Professional Practices
|-- Day 22-23: Allure Reporting
|   |-- Allure annotations (@Step, @Description)
|   |-- Attachments (request/response logs)
|   |-- Allure with Cucumber integration
|   +-- Report customization
|
|-- Day 24-25: Jenkins Pipeline
|   |-- Jenkinsfile syntax
|   |-- Pipeline stages design
|   |-- Allure plugin configuration
|   +-- Scheduled builds & triggers
|
+-- Day 26-28: Production Readiness
    |-- Environment configuration management
    |-- Parallel test execution
    |-- Test retry mechanism
    +-- Code review & documentation
```

### Daily Practice Checklist

| Day | Focus Area | Hands-On Task | Deliverable |
|-----|------------|---------------|-------------|
| 1 | HTTP Basics | Explore Restful Booker in Postman | Collection with all endpoints |
| 3 | Rest Assured | Write first GET/POST test | Basic test class |
| 5 | Specifications | Implement RequestSpecBuilder | BaseTest class |
| 7 | POJO | Create Booking POJO | Complete POJO package |
| 10 | Singleton | Implement ConfigReader | Config management |
| 14 | TestNG | Add SoftAsserts, DataProvider | TestNG suite |
| 17 | Cucumber | First feature file | BDD scenarios |
| 21 | Integration | Full CRUD test suite | Complete framework |
| 24 | Allure | Rich reports | Report integration |
| 28 | Jenkins | Working pipeline | CI/CD deployment |

---

## Quick Start

```bash
# Clone the repository
git clone https://github.com/haloscript3/api-testing-automation.git
cd api-testing-automation

# Run all tests
mvn clean test

# Run smoke tests only
mvn clean test -Dcucumber.filter.tags="@smoke"

# Generate and open Allure report
mvn allure:serve

# Run from Jenkins (shell command)
mvn clean test -DsuiteXmlFile=src/test/resources/testng.xml && mvn allure:report
```

---

## Additional Resources

- [Rest Assured Documentation](https://rest-assured.io/)
- [Cucumber Documentation](https://cucumber.io/docs/cucumber/)
- [TestNG Documentation](https://testng.org/doc/documentation-main.html)
- [Allure Framework](https://docs.qameta.io/allure/)
- [Jackson Databind](https://github.com/FasterXML/jackson-databind)
- [Project Lombok](https://projectlombok.org/)
- [Restful Booker API](https://restful-booker.herokuapp.com/apidoc/index.html)

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  <b>Built for QA Engineers transitioning to API Testing</b>
  <br>
  <i>Master the art of API Automation!</i>
</p>
