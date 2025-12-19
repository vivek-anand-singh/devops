# Introduction to GitHub Actions

## What Is GitHub Actions? (In Simple Terms)

**GitHub Actions** lets you **run automated tasks** when something happens in your repo.

**Examples:**

- When **code is pushed** → build the app  
- When a **PR is created** → run tests  

These automated tasks are defined as **workflows** written in **YAML**.

---

## Project Structure (Simple Java App)

We'll use a very basic Java project:

```
java-github-actions-demo/
├── src/
│   └── Main.java
├── pom.xml
└── .github/
    └── workflows/
        └── build.yml
```

---

## Create a Simple Java Application

### src/Main.java

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello from GitHub Actions!");
    }
}
```

---

## Add Maven Configuration

### pom.xml

This tells Maven how to build your Java project.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>github-actions-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
</project>
```

---

## Understanding GitHub Actions Components

Before writing the workflow, understand these terms:

| Term | Meaning |
|------|--------|
| **Workflow** | The automation (YAML file) |
| **Job** | A set of steps |
| **Step** | A single action or command |
| **Runner** | Machine that runs the job (Linux, Windows, Mac) |
| **Action** | Reusable code (e.g. setup Java) |

---

## Create the GitHub Actions Workflow

Create this file: **`.github/workflows/build.yml`**

### Basic Java Build Workflow (Explained Line-by-Line)

| Section | Purpose |
|--------|--------|
| `name: Java Build Workflow` | Name shown in GitHub Actions UI |
| `on: push: branches: [main]` | Trigger workflow when code is pushed to **main** branch |
| `jobs: build:` | Define a job named **build** |
| `runs-on: ubuntu-latest` | Job runs on a Linux virtual machine |
| `steps:` | List of steps inside this job |

#### Step 1: Checkout the Code

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

Downloads your repository code into the runner.

#### Step 2: Setup Java

```yaml
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
```

Installs Java 17 on the runner.

#### Step 3: Build Using Maven

```yaml
- name: Build with Maven
  run: mvn clean package
```

- Runs Maven build  
- Compiles Java code and creates a JAR file  

---

## Full Workflow File (Final)

```yaml
name: Java Build Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Build with Maven
        run: mvn clean package
```

---

## What Happens When You Push Code?

1. **GitHub** detects a push to **main**  
2. A **Linux VM** is created  
3. **Code** is checked out  
4. **Java** is installed  
5. **Maven** builds the project  
6. If build **succeeds** → green check ✓  
7. If build **fails** → red cross ✗ with logs  

---

## Where to See the Output

- Go to **GitHub Repo** → **Actions** tab  
- Click on the **workflow run**  
- Open **"Build with Maven"** step to see logs  

---

## Common Beginner Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong Java version | Match Java in `pom.xml` (e.g. 17) |
| Missing `.github/workflows` | Folder name must be **exact** |
| YAML indentation error | Use **2 spaces**, not tabs |
| Maven not found | Use **setup-java** action before `mvn` |