<h1>
  <span class="headline">Running Selenium Tests in CI Pipelines Lab</span>
  <span class="subhead">Setup</span>
</h1>

## Setup

For this project we will need to use github Actions. This feature is available for free on **Github.com**.

## Log in and Create a Repo

Go to [Github](https://github.com/) and sign into your account.

Create a new repository named **`running-selenium-tests-in-ci-piplines-lab`**.

- Set the repo to **Public**
- Check the box to include a **README.md**
- You do **not** need a `.gitignore` file

Using the `Quick Setup` option, clone your newly created repo onto your local machine with the `git clone` command:

```bash
git clone https://github.com/<your-username>/running-selenium-tests-in-ci-piplines-lab.git
```

> Note: In the link above, where it says `<your-username>`, you should see the username from your GitHub account.

Next, `cd` into your new cloned directory, `running-selenium-tests-in-ci-piplines-lab`:

```bash
cd running-selenium-tests-in-ci-piplines-lab
```

Open the directory in your code editor:

```bash
code .
```

## Set Up a Virtual Environment

Create and activate a new virtual environment:

**macOS/Linux:**

```bash
python -m venv venv
source venv/bin/activate
```

**Windows:**

```bash
python -m venv venv
venv\Scripts\activate
```

## Install Dependencies

Create a `requirements.txt` file in the root directory.

Paste the following dependencies in that file.

```txt
fastapi==0.110.0
uvicorn==0.29.0
httpx==0.27.0
pytest==8.1.1
python-multipart==0.0.9
selenium==4.20.0
pydantic==2.7.1
pydantic[email]==2.7.1
```

And install with:

```bash
pip install -r requirements.txt
```

## Create your starter files

1. Create three new directories called `app/`, `static/`, and `tests/`.

  ```bash
  mkdir -p app/static tests
  ```

2. Create special `__init__.py` files so that Python treats these folders as packages:

  ```bash
  touch app/__init__.py tests/__init__.py
  ```

  > These files can be empty, but they’re required for Python to recognize the folders as importable modules.

3. Inside the `app/static/` folder, create a new file called `newsletter_signup.html`.

    ```bash
    touch app/static/newsletter_signup.html
    ```

  Paste the following code into it:

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <title>ByteBeat Newsletter Signup</title>
      <style>
        body {
          background-color: #121212;
          color: #ffffff;
          font-family: Arial, sans-serif;
          display: flex;
          justify-content: center;
          align-items: center;
          height: 100vh;
          margin: 0;
        }

        .container {
          background-color: #1e1e1e;
          padding: 2rem;
          border-radius: 10px;
          box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
          width: 100%;
          max-width: 400px;
        }

        h1 {
          text-align: center;
          margin-bottom: 1rem;
        }

        p.description {
          text-align: center;
          font-size: 0.9rem;
          color: #bbbbbb;
          margin-bottom: 2rem;
        }

        input,
        button {
          width: 100%;
          padding: 0.75rem;
          margin-bottom: 1rem;
          border: none;
          border-radius: 5px;
          font-size: 1rem;
          box-sizing: border-box;
        }

        input {
          background-color: #2c2c2c;
          color: #ffffff;
        }

        button {
          background-color: #3b82f6;
          color: white;
          cursor: pointer;
          transition: background-color 0.3s ease;
        }

        button:hover {
          background-color: #2563eb;
        }

        .message {
          text-align: center;
          margin-top: 1rem;
          font-size: 0.9rem;
          color: #00e676;
        }
      </style>
    </head>
    <body>
      <div class="container">
        <h1>Subscribe to ByteBeat</h1>
        <p class="description">
          ByteBeat is your weekly digest of tech tips, dev tools, and productivity
          hacks. Join us to stay sharp and informed.
        </p>
        <form id="signup-form">
          <input type="text" id="name" placeholder="Your Name" required />
          <input type="email" id="email" placeholder="Your Email" required />
          <button type="submit">Subscribe</button>
        </form>
        <div class="message" id="message"></div>

        <script>
          document
            .getElementById('signup-form')
            .addEventListener('submit', function (e) {
              e.preventDefault();
              const name = document.getElementById('name').value;
              const email = document.getElementById('email').value;
              if (name && email) {
                fetch('/signup', {
                  method: 'POST',
                  headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                  },
                  body: new URLSearchParams({ name, email }),
                })
                  .then((response) => response.json())
                  .then((data) => {
                    document.getElementById('message').textContent =
                      data.message || 'Thanks for subscribing, ' + name + '!';
                  })
                  .catch(() => {
                    document.getElementById('message').textContent =
                      'Something went wrong. Please try again later.';
                  });
                document.getElementById('signup-form').reset();
              }
            });
        </script>
      </div>
    </body>
  </html>
  ```

4. Create a new file inside `app/` called `main.py`.

  Paste the following code into it:

  ```python
  from fastapi import FastAPI, Form
  from fastapi.responses import HTMLResponse
  from fastapi.staticfiles import StaticFiles
  from pydantic import BaseModel, EmailStr, constr
  from pathlib import Path

  app = FastAPI()

  # Define the static directory path
  STATIC_DIR = Path(__file__).resolve().parent / "static"

  # Serve files from the "static" folder
  app.mount("/static", StaticFiles(directory=STATIC_DIR), name="static")

  # Model for validating form input
  class SignupForm(BaseModel):
      name: constr(min_length=1, max_length=100)
      email: EmailStr


  @app.get("/signup", response_class=HTMLResponse)
  def get_signup_form():
      html_path = STATIC_DIR / "newsletter_signup.html"
      return html_path.read_text()


  @app.post("/signup")
  def post_signup(
      name: str = Form(...),
      email: str = Form(...)
  ):
      form = SignupForm(name=name, email=email)
      return {"message": f"Thanks for subscribing, {form.name}!"}
  ```

You are ready to go!
