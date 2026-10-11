# Mergington High School Activities API

A FastAPI application for viewing extracurricular activities. Teachers sign in to register or unregister students; visitors can browse activities and participant lists without signing in.

## Features

- View all available extracurricular activities
- Browse activities and participant lists without signing in
- Teacher-only student registration and removal
- Eight-hour teacher sessions in signed, HTTP-only cookies

## Getting Started

1. Install the dependencies:

   ```
   pip install -r requirements.txt
   ```

2. Create a local teacher credentials file from the example:

   ```
   cp src/teachers.example.json src/teachers.json
   ```

   Generate a salted password hash and add the resulting object to the `teachers` array:

   ```python
   import getpass
   import hashlib
   import secrets

   username = input("Teacher username: ")
   password = getpass.getpass("Teacher password: ").encode()
   salt = secrets.token_bytes(16)
   password_hash = hashlib.pbkdf2_hmac("sha256", password, salt, 310_000).hex()
   print({"username": username, "salt": salt.hex(), "password_hash": password_hash})
   ```

   `src/teachers.json` is local and ignored by Git. Keep it private and assign a unique password to each teacher.

3. Set a persistent session signing secret and run the application:

   ```sh
   export SESSION_SECRET="$(python -c 'import secrets; print(secrets.token_urlsafe(32))')"
   uvicorn src.app:app --reload
   ```

   For HTTPS deployments, set `COOKIE_SECURE=true`. Do not enable it for local HTTP development.

4. Open your browser and go to:
   - API documentation: http://localhost:8000/docs
   - Alternative documentation: http://localhost:8000/redoc
   - Activities page: http://localhost:8000/

## API Endpoints

| Method | Endpoint                                                          | Description                                                         |
| ------ | ----------------------------------------------------------------- | ------------------------------------------------------------------- |
| GET    | `/activities`                                                     | Get all activities with their details and current participant count |
| POST   | `/auth/login`                                                      | Sign in as a teacher and start a session                             |
| GET    | `/auth/session`                                                    | Get the current teacher session                                     |
| DELETE | `/auth/session`                                                    | Sign out                                                            |
| POST   | `/activities/{activity_name}/signup?email=student@mergington.edu` | Register a student (teacher session required)                       |
| DELETE | `/activities/{activity_name}/unregister?email=student@mergington.edu` | Unregister a student (teacher session required)                  |

## Data Model

The application uses a simple data model with meaningful identifiers:

1. **Activities** - Uses activity name as identifier:

   - Description
   - Schedule
   - Maximum number of participants allowed
   - List of student emails who are signed up

2. **Students** - Uses email as identifier:
   - Name
   - Grade level

Activity and participant data is stored in memory and resets when the server restarts. Teacher credentials are stored locally in `src/teachers.json` as PBKDF2 hashes; the template is `src/teachers.example.json`.
