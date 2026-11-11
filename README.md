# Experiment: Selenium + TestNG + HTML/CSS + GitHub Actions + Docker Hub

## 1. Objective

Create an HTML/CSS login page, automate its validation using Selenium WebDriver and TestNG, push the project to GitHub, and configure GitHub Actions to run the tests and publish a Docker image to Docker Hub only after tests succeed.

**Publishing is automated by GitHub Actions.** Do not use manual `docker login`, `docker tag`, or `docker push` commands to publish.

## 2. Requirements

- JDK 17 and Maven
- Eclipse IDE or another Java IDE
- Google Chrome for local testing
- Git and GitHub account
- Docker Hub account
- Docker Desktop for optional local container testing

Check:

```powershell
java -version
mvn -version
git --version
docker --version
```

## 3. Create Maven project

Create a Maven project named `SeleniumLocalWebTest` with:

here you have use maven-archtype-quickstart

| Field | Value |
|---|---|
| Group Id | `com.example` |
| Artifact Id | `SeleniumLocalWebTest` |
| Version | `1.0-SNAPSHOT` |

Use this layout:

```text
SeleniumLocalWebTest/
├── webpage/login.html
├── src/test/java/LocalLoginTest.java
├── pom.xml
└── testng.xml
```

This follows the lab's root-level `webpage/login.html` layout.

## 4. Configure `pom.xml`

Replace the generated POM with:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>SeleniumLocalWebTest</artifactId>
  <version>1.0-SNAPSHOT</version>
  <properties>
    <maven.compiler.release>17</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <selenium.version>4.25.0</selenium.version>
    <testng.version>7.10.2</testng.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.seleniumhq.selenium</groupId>
      <artifactId>selenium-java</artifactId>
      <version>${selenium.version}</version>
    </dependency>
    <dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>${testng.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.13.0</version>
        <configuration><release>17</release></configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.2.5</version>
        <configuration>
          <suiteXmlFiles>
            <suiteXmlFile>testng.xml</suiteXmlFile>
          </suiteXmlFiles>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

Selenium `4.25.0` and TestNG `7.10.2` are the versions used in the lab material.

## 5. Create the HTML/CSS login page

Create `webpage/login.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Student Login</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 100px;
            background-color: lightblue;
        }
        .login-box {
            width: 300px;
            margin: auto;
            padding: 20px;
            background: white;
            border: 1px solid #cccccc;
            border-radius: 8px;
        }
        input {
            display: block;
            width: 90%;
            margin: 10px auto;
            padding: 8px;
        }
        button { padding: 8px 18px; }
        #message { margin-top: 15px; font-weight: bold; }
    </style>
</head>
<body>
    <div class="login-box">
        <h2>Student Login</h2>
        <input type="text" id="username" placeholder="Username">
        <input type="password" id="password" placeholder="Password">
        <button id="loginButton" onclick="login()">Login</button>
        <p id="message"></p>
    </div>

    <script>
        function login() {
            const username = document.getElementById("username").value;
            const password = document.getElementById("password").value;
            const message = document.getElementById("message");

            if (username === "student" && password === "12345") {
                message.textContent = "Login Successful";
                message.style.color = "green";
            } else {
                message.textContent = "Invalid Username or Password";
                message.style.color = "red";
            }
        }
    </script>
</body>
</html>
```

Demo login credentials: username `student`, password `12345`. These are test credentials only, not production authentication.

## 6. Create Selenium TestNG class

Create `src/test/java/LocalLoginTest.java`:

```java
import java.io.File;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

public class LocalLoginTest {
    private WebDriver driver;

    @BeforeMethod
    public void setUp() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless=new");
        options.addArguments("--no-sandbox");
        options.addArguments("--disable-dev-shm-usage");
        options.addArguments("--window-size=1920,1080");

        driver = new ChromeDriver(options);

        String page = new File("webpage/login.html")
                .getAbsoluteFile()
                .toURI()
                .toString();
        driver.get(page);
    }

    @Test
    public void validLoginTest() {
        driver.findElement(By.id("username")).sendKeys("student");
        driver.findElement(By.id("password")).sendKeys("12345");
        driver.findElement(By.id("loginButton")).click();

        Assert.assertEquals(
            driver.findElement(By.id("message")).getText(),
            "Login Successful"
        );
    }

    @Test
    public void invalidLoginTest() {
        driver.findElement(By.id("username")).sendKeys("wrong");
        driver.findElement(By.id("password")).sendKeys("wrong");
        driver.findElement(By.id("loginButton")).click();

        Assert.assertEquals(
            driver.findElement(By.id("message")).getText(),
            "Invalid Username or Password"
        );
    }

    @Test
    public void backgroundColorTest() {
        String background = driver.findElement(By.tagName("body"))
                .getCssValue("background-color");
        Assert.assertEquals(background, "rgba(173, 216, 230, 1)");
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

The test suite covers valid login, invalid login and the page background colour. The headless Chrome arguments are needed for CI/container execution. Remove `--headless=new` if you want to watch the browser during a local run.

## 7. Create `testng.xml`

Create `testng.xml` in the project root:

```xml
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Selenium Test Suite">
    <test name="Login Page Tests">
        <classes>
            <class name="LocalLoginTest"/>
        </classes>
    </test>
</suite>
```

If you add a Java package declaration to the test class, update the class name in this XML to its fully qualified name.

## 8. Run tests locally

Install Google Chrome, open PowerShell in the project root, and run:

```powershell
mvn clean test
```

Expected: three tests pass with zero failures. Reports are generated under `target/surefire-reports/`.

Selenium Manager locates/downloads a compatible driver. If the browser cannot start, check Chrome installation and network access for Selenium Manager.

## 9. Push the project to GitHub

Create a GitHub repository named `SeleniumLocalWebTest`, then run from the project root:

```powershell
git init
git add .
git commit -m "Create Selenium TestNG login automation"
git branch -M main
git remote add origin https://github.com/YOUR-GITHUB-USERNAME/SeleniumLocalWebTest.git
git push -u origin main
```

Replace the GitHub username placeholder.

## 10. Configure GitHub Actions test workflow

Create `.github/workflows/selenium-tests.yml`:

```yaml
name: Selenium TestNG and Docker Hub Publish

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: read

jobs:
  selenium-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
          cache: maven

      - name: Set up Chrome
        uses: browser-actions/setup-chrome@v1

      - name: Verify Java and Chrome
        run: |
          java -version
          google-chrome --version

      - name: Run Selenium TestNG tests
        run: mvn --batch-mode clean test

  docker-publish:
    needs: selenium-test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - name: Checkout source
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/selenium-testng-automation:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/selenium-testng-automation:${{ github.sha }}
```

The publishing job depends on the test job. If Selenium tests fail, Docker publishing is skipped. Pull requests run tests but do not publish.

## 11. Create Dockerfile

Create a file named exactly `Dockerfile` in the project root:

```dockerfile
FROM maven:3.9.9-eclipse-temurin-17

WORKDIR /app

# Install Google Chrome for headless Selenium tests.
RUN apt-get update     && apt-get install -y --no-install-recommends ca-certificates curl gnupg     && install -d -m 0755 /etc/apt/keyrings     && curl -fsSL https://dl.google.com/linux/linux_signing_key.pub        | gpg --dearmor -o /etc/apt/keyrings/google-chrome.gpg     && chmod a+r /etc/apt/keyrings/google-chrome.gpg     && echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/google-chrome.gpg] https://dl.google.com/linux/chrome/deb/ stable main"        > /etc/apt/sources.list.d/google-chrome.list     && apt-get update     && apt-get install -y --no-install-recommends google-chrome-stable     && rm -rf /var/lib/apt/lists/*

COPY pom.xml .
COPY testng.xml .
COPY webpage ./webpage
COPY src ./src

CMD ["mvn", "--batch-mode", "clean", "test"]
```

This is a test-runner image. Running the container starts headless Chrome, executes the TestNG suite and exits. It does not host the HTML page as a web server.

## 12. Create Docker Hub repository and access token

1. Sign in to Docker Hub.
2. Create a repository named `selenium-testng-automation`.
3. Open **Account Settings → Security → Personal access tokens**.
4. Generate a token with **Read & Write** permission.
5. Copy the token when shown and store it securely.

Do not reuse any example token printed in a lab PDF. Never commit your token to Git.

## 13. Add GitHub repository secrets

Open GitHub repository → **Settings → Secrets and variables → Actions → New repository secret**.

Add:

| Secret name | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Your Docker Hub Read & Write access token |

The workflow consumes these values as `${{ secrets.DOCKERHUB_USERNAME }}` and `${{ secrets.DOCKERHUB_TOKEN }}`. Do not hard-code credentials in YAML.

## 14. Commit and push Docker configuration

```powershell
git add Dockerfile .github/workflows/selenium-tests.yml
git commit -m "Add automated Selenium Docker Hub publishing"
git push origin main
```

## 15. Verify the complete pipeline

### GitHub Actions

1. Open GitHub → your repository → **Actions**.
2. Open the latest `Selenium TestNG and Docker Hub Publish` run.
3. Confirm `selenium-test` succeeds.
4. Confirm `docker-publish` succeeds on a push to `main`.

### Docker Hub

1. Open Docker Hub → `selenium-testng-automation` → **Tags**.
2. Confirm `latest` and the commit-SHA tag are present.

Optional: pull and run the image on another computer:

```powershell
docker pull YOUR-DOCKERHUB-USERNAME/selenium-testng-automation:latest
docker run --rm YOUR-DOCKERHUB-USERNAME/selenium-testng-automation:latest
```

The container should execute the Selenium/TestNG suite.

## 16. Final project structure

```text
SeleniumLocalWebTest/
├── .github/
│   └── workflows/
│       └── selenium-tests.yml
├── Dockerfile
├── pom.xml
├── testng.xml
├── webpage/
│   └── login.html
└── src/
    └── test/
        └── java/
            └── LocalLoginTest.java
```

## Pipeline summary

```text
Create HTML/CSS page and Maven TestNG project
→ Write Selenium tests → Test locally → Push to GitHub
→ GitHub Actions runs Selenium tests
→ If tests pass, Docker build job runs
→ Authenticate using GitHub repository secrets
→ Build and push image to Docker Hub → Verify tags
```

No manual Docker Hub publishing commands and no Kubernetes are included.
