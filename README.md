# Quiz Master

An interactive, full-stack quiz management and assessment platform designed to facilitate learning **Gurmukhi** and **Sikh History**. It features a robust Flask RESTful API backend, a dynamic Vue 3 frontend, cached API responses via Redis, asynchronous background processing with Celery, and automated notifications/reports.

---

## 🚀 Features

### 👨‍🎓 Student Features
- **Interactive Quizzes**: Take quizzes with real-time timers.
- **Personal Dashboard**: Track overall performance metrics and history.
- **Visual Analytics**: Dynamic graphs displaying scoring history and performance stats (powered by Chart.js).
- **Asynchronous Reports**: Export personal quiz results and detailed answer sheets as CSV.
- **Multi-Category Filter**: Sort and filter quizzes by Subject and Chapter.

### 👑 Admin Features
- **Subject & Chapter CRUD**: Manage course structure easily.
- **Quiz & Question Builder**: Create quizzes, customize durations, add multiple-choice questions (options A-D), assign marks, and specify correct answers.
- **Student Leaderboard & Details**: View attempts and tracking information for all users.

### ⚙️ Asynchronous & Scheduled Tasks (Celery & Redis)
- **Daily Reminders**: Automated alerts to users on Google Chat spaces about unattempted quizzes.
- **Monthly Progress Reports**: Rich HTML emails containing comprehensive score tables sent to students (templated with Jinja2).
- **CSV Data Exports**: Generates downloadable CSV exports for answers and scores asynchronously.

---

## 🛠️ Tech Stack

- **Frontend**: [Vue 3](https://vuejs.org/), [Vue Router](https://router.vuejs.org/), [Vuex](https://vuex.vuejs.org/), [Axios](https://axios-http.com/), [Chart.js](https://www.chartjs.org/)
- **Backend**: [Flask](https://flask.palletsprojects.com/), [Flask-RESTful](https://flask-restful.readthedocs.io/), [Flask-SQLAlchemy](https://flask-sqlalchemy.readthedocs.io/) (SQLite)
- **Authentication**: JWT ([Flask-JWT-Extended](https://flask-jwt-extended.readthedocs.io/))
- **Caching**: [Flask-Caching](https://flask-caching.readthedocs.io/) (Redis backend)
- **Task Queue & Scheduler**: [Celery](https://docs.celeryq.dev/) & [Redis](https://redis.io/)
- **Mailing**: SMTP ([localhost:1025](file:///Users/singh/learn/sikh/quiz_master/backend/async_tasks/mail_send.py#L8))

---

## 💻 Setup & Installation

### 1. Redis Server Setup
Redis is required as the Flask Cache backend, Celery Message Broker, and Celery Result Backend.
Ensure Redis is installed and running on default port `6379`:
```bash
# On macOS via Homebrew (install first if not already installed)
brew install redis

# Start the Redis service in the background
brew services start redis
```

### 2. Mock SMTP Mail Server
For receiving monthly reports locally, run a test SMTP server on port `1025`. 

*(Note: The built-in `smtpd` Python module has been removed in Python 3.12+. Use one of the modern options below:)*

#### Option A: Using Mailpit (Recommended)
Mailpit is a fast, modern email testing tool that acts as an SMTP server and provides a web interface to inspect sent emails.
```bash
# Install Mailpit via Homebrew
brew install mailpit

# Run Mailpit (SMTP server runs on localhost:1025, Web UI on http://localhost:8025)
mailpit
```

#### Option B: Using `aiosmtpd`
An asyncio-based SMTP server for Python 3.
```bash
# Install the package
pip install aiosmtpd

# Run SMTP server on port 1025
aiosmtpd -n -l localhost:1025
```

### 3. Backend Setup (Flask & Celery)
1. **Initialize Virtual Environment**:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
2. **Install Dependencies**:
   ```bash
   pip3 install -r requirements.txt
   ```
3. **Run Celery Worker** (In a separate terminal with `.venv` active):
   ```bash
   cd backend
   celery -A app.celery worker --loglevel=info
   ```
4. **Run Celery Beat Scheduler** (In a separate terminal with `.venv` active, handles daily & monthly reminders):
   ```bash
   cd backend
   celery -A app.celery beat --loglevel=info
   ```
5. **Start Flask Dev Server**:
   ```bash
   cd backend
   python3 app.py
   ```
   *(Note: The database is initialized and seeded automatically on start. SQLite database file `quizmaster.db` will be created inside the `backend/instance/` directory.)*

### 4. Frontend Setup (Vue 3)
1. **Navigate to frontend directory**:
   ```bash
   cd frontend
   ```
2. **Install Dependencies**:
   ```bash
   npm install
   ```
3. **Run Development Server**:
   ```bash
   npm run serve
   ```
   Open [http://localhost:8080](http://localhost:8080) to access the app.

---

## 🔑 Default Credentials

The database automatically seeds two test accounts upon startup:

* **Admin Role**:
  - **Email**: `singhadmin@gmail.com`
  - **Password**: `penduwarrior`
* **Student Role**:
  - **Email**: `dalyodhsingh1@gmail.com`
  - **Password**: `penduwarrior`

---

## 🔗 Main API Endpoints

| Endpoint | HTTP Method | Auth Role | Description |
|---|---|---|---|
| `/api/signup` | `POST` | Public | Register new students |
| `/api/login` | `POST` | Public | Authenticate users and retrieve JWT token |
| `/api/user` | `GET` | Admin, Student | Fetch current user profile details |
| `/api/users` | `GET` | Admin | Fetch details of all registered users |
| `/api/subject` | `GET`, `POST`, `PUT`, `DELETE` | Admin (Write/Delete) | Manage learning subjects |
| `/api/chapter` | `GET`, `POST`, `PUT`, `DELETE` | Admin (Write/Delete) | Manage chapters under subjects |
| `/api/quiz` | `GET`, `POST`, `PUT`, `DELETE` | Admin (Write/Delete) | Manage quizzes |
| `/api/question` | `GET`, `POST`, `PUT`, `DELETE` | Admin (Write/Delete) | Manage multiple-choice questions |
| `/api/score` | `GET`, `POST` | Admin, Student | Score calculations & attempt history |
| `/api/export_csv/<score_id>`| `GET` | Student, Admin | Trigger background Celery task for score export |
| `/api/export_quiz_csv/<quiz_id>`| `GET` | Student, Admin | Trigger background Celery task for quiz export |
| `/api/csv_result/<id>` | `GET` | Public | Fetch/download generated CSV files |
