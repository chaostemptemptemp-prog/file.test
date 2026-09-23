# Jenkins CI/CD Cheatsheet: New Content Only

Covers every question in the PDFs that is **not already** in `devops_cheatsheet.md` or `jenkins_code_and_pipelines.txt`. Each section has Python, `requirements.txt`, Jenkinsfile and Git in separate blocks.

---

## 0. Coverage Map (what was skipped as duplicate)

| PDF source | Where it already lives | Status here |
|---|---|---|
| Two_Stage_Pipeline_Python_Questions (Q1 add, Q2 calculator, Q3 student result) | `devops_cheatsheet.md` §1-3 | Skipped |
| Jenkins_Pipeline_Jobs (5 projects: build/test, parametrized, approval, env+lint, post) | `devops_cheatsheet.md` §4-8 | Skipped |
| Pipeline-task-1 (tasks 1-5: dir, cd, echo, create file, read file) | `.txt` §2.2-2.6 | Skipped |
| Addition-pipeline (`add.py` + parameterized pipeline) | `.txt` §1.1, §2.1 | Skipped |
| Jenkins-1 (Freestyle + SCM + Poll SCM + `addition.py`) | `.txt` §1.2, §3.1-3.3 | Skipped |
| **Jenkins-2 (menu calculator + pytest, Build→Test→Deploy)** | not present | **Section 1** |
| **Milestone example** | not present | **Section 2** |
| **Mail stage example** | not present | **Section 3** |
| **Lab Manual: "5 More Pipeline Projects"** | not present | **Section 4** |
| **Practice Questions Q1-Q7** | not present (Q4 and most of Q2 overlap `.md` P4/P2, so only the new parts are given) | **Section 5** |

---

## 1. Jenkins-2: Menu-Driven Calculator (Build → Test → Deploy)

Job: `Python-Calculator-Pipeline` (Pipeline → *Pipeline script*). No GitHub needed; the pipeline writes its own files.

### 1.1 `calculator.py`
```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

def main():
    while True:
        print("\n--- MENU DRIVEN CALCULATOR ---")
        print("1. Add")
        print("2. Subtract")
        print("3. Multiply")
        print("4. Divide")
        print("5. Exit")
        choice = input("Enter your choice: ")
        if choice == "5":
            print("Exiting...")
            break
        if choice not in ["1", "2", "3", "4"]:
            print("Invalid choice")
            continue
        a = float(input("Enter first number: "))
        b = float(input("Enter second number: "))
        if choice == "1":
            print("Result:", add(a, b))
        elif choice == "2":
            print("Result:", subtract(a, b))
        elif choice == "3":
            print("Result:", multiply(a, b))
        elif choice == "4":
            try:
                print("Result:", divide(a, b))
            except ValueError as e:
                print(e)

if __name__ == "__main__":
    main()
```

### 1.2 `test_calculator.py`
```python
from calculator import add, subtract, multiply, divide

def test_add():
    assert add(10, 5) == 15

def test_subtract():
    assert subtract(10, 5) == 5

def test_multiply():
    assert multiply(10, 5) == 50

def test_divide():
    assert divide(10, 5) == 2

def test_divide_by_zero():
    try:
        divide(10, 0)
        assert False
    except ValueError:
        assert True
```

### 1.3 Install / run commands (no requirements file; installed inline)
```bash
python -m pip install pytest
python -m pytest -v          # use this, not "pytest -v" (PATH issue on Windows)
```
Expected: `5 passed`.

### 1.4 Jenkins Pipeline Script (self-contained)
```groovy
pipeline {
    agent any
    stages {
        stage('Prepare Source') {
            steps {
                writeFile file: 'calculator.py', text: """def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
"""
                writeFile file: 'test_calculator.py', text: """from calculator import add, subtract, multiply, divide

def test_add():
    assert add(10, 5) == 15

def test_subtract():
    assert subtract(10, 5) == 5

def test_multiply():
    assert multiply(10, 5) == 50

def test_divide():
    assert divide(10, 5) == 2

def test_divide_by_zero():
    try:
        divide(10, 0)
        assert False
    except ValueError:
        assert True
"""
            }
        }
        stage('Build') {
            steps {
                echo 'BUILD: Checking Python source'
                bat 'python -m py_compile calculator.py'
            }
        }
        stage('Test') {
            steps {
                echo 'TEST: Running automated tests'
                bat 'python -m pip install pytest'
                bat 'python -m pytest -v'
            }
        }
        stage('Deploy') {
            steps {
                echo 'DEPLOY: Creating deployment folder'
                bat 'if not exist deploy mkdir deploy'
                bat 'copy calculator.py deploy\\calculator.py'
            }
        }
    }
    post {
        success { echo 'BUILD -> TEST -> DEPLOY completed successfully!' }
        failure { echo 'Pipeline failed. Check Console Output.' }
    }
}
```

### 1.5 Failure demo (change `add` inside *Prepare Source*)
```python
def add(a, b):
    return a + b + 1   # intentional error -> test_add FAILED, "1 failed, 4 passed", Deploy skipped
```
Fix by returning `a + b` again → `5 passed`, Deploy runs.

### 1.6 Stage purpose
| Stage | Action | Purpose |
|---|---|---|
| Prepare Source | `writeFile` both `.py` files | Provide source without GitHub |
| Build | `py_compile` | Syntax check |
| Test | `python -m pytest -v` | 5 automated tests |
| Deploy | copy to `deploy\` | Simulated deployment |

---

## 2. Milestone Step Example

### 2.1 `app.py`
```python
def add(a, b):
    return a + b
```

### 2.2 Jenkinsfile
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
                milestone(1)
                echo 'Build stage passed milestone 1'
            }
        }
        stage('Deploy') {
            steps {
                milestone(2)
                echo 'Deploying application...'
            }
        }
    }
}
```

### 2.3 Classroom demo variant (make Build slow)
```groovy
stage('Build') {
    steps {
        bat 'python -m py_compile app.py'
        sleep(time: 15, unit: 'SECONDS')
        milestone(1)
        echo 'Build stage passed milestone 1'
    }
}
```
Click **Build Now** twice quickly. The older build is **aborted** when it reaches a milestone a newer build already passed.

**Rule:** a newer build may always pass a milestone; an older build is aborted if a newer one already passed it. This prevents stale code from deploying after newer code.

---

## 3. Mail Stage Example

### 3.1 `app.py`
```python
def add(a, b):
    return a + b
```

### 3.2 Jenkinsfile
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
                echo 'Build successful: app.py compiled with no syntax errors'
            }
        }
        stage('Send Notification') {
            steps {
                mail to: 'student@example.com',
                     cc: 'instructor@example.com',
                     subject: "Build Notification: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "The build for ${env.JOB_NAME} has completed.\n\nCheck it here: ${env.BUILD_URL}"
            }
        }
    }
}
```

### 3.3 Workaround when no SMTP is configured
```groovy
stage('Send Notification') {
    steps {
        echo "EMAIL WOULD BE SENT -> To: student@example.com | Subject: Build Notification: ${env.JOB_NAME} #${env.BUILD_NUMBER} | URL: ${env.BUILD_URL}"
    }
}
```

**SMTP setup:** Manage Jenkins → System → E-mail Notification (SMTP host, port, credentials). Without it, `mail` fails with a connection error.

**Key point:** `mail` in a normal stage runs only if earlier stages passed. A syntax error in `app.py` fails Build, so no email is sent. `post {}` runs either way.

---

## 4. Lab Manual: "5 More Pipeline Projects" (Windows agent)

All use Stage 1 = Checkout from GitHub. `requirements.txt`: none needed.

### 4.1 Project 1: Parameterized Build (`choice`)
**Files:** `Jenkinsfile` only.
```groovy
pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Select the deployment environment')
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
        stage('Build for Environment') {
            steps {
                echo "Building the application for the ${params.ENVIRONMENT} environment..."
            }
        }
    }
}
```
First run must be **Build Now** (Jenkins has not read the `parameters` block yet). After that, **Build with Parameters** appears.

### 4.2 Project 2: Archive Build Artifacts
`app.py`
```python
with open("report.txt", "w") as f:
    f.write("Application Report\n")
    f.write("Total Users: 120\n")
    f.write("Active Sessions: 45\n")
print("Report generated.")
```
`Jenkinsfile`
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<student-username>/<repo-name>.git'
            }
        }
        stage('Generate Report') {
            steps {
                bat 'python app.py'
            }
        }
        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'report.txt', fingerprint: true
            }
        }
    }
}
```
View at the build page → *Build Artifacts* → `report.txt`. Each build keeps its own copy.

### 4.3 Project 3: Parallel Stages
`frontend_check.py`
```python
import time
print("Running frontend checks...")
time.sleep(3)
print("Frontend checks passed.")
```
`backend_check.py`
```python
import time
print("Running backend checks...")
time.sleep(3)
print("Backend checks passed.")
```
`Jenkinsfile`
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
        stage('Summary') {
            steps {
                echo 'Both frontend and backend checks are complete.'
            }
        }
    }
}
```
Timing: sequential ≈ 6 s, parallel ≈ 3 s (the slower one).

### 4.4 Project 4: Conditional Stage (`when`)
`app.py`
```python
def greet(name):
    return "Hello, " + name
```
`Jenkinsfile`
```groovy
pipeline {
    agent any
    parameters {
        booleanParam(name: 'RUN_EXTRA_CHECK', defaultValue: true, description: 'Run the extra check stage')
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
        stage('Extra Check') {
            when {
                expression { params.RUN_EXTRA_CHECK == true }
            }
            steps {
                echo 'Running extra check: verifying greet() output format...'
                bat 'python -c "from app import greet; print(greet(\'Student\'))"'
            }
        }
    }
}
```
Unchecked → *Extra Check* is skipped (greyed out); its steps never run.

### 4.5 Project 5: Custom Environment Variables
`app.py`
```python
def add(a, b):
    return a + b
```
`Jenkinsfile`
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
        stage('Show App Info') {
            steps {
                echo "Building ${env.APP_NAME}, version ${env.APP_VERSION}"
            }
        }
        stage('Build') {
            steps {
                bat 'python -m py_compile app.py'
                echo "${env.APP_NAME} version ${env.APP_VERSION} compiled successfully."
            }
        }
    }
}
```
Change `APP_VERSION` to `'2.0.0'`, push, rebuild: both stages pick it up.

### 4.6 Git for Section 4
All commands are already in `devops_cheatsheet.md` §9 (`init`, `add .`, `commit -m`, `branch -M main`, `remote add origin`, `push -u origin main`). Nothing new.

---

## 5. Practice Questions (Q1-Q7)

Placeholder repo URL: `https://github.com/<student-username>/<repo-name>.git`

### Q1: multiply/divide, 3 stages + `post`
`app.py`
```python
def multiply(a, b):
    return a * b

def divide(a, b):
    return a / b
```
`test_app.py`
```python
from app import multiply, divide

def test_multiply():
    assert multiply(4, 5) == 20

def test_divide():
    assert divide(10, 2) == 5
```
`requirements.txt`
```text
pytest
```
`Jenkinsfile`
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
Break it: change `multiply` to `return a + b`, then push. `test_multiply` fails, the build goes red, and the **failure** message shows.
```bash
git commit -am "Break multiply"
git push
```

### Q2: find_min / count_odds, parametrized (`-v`)
Jenkinsfile: same as `devops_cheatsheet.md` §5.5 (Checkout → Install Dependencies → `pytest test_app.py -v`). `requirements.txt`: `pytest`.

`app.py`
```python
def find_min(numbers):
    return min(numbers)

def count_odds(numbers):
    return len([n for n in numbers if n % 2 != 0])
```
`test_app.py`
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
**Result:** `6 passed`, from 2 test functions × 3 cases each; every parametrized case counts as its own test.
Wrong-case demo: add `([4, 2, 9], 999)` to `test_find_min`. Only `test_find_min[numbers3-999]` fails, and the other 6 pass.

### Q3: Environment block + approval with custom message
`app.py`
```python
print("Deploying application...")
print("Deployment complete.")
```
`Jenkinsfile`
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
- **Release:** pipeline resumes, runs `app.py`, and finishes SUCCESS.
- **Abort:** build is marked ABORTED, and `app.py` never runs (nothing deployed).

### Q4: Build info + linter
Jenkinsfile and `app.py`: same as `devops_cheatsheet.md` §7. Only the new steps are here.

Compare two builds: `BUILD_NUMBER` changes (#1 → #2); `JOB_NAME` and `WORKSPACE` stay the same.
Linter failure demo: add an unused import at the top of `app.py`.
```python
import os   # F401 'os' imported but unused -> flake8 fails the stage

def greet(name):
    return "Hello, " + name
```

### Q5: `choice` + `booleanParam` + `when`
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
- **Why no *Build with Parameters* on the first build:** Jenkins learns the `parameters` block only after it has run the Jenkinsfile once.
- **Checked:** all 3 stages run. **Unchecked:** *Extra Check* appears greyed out and skipped.

### Q6: Parallel checks + archive reports
`frontend_check.py`
```python
import time
print("Running frontend checks...")
time.sleep(4)
with open("frontend_report.txt", "w") as f:
    f.write("Frontend checks passed.\n")
print("Frontend report written.")
```
`backend_check.py`
```python
import time
print("Running backend checks...")
time.sleep(4)
with open("backend_report.txt", "w") as f:
    f.write("Backend checks passed.\n")
print("Backend report written.")
```
`Jenkinsfile`
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
Timing: parallel ≈ 4 s versus ≈ 8 s sequential. Older builds' artifacts remain on their own build pages after a newer build runs.

### Q7: Milestone + mail notification
`app.py`
```python
def add(a, b):
    return a + b
```
`Jenkinsfile`
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
                // No SMTP? Replace mail with the echo workaround from Section 3.3
                mail to: 'student@example.com',
                     subject: "Build Notification: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: "Build finished. Check it here: ${env.BUILD_URL}"
            }
        }
    }
}
```
- **Two quick Build Now clicks:** the older build waits 15 s, and the newer build reaches `milestone(1)` first. When the older build tries to pass it, Jenkins **aborts it**. It never reaches *Send Notification*, so **it sends no notification**; only the newer build does.
- **Syntax error in `app.py`:** `py_compile` fails the Build stage, the pipeline stops, and *Send Notification* never runs. `mail` sits in a normal stage, not `post`, so no email is sent.

---

## 6. Git: Extra Commands (not in your `.md`/`.txt`)

Everything else (`init`, `add`, `commit -m`, `branch -M`, `remote add`, `push -u`, `log --oneline`, `log --graph`, `config`, `clone`, `checkout -b`) is already in your existing files.
```bash
git commit -am "message"        # stage + commit already-tracked files in one step (Q1 break-and-push)
git diff                        # show unstaged changes before committing
git restore app.py              # discard local edits to app.py (undo a "break")
git revert HEAD                 # new commit that undoes the last commit (safe fix after a pushed break)
git branch                      # list local branches
git checkout main               # switch back to main after the develop demo
```

---

## 7. Extra Jenkinsfile Syntax Quick Reference (new items only)

```groovy
parameters { choice(name: 'ENV', choices: ['dev','staging','prod'], description: '...') }
parameters { booleanParam(name: 'FLAG', defaultValue: true, description: '...') }
environment { APP_NAME = 'GradeBookApp'; APP_VERSION = '1.0.0' }   // custom vars, use ${env.APP_NAME}
when { expression { params.FLAG == true } }                         // conditional stage
parallel { stage('A') { steps { ... } } stage('B') { steps { ... } } }
archiveArtifacts artifacts: 'file1.txt, file2.txt', fingerprint: true
milestone(1)                                                        // ordered checkpoint; older builds get aborted
sleep(time: 15, unit: 'SECONDS')
mail to: '...', cc: '...', subject: '...', body: '...'
writeFile file: 'x.py', text: '...'                                 // create a file from the pipeline
input message: '...', ok: 'Release'                                 // manual approval gate
```
