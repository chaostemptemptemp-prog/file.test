# Jenkins CI/CD Practice Questions (Q1-Q7): Standalone Cheatsheet

ISWE406L, Windows agent. Replace `<student-username>/<repo-name>` with your own repo.

## 0. One-time setup and common Git flow

```bash
pip install flake8 pytest            # on the Jenkins agent (needed for Q1, Q2, Q4)
python --version                     # verify Python is on PATH
```

Initial push (new repo per question):
```bash
git init
git config --global user.name "YOUR-NAME"
git config --global user.email "YOUR-EMAIL"
git add .
git commit -m "Ver 1.0"
git branch -M main
git remote add origin https://github.com/<student-username>/<repo-name>.git
git push -u origin main
```

Change, break or fix, then rebuild:
```bash
git status
git add .
git commit -m "Ver 1.1"
git push
git log --oneline
```

Jenkins job: New Item → Pipeline → paste the Jenkinsfile in *Pipeline script* (or *Pipeline script from SCM*) → Build Now.

---

## Q1: multiply/divide, 3 stages + `post`

**app.py**
```python
def multiply(a, b):
    return a * b

def divide(a, b):
    return a / b
```

**test_app.py**
```python
from app import multiply, divide

def test_multiply():
    assert multiply(4, 5) == 20

def test_divide():
    assert divide(10, 2) == 5
```

**requirements.txt**
```text
pytest
```

**Jenkinsfile**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
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
    post {
        success {
            echo 'Build succeeded: all unit tests passed.'
        }
        failure {
            echo 'Build failed: a unit test did not pass.'
        }
    }
}
```

**Break it** (in `app.py`), then commit and push:
```python
def multiply(a, b):
    return a + b      # wrong on purpose
```
**Result:** first build passes and prints the success message. After the break, `test_multiply` fails (`assert 9 == 20`), the build goes red, and the **failure** message from `post` shows. `post` runs either way and picks the matching block.

---

## Q2: find_min / count_odds, parametrized tests (`-v`)

**app.py**
```python
def find_min(numbers):
    return min(numbers)

def count_odds(numbers):
    return len([n for n in numbers if n % 2 != 0])
```

**test_app.py**
```python
import pytest
from app import find_min, count_odds

@pytest.mark.parametrize("numbers, expected", [
    ([4, 2, 9], 2),
    ([-5, -1, -9], -9),
    ([7, 7, 7], 7),
])
def test_find_min(numbers, expected):
    assert find_min(numbers) == expected

@pytest.mark.parametrize("numbers, expected", [
    ([1, 2, 3, 4], 2),
    ([2, 4, 6], 0),
    ([1, 3, 5, 7], 4),
])
def test_count_odds(numbers, expected):
    assert count_odds(numbers) == expected
```

**requirements.txt**
```text
pytest
```

**Jenkinsfile**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                bat 'pip install -r requirements.txt'
            }
        }
        stage('Run Unit Tests (Verbose)') {
            steps {
                bat 'pytest test_app.py -v'
            }
        }
    }
}
```

**Console:** `6 passed`, with lines like `test_find_min[numbers0-2] PASSED`.
**Why 6, not 2:** each parametrized case counts as its own test (2 functions × 3 cases = 6).
**Wrong case demo:** add `([4, 2, 9], 999)` to the `find_min` list. Only `test_find_min[numbers3-999]` fails; the other 6 still pass, which shows per-input feedback.

---

## Q3: `environment` block + approval `input` ("Release")

**app.py**
```python
print("Deploying application...")
print("Deployment complete.")
```

**Jenkinsfile**
```groovy
pipeline {
    agent any
    environment {
        APP_NAME = 'GradeBookApp'
        APP_VERSION = '1.0.0'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
            }
        }
        stage('Build') {
            steps {
                bat 'python -m py_compile app.py'
                echo 'Build successful: app.py compiled with no syntax errors'
            }
        }
        stage('Deploy') {
            steps {
                input message: "Approve deployment of ${env.APP_NAME} version ${env.APP_VERSION}?", ok: 'Release'
                bat 'python app.py'
            }
        }
    }
}
```

**Release:** pipeline resumes, runs `app.py`, and ends SUCCESS. **Abort:** build is marked ABORTED, and `app.py` never runs, so nothing is deployed. Automated stages run freely, but the risky step waits for a human.

---

## Q4: Show Build Info + linter (`flake8`)

**app.py** (clean)
```python
def greet(name):
    return "Hello, " + name
```

**Jenkinsfile**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
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

**Run twice:** `BUILD_NUMBER` changes (#1 → #2); `JOB_NAME` and `WORKSPACE` stay the same.
**Linter failure demo** (add unused import, push, rebuild):
```python
import os        # flake8: F401 'os' imported but unused

def greet(name):
    return "Hello, " + name
```
The *Run Linter* stage fails and the build turns red.

---

## Q5: Parameterized pipeline (`choice` + `booleanParam` + `when`)

**Jenkinsfile**
```groovy
pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Select environment')
        booleanParam(name: 'RUN_EXTRA_CHECK', defaultValue: true, description: 'Run the Extra Check stage')
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
            }
        }
        stage('Show Parameter') {
            steps {
                echo "Selected environment: ${params.ENVIRONMENT}"
            }
        }
        stage('Extra Check') {
            when {
                expression { params.RUN_EXTRA_CHECK == true }
            }
            steps {
                echo 'Running extra check...'
            }
        }
    }
}
```

- **No *Build with Parameters* on the first build:** Jenkins only learns about the `parameters` block after it has run the Jenkinsfile once. From the second build on, the button appears.
- **Checked:** all 3 stages run. **Unchecked:** *Extra Check* is greyed out/skipped in the pipeline view. The `when` is evaluated first, so a skipped stage never runs its steps (it does not "run and fail").

---

## Q6: Parallel checks + `archiveArtifacts`

**frontend_check.py**
```python
import time
print("Running frontend checks...")
time.sleep(4)
with open("frontend_report.txt", "w") as f:
    f.write("Frontend checks passed.\n")
print("Frontend report written.")
```

**backend_check.py**
```python
import time
print("Running backend checks...")
time.sleep(4)
with open("backend_report.txt", "w") as f:
    f.write("Backend checks passed.\n")
print("Backend report written.")
```

**Jenkinsfile**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
            }
        }
        stage('Parallel Checks') {
            parallel {
                stage('Frontend Check') {
                    steps {
                        bat 'python frontend_check.py'
                    }
                }
                stage('Backend Check') {
                    steps {
                        bat 'python backend_check.py'
                    }
                }
            }
        }
        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'frontend_report.txt, backend_report.txt', fingerprint: true
            }
        }
    }
}
```

**Timing:** parallel ≈ **4 s** (the slower script); sequential ≈ **8 s** (sum).
**Older artifacts:** change a report's text, push, rebuild, then open the older build page → *Build Artifacts*. Its files still show the old contents because each build keeps its own copy.

---

## Q7: Milestone + mail notification

**app.py**
```python
def add(a, b):
    return a + b
```

**Jenkinsfile**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
            }
        }
        stage('Build') {
            steps {
                bat 'python -m py_compile app.py'
                sleep(time: 15, unit: 'SECONDS')
                milestone(1)
                echo 'Build stage passed milestone 1'
            }
        }
        stage('Send Notification') {
            steps {
                mail to: 'student@example.com',
                     subject: "Build Notification: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Build finished. Check it here: ${env.BUILD_URL}"
            }
        }
    }
}
```

**Echo workaround** (no SMTP): replace the `mail` step with
```groovy
echo "EMAIL WOULD BE SENT -> To: student@example.com | Subject: Build Notification: ${env.JOB_NAME} #${env.BUILD_NUMBER} | URL: ${env.BUILD_URL}"
```
**SMTP setup for real mail:** Manage Jenkins → System → E-mail Notification (host, port, credentials).

**Two quick Build Now clicks:** both builds sleep 15 s, and the newer build reaches `milestone(1)` first. When the older build tries to pass it, Jenkins **aborts the older build**. It never reaches *Send Notification*, so **it sends no notification**; only the newer build does.
**Syntax error in `app.py`:** `py_compile` fails the *Build* stage and the pipeline stops. *Send Notification* is in a normal stage (not `post`), so it only runs if earlier stages passed, and **no email is sent**.

---

## Quick syntax reference

```groovy
post { success { echo '...' }  failure { echo '...' } }             // Q1
bat 'pytest test_app.py -v'                                          // Q2 verbose
environment { APP_NAME = 'X'; APP_VERSION = '1.0.0' }                // Q3, use ${env.APP_NAME}
input message: '...', ok: 'Release'                                  // Q3 approval
echo "${env.BUILD_NUMBER} ${env.JOB_NAME} ${env.WORKSPACE}"          // Q4
parameters { choice(...)  booleanParam(...) }                        // Q5, use ${params.X}
when { expression { params.RUN_EXTRA_CHECK == true } }               // Q5
parallel { stage('A') { steps { ... } }  stage('B') { steps { ... } } }  // Q6
archiveArtifacts artifacts: 'a.txt, b.txt', fingerprint: true        // Q6
milestone(1)   sleep(time: 15, unit: 'SECONDS')                      // Q7
mail to: '...', subject: '...', body: '...'                          // Q7
```
