# Agile Development & DevOps Cheatsheet

**Student Name:** Mark Byju  
**Register No:** 24MIS0497  
**Course:** Agile Development and DevOps (Lab Assessment - 4)  

---

## Quick Reference Table of Projects & Scripts

| # | Question / Project Title | Key Topics Covered | Python Script(s) | Jenkins Features / Plugins |
|---|---|---|---|---|
| **Q1** | Basic Python Application | Command-line arguments, Basic Git operations | `add.py` | Parameterized Build, Batch execution |
| **Q2** | Python Calculator | Arithmetic logic, Conditional output | `calculator.py` | Parameterized Build (`num1`, `num2`) |
| **Q3** | Python Student Result Program | Grade calculation, Formatting output | `student_result.py` | Multi-parameter pipeline (`m1`, `m2`, `m3`) |
| **P1** | Build & Test Pipeline | Unit testing, Dependency management | `app.py`, `test_app.py` | SCM Checkout, `pip install`, `pytest` |
| **P2** | List Utilities Pipeline | Parametrized unit testing | `app.py`, `test_app.py` | SCM Checkout, Parametrized Pytest |
| **P3** | Manual Approval Gate Deploy Pipeline | Syntax compilation, Manual gate | `app.py` | `py_compile`, `input` stage approval |
| **P4** | Environment Variables & Linting | Code quality, Environment metrics | `app.py` | `env` variables (`BUILD_NUMBER`, `WORKSPACE`), `flake8` |
| **P5** | Post-Build Pipeline | Error handling, Build notifications | `app.py` | `post` block (`success`, `failure`) |

---

## Section 1: Question 1 - Basic Python Application

### 1.1 Python Application (`add.py`)
```python
import sys

def main():
    num1 = float(sys.argv[1]) if len(sys.argv) > 1 else 3.0
    num2 = float(sys.argv[2]) if len(sys.argv) > 2 else 18.0
    result = num1 + num2
    print(f"Number 1 (num1): {num1}")
    print(f"Number 2 (num2): {num2}")
    print(f"The sum is: {result}")

if __name__ == "__main__":
    main()
```

### 1.2 Git Workflow & Commands
```bash
# Initialize local repository
git init

# Configure Global User Credentials
git config --global user.name "chaostemptesptemp-prog"
git config --global user.email "chaos.temp.temp.temp@gmail.com"

# Check status and commit log
git status
git log --oneline

# Add Remote Origin and Push Initial Code
git remote add origin https://github.com/chaostemptemptemp-prog/Assessment-4.git
git branch -M main
git push -u origin main

# Staging, Committing, and Pushing Updates
git add .
git commit -m "Version 1.1"
git push
git log --oneline
```

### 1.3 Jenkins Declarative Pipeline
```groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'num1', defaultValue: '10', description: 'First Number')
        string(name: 'num2', defaultValue: '20', description: 'Second Number')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Run Python Script') {
            steps {
                bat "python add.py ${params.num1} ${params.num2}"
            }
        }
    }
}
```

---

## Section 2: Question 2 - Python Calculator

### 2.1 Python Application (`calculator.py`)
```python
import sys

def main():
    num1 = float(sys.argv[1]) if len(sys.argv) > 1 else 10.0
    num2 = float(sys.argv[2]) if len(sys.argv) > 2 else 5.0
    
    print(f"Calculator Outputs for Inputs: {num1} and {num2}")
    print(f"Addition: {num1} + {num2} = {num1 + num2}")
    print(f"Subtraction: {num1} - {num2} = {num1 - num2}")
    print(f"Multiplication: {num1} * {num2} = {num1 * num2}")
    
    if num2 != 0:
        print(f"Division: {num1} / {num2} = {num1 / num2}")
    else:
        print("Division: Error (Cannot divide by zero)")

if __name__ == "__main__":
    main()
```

### 2.2 Git Workflow & Commands
```bash
# Check repository status
git status

# Stage updated files and calculator script
git add .
git log --oneline

# Commit and Push Version 1.2
git commit -m "Version 1.2"
git push
```

### 2.3 Jenkins Declarative Pipeline
```groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'num1', defaultValue: '10', description: 'First number')
        string(name: 'num2', defaultValue: '5', description: 'Second number')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Run Calculator') {
            steps {
                bat "python calculator.py ${params.num1} ${params.num2}"
            }
        }
    }
}
```

---

## Section 3: Question 3 - Python Student Result Program

### 3.1 Python Application (`student_result.py`)
```python
import sys

def main():
    m1 = float(sys.argv[1]) if len(sys.argv) > 1 else 60.0
    m2 = float(sys.argv[2]) if len(sys.argv) > 2 else 70.0
    m3 = float(sys.argv[3]) if len(sys.argv) > 3 else 80.0
    
    total = m1 + m2 + m3
    average = total / 3.0
    status = "PASS" if average >= 50.0 else "FAIL"
    
    print("Student Result Analysis")
    print(f"Subject 1 Marks: {m1}")
    print(f"Subject 2 Marks: {m2}")
    print(f"Subject 3 Marks: {m3}")
    print(f"Total Marks: {total}")
    print(f"Average Marks: {average:.2f}")
    print(f"Final Status: {status}")

if __name__ == "__main__":
    main()
```

### 3.2 Git Workflow & Commands
```bash
# View commit history graph
git log --graph --oneline

# Check status, add files, and review staging
git status
git add .
git status

# Commit Version 1.3
git commit -m "Version 1.3"
git log --oneline

# Push to Remote Repository
git push -u origin main
git log --oneline
```

### 3.3 Jenkins Declarative Pipeline
```groovy
pipeline {
    agent any
    
    parameters {
        string(name: 'm1', defaultValue: '60', description: 'Marks for Subject 1')
        string(name: 'm2', defaultValue: '70', description: 'Marks for Subject 2')
        string(name: 'm3', defaultValue: '80', description: 'Marks for Subject 3')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Run Student Analysis') {
            steps {
                bat "python student_result.py ${params.m1} ${params.m2} ${params.m3}"
            }
        }
    }
}
```

---

## Section 4: PROJECT 1 - Build & Test Pipeline

### 4.1 Application Code (`app.py`)
```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b
```

### 4.2 Test Suite (`test_app.py`)
```python
from app import add, subtract

def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 3) == 2
```

### 4.3 Requirements File (`requirements.txt`)
```text
pytest
```

### 4.4 Git Workflow & Commands
```bash
# Initialize repository
git init
git status

# Stage untracked project files
git add .
git status

# Commit initial release
git commit -m "Ver 1.0"

# Set remote tracking branch and push
git push -u origin main
git log --oneline
```

### 4.5 Jenkinsfile Script
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/chaostemptemptemp-prog/1.0-Assessment.git']]
                )
            }
        }
        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }
        stage('Run Unit Tests') {
            steps {
                bat 'pytest test_app.py'
            }
        }
    }
}
```

---

## Section 5: PROJECT 2 - List Utilities Pipeline (Parametrized Tests)

### 5.1 Application Code (`app.py`)
```python
def find_max(numbers):
    return max(numbers)

def count_evens(numbers):
    return len([n for n in numbers if n % 2 == 0])
```

### 5.2 Parametrized Test Suite (`test_app.py`)
```python
import pytest
from app import find_max, count_evens

@pytest.mark.parametrize("numbers, expected", [
    ([1, 5, 3], 5),
    ([-10, -2, -7], -2),
    ([4, 4, 4], 4)
])
def test_find_max(numbers, expected):
    assert find_max(numbers) == expected

@pytest.mark.parametrize("numbers, expected", [
    ([1, 2, 3, 4], 2),
    ([1, 3, 5], 0),
    ([2, 4, 6, 8], 4)
])
def test_count_evens(numbers, expected):
    assert count_evens(numbers) == expected
```

### 5.3 Requirements File (`requirements.txt`)
```text
pytest
```

### 5.4 Git Workflow & Commands
```bash
# Initialize Git Repository
git init
git status
git add .
git commit -m "Ver 1.0"
git log --oneline

# Track modifications and update commit
git status
git add .
git commit -m "Ver 1.1"
git log --oneline

# Set configuration and rename branch
git config --global user.name "chaostemptemptemp-prog"
git config --global user.email "chaos.temp.temp.temp@gmail.com"
git branch -M main

# Add remote origin and push initial main branch
git remote add origin https://github.com/chaostemptemptemp-prog/-1.1-Assessment-.git
git push origin main

# Commit & push final changes
git add .
git commit -m "Ver 1.2"
git push -u origin main
```

### 5.5 Jenkinsfile Script
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/chaostemptemptemp-prog/-1.1-Assessment-.git']]
                )
            }
        }
        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }
        stage('Run Unit Tests') {
            steps {
                bat 'pytest test_app.py -v'
            }
        }
    }
}
```

---

## Section 6: PROJECT 3 - Manual Approval Gate Deploy Pipeline

### 6.1 Application Code (`app.py`)
```python
def main():
    print("Deploying application version 1.0...")
    print("Deployment complete.")

if __name__ == "__main__":
    main()
```

### 6.2 Git Workflow & Commands
```bash
# Initialize local repo
git init
git status

# Add and commit project files
git add .
git commit -m "Ver 1.0"

# Connect remote repository and push main
git remote add origin https://github.com/chaostemptemptemp-prog/1.3-Assessment.git
git branch -M main
git push -u origin main
```

### 6.3 Jenkinsfile Script (with Interactive Approval Input)
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/chaostemptemptemp-prog/1.3-Assessment.git'
            }
        }
        stage('Build') {
            steps {
                bat 'python -m py_compile app.py'
                echo "Build successful: app.py compiled with no syntax errors"
            }
        }
        stage('Deploy') {
            steps {
                input message: 'Approve deployment to production?', ok: 'Deploy'
                bat 'python app.py'
            }
        }
    }
}
```

---

## Section 7: PROJECT 4 - Environment Variables & Linting Pipeline

### 7.1 Application Code (`app.py`)
```python
def greet(name):
    return "Hello, " + name
```

### 7.2 Git Workflow & Commands
```bash
# Initialize Git repository
git init
git status

# Stage & Commit files
git add .
git commit -m "ver 1.0"

# Add Remote and Push to Main
git remote add origin https://github.com/chaostemptemptemp-prog/1.4.git
git branch -M main
git push -u origin main
```

### 7.3 Jenkinsfile Script (Inspecting Env Variables & Linting)
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/chaostemptemptemp-prog/1.4.git']]
                )
            }
        }
        stage('Show Build Info') {
            steps {
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Job Name: ${env.JOB_NAME}"
                echo "Workspace: ${env.WORKSPACE}"
            }
        }
        stage('Run Linter') {
            steps {
                bat 'flake8 app.py'
            }
        }
    }
}
```

---

## Section 8: PROJECT 5 - Post-Build Success/Failure Pipeline

### 8.1 Application Code (`app.py`)
```python
def multiply(a, b):
    return a * b

print(multiply(4, 5))
```

### 8.2 Git Workflow & Commands
```bash
# Reinitialize Git repository
git init
git status

# Configure Remote Origin
git remote add origin https://github.com/chaostemptemptemp-prog/Proj-5.git

# Stage and Commit
git add .
git commit -m "Ver 1.0"

# Branch & Push
git branch -M main
git push -u origin main
git log --oneline
```

### 8.3 Jenkinsfile Script (Post-Build Hooks)
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scmGit(
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[url: 'https://github.com/chaostemptemptemp-prog/Proj-5.git']]
                )
            }
        }
        stage('Compile Check') {
            steps {
                bat 'python -m py_compile app.py'
            }
        }
    }
    
    post {
        success {
            echo "Build succeeded: app.py has no syntax errors."
        }
        failure {
            echo "Build failed: check app.py for syntax errors."
        }
    }
}
```

---

## Section 9: Common Git Command Reference Summary

```bash
# Repository Initialization & Setup
git init                                 # Initialize empty Git repository
git config --global user.name "NAME"     # Set global username
git config --global user.email "EMAIL"   # Set global email address

# Remote Synchronization
git remote add origin <URL>              # Connect local repo to remote URL
git branch -M main                       # Rename active branch to 'main'
git push -u origin main                  # Initial push & set tracking upstream
git push                                 # Push commits to tracked remote branch

# Staging & Committing
git status                               # View working directory status
git add .                                # Stage all changed/untracked files
git commit -m "Commit Message"           # Commit staged changes with message

# Branching & History Inspection
git log --oneline                        # Compact single-line commit history
git log --graph --oneline                # Graphical visualization of commit history
```

---

## Section 10: Jenkins Declarative Pipeline Cheat Sheet

```groovy
// Basic Jenkinsfile Layout
pipeline {
    agent any                            // Execution agent node
    
    parameters {                         // Parameterized job inputs
        string(name: 'PARAM_NAME', defaultValue: 'VAL', description: 'Desc')
    }
    
    stages {
        stage('Stage Name') {
            steps {
                // Shell / Batch commands
                bat 'python app.py'      // Windows Batch Execution
                sh 'python3 app.py'      // Linux/macOS Shell Execution
                
                // Echo & Interpolation
                echo "Value: ${params.PARAM_NAME}"
                echo "Build Number: ${env.BUILD_NUMBER}"
                
                // Manual Approval Input Gate
                input message: 'Approve Deployment?', ok: 'Deploy'
            }
        }
    }
    
    post {                               // Post-execution triggers
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
```
