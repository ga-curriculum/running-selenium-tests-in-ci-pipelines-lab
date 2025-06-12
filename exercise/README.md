<h1>
  <span class="headline">Running Selenium Tests in CI Pipelines Lab</span>
  <span class="subhead">Exercise</span>
</h1>

## Overview

In this lab, you’ll extend your CI pipeline by automating **browser-based tests** using **Selenium** inside **GitHub Actions**. You’ll simulate real user flows like filling a form or checking UI feedback, then automate them headlessly using Chromium inside CI.

## Learning Goals

By the end of this lab, you will be able to:

- Write basic Selenium tests that interact with a webpage.
- Serve a FastAPI app locally for testing.
- Run Selenium tests in headless mode.
- Automate the entire flow using GitHub Actions.

## Step 1: Preview the App

Start your FastAPI app locally:

```bash
uvicorn app.main:app --reload
```

Then open the browser to view the form at [http://localhost:8000](http://localhost:8000).

## Step 2: Add API Tests

Create a file in `tests/` called `test_api.py` and paste in the following code:

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

# Test 1
def test_signup_successful():
    payload = {"name": "Alice Example", "email": "alice@example.com"}
    response = client.post("/signup", data=payload)
    assert response.status_code == 200
    assert response.json() == {
        "message": "Thanks for subscribing, Alice Example!"}

# Test 2
def test_signup_blank_name():
    payload = {"name": "", "email": "blank@name.com"}
    response = client.post("/signup", data=payload)
    assert response.status_code == 422

# Test 3
def test_signup_missing_email():
    payload = {"name": "Missing Email"}
    response = client.post("/signup", data=payload)
    assert response.status_code == 422
```

Run these API tests locally using this command:

```bash
PYTHONPATH=. pytest tests/test_api.py
```

## Step 3: Add Selenium UI Tests

Create a file in `tests/` called `test_ui_selenium.py`.

Paste in the following code:

```python
import os
from selenium import webdriver
from selenium.webdriver.common.by import By
import time

def setup_browser():
    """Configure browser differently for local vs CI (GitHub Actions)"""
    options = webdriver.ChromeOptions()

    if os.getenv("CI"):  # Running in GitHub Actions
        options.add_argument("--headless")
        options.add_argument("--no-sandbox")
        options.add_argument("--disable-dev-shm-usage")
        return webdriver.Chrome(options=options)
    else:
        # Local (assumes chromedriver is on PATH)
        return webdriver.Chrome()

# Test 1
def test_invalid_email_shows_error_message():
    driver = setup_browser()
    try:
        driver.get("http://localhost:8000/signup")
        # TODO: Enter valid name, invalid email
        # TODO: Submit form
        # TODO: Confirm success message does NOT appear
    finally:
        driver.quit()

# Test 2
def test_blank_password_prevents_submit():
    driver = setup_browser()
    try:
        driver.get("http://localhost:8000/signup")
        # TODO: Leave email blank
        # TODO: Submit form
        # TODO: Check form validation blocks submit
    finally:
        driver.quit()

# Test 3
def test_successful_signup_shows_thank_you():
    driver = setup_browser()
    try:
        driver.get("http://localhost:8000/signup")
        # TODO: Fill form with valid data
        # TODO: Submit
        # TODO: Check thank-you message with name
    finally:
        driver.quit()

# Test 4
def test_form_resets_after_submit():
    driver = setup_browser()
    try:
        driver.get("http://localhost:8000/signup")
        # TODO: Submit valid data
        # TODO: Confirm fields reset
    finally:
        driver.quit()
```

Use your new selenium skills to complete the `TODOs` in the test file.

Fix any broken selectors or paths.

Run these Selenium tests locally using this command:

```bash
PYTHONPATH=. pytest tests/test_ui_selenium.py
```

## Step 4: Add CI Workflow

Create a file: `.github/workflows/ci.yml`

```yaml
name: Python & Selenium CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Install Chromium and Chromedriver
        run: |
          sudo apt-get update
          sudo apt-get install -y chromium-browser chromium-chromedriver
          # Only create symlink if it doesn't exist
          if [ ! -f /usr/bin/chromedriver ]; then
            sudo ln -s /usr/lib/chromium-browser/chromedriver /usr/bin/chromedriver
          fi

      - name: Start FastAPI app
        run: uvicorn app.main:app --host 0.0.0.0 --port 8000 &
        shell: bash

      - name: Wait for server to be ready
        run: |
          sleep 5
          curl --retry 5 --retry-connrefused --retry-delay 2 http://localhost:8000/signup

      - name: Run tests
        env:
          PATH: /usr/lib/chromium-browser:$PATH
        run: pytest
```

## Step 5: Confirm File Structure

Your application structure should look like this before pushing to github.

```
running-selenium-tests-in-ci-pipeline-lab/
├── app/
|   ├── __init__.py
│   ├── main.py
│   └── static/
│       └── newsletter_signup.html
├── tests/
|   ├── __init__.py
│   ├── test_api.py
│   └── test_ui_selenium.py
├── requirements.txt
└── .github/workflows/
    └── ci.yml
```

## Step 6: Push and Observe CI

Make any change and push to GitHub. Visit the **Actions** tab to see your CI workflow in motion.

✅ Congrats! You now have full-stack automated testing with API + UI tests in GitHub Actions.

<div class="activity discussion">
  <h2 class="title">Group Reflection</h2>
  <span class="minutes">10 min</span>
</div>

- What made browser-based testing different from earlier tests?
- Why does GitHub Actions require headless mode and Chromium install?
- How would this flow change if your frontend was a React app?

## Deliverable

- Working pipeline (can show passing check on GitHub)
