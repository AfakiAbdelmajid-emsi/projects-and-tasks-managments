## Project & Task Management (Flask)

A lightweight Project, Task, Team, and Event management web app built with Flask, SQLAlchemy, Flask-Login, and Flask-Bcrypt. It supports two roles: Admin and Employee. Admins can manage users, teams, projects, tasks, and calendar events. Employees can view assigned work, update progress, manage personal tasks, collaborate via teams, and view a role-aware calendar.

### Features
- **Authentication & Roles**: Login/logout with password hashing; roles for Admin and Employee.
- **User Management (Admin)**: Activate/deactivate users, assign roles, delete users, view user details.
- **Teams**: Create/update/delete teams, assign members and supervisors, view team composition.
- **Projects**: Create/update/delete projects, link teams to projects, open/close status with constraints.
- **Tasks**:
  - Admin: create/update/delete tasks, assign employees, link to projects, status management.
  - Employee: view assigned tasks, mark done, add progressions.
- **Personal Tasks (Employee)**: CRUD and per-task progression tracking.
- **Events & Calendar**: Create/update/delete events, associate teams/employees; unified calendar view for Admin and role-aware view for Employee.
- **Profiles**: Update profile info and upload profile pictures (stored as binary in DB).

### Tech Stack
- Python, Flask
- Flask-SQLAlchemy, MySQL
- Flask-Login, Flask-Bcrypt
- Jinja2 templates

### Repository Structure (high level)
```
Project_and_Task_Management-main/
  app.py                 # Flask app factory/bootstrap
  config.py              # App configuration (env-driven)
  extensions.py          # db, bcrypt, login_manager, allowed file types
  modals.py              # SQLAlchemy models (Users, Teams, Project, Task, Event, ...)
  authroutes.py          # Auth routes (login, logout)
  adminroutes.py         # Admin blueprint: users, teams, projects, tasks, calendar, profile
  employeeroutes.py      # Employee blueprint: tasks, personal tasks, teams, projects, calendar, profile
  utils.py               # Helpers for status and sorting
  templates/             # Jinja templates for admin, employee, shared components
  static/                # Static assets (e.g., logos)
```

### Getting Started

#### Prerequisites
- Python 3.10+
- MySQL 8.x (or compatible)

#### 1) Clone and create a virtual environment
```bash
git clone <your-fork-or-repo-url>
cd Project_and_Task_Management-main
python -m venv .venv
.venv\Scripts\activate  # Windows PowerShell
```

#### 2) Install dependencies
Install the following packages (or add them to a `requirements.txt`):
```bash
pip install Flask Flask-SQLAlchemy Flask-Bcrypt Flask-Login PyMySQL
```

#### 3) Configure environment
Set environment variables or edit `config.py` defaults.
- **SECRET_KEY**: any random string for session security
- **DATABASE_URL**: SQLAlchemy URI to your MySQL database

Examples (Windows PowerShell):
```powershell
$env:SECRET_KEY = "change-me"
$env:DATABASE_URL = "mysql+pymysql://root:yourpassword@localhost/python_project"
```

Create the database beforehand (from a MySQL shell):
```sql
CREATE DATABASE python_project CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Note: In development, if you are not using HTTPS, you may set `SESSION_COOKIE_SECURE` to `False` in `config.py` to avoid cookie issues locally.

#### 4) Run the app
```bash
python app.py
```
The app will create all tables on first run and be available at `http://127.0.0.1:5000/`.

### Accounts and Roles
- Login page: `/login`
- There is no public registration flow by default. Admins can create users from the Admin area.
- For the very first Admin account, create a user directly in the database and set fields:
  - `users.status = 'active'`
  - `users.usertype = 'Admin'`
  - `users.Pasword` should be a bcrypt hash. Generate one quickly via Python:
    ```python
    from flask_bcrypt import Bcrypt
    bcrypt = Bcrypt()
    bcrypt.generate_password_hash("YourPassword").decode("utf-8")
    ```
  Insert the resulting string into the `Pasword` column.

### Configuration Reference (`config.py`)
- `SECRET_KEY`: Flask secret key (env: `SECRET_KEY`)
- `SQLALCHEMY_DATABASE_URI`: DB URI (env: `DATABASE_URL`), e.g. `mysql+pymysql://user:pass@host/db`
- `SESSION_COOKIE_HTTPONLY`: True
- `SESSION_COOKIE_SECURE`: True (consider False in local dev)
- `SQLALCHEMY_TRACK_MODIFICATIONS`: False

### Data Models (summary)
- `users`: accounts with role (`usertype`: `Admin`|`employee`), status, profile picture
- `Task`, `TaskAssignment`, `Task_Progression`
- `PersonalTask`, `PersonalTaskProgression`
- `Teams`, `TeamsMember`
- `Project`, `ProjectTeam`
- `Event`, `EventEmployee`, `EventTeam`

### Route Map (high level)

Auth (no prefix)
- `GET /` → redirect to `/login`
- `GET|POST /login` → login
- `GET /logout` → logout

Admin (`/admin` prefix)
- Users: `/usersdashboard`, `/usersdashboard/userdetails/<Utoken>`, `POST /update_status/<userid>`, `POST /delete_user/<userid>`
- Profile: `/profile/<Utoken>`, `POST /change_full_name/<Utoken>`, `POST /change_email/<Utoken>`, `POST /change_password/<Utoken>`
- Teams: `GET /teams`, `GET|POST /teams/Create_new_team`, `GET|POST /teams/update/<TETOKEN>`, `GET|POST /teams/delete_team/<TETOKEN>`, member add/remove
- Projects: `GET|POST /projects`, `GET|POST /projects/create_project`, `GET|POST /projects/update/<token>`, `POST /projects/delete/<id>`, `GET|POST /projects/project_details/<token>`
- Tasks: `GET|POST /tasks`, `GET|POST /tasks/Create_new_task`, `GET|POST /tasks/update/<token>`, `POST /tasks/delete/<id>`, `POST /update_task_statut`
- Calendar/Events: `GET /calendar`, `GET|POST /calendar/events/update/<token>`, `POST /calendar/events/delete/<token>`, `GET|POST /calendar/calendar/add_event`

Employee (`/employee` prefix)
- Tasks: `GET|POST /Assignedtasks`, `GET|POST /Assignedtasks/taskdetails/<token>/<etoken>`, `GET|POST /Assignedtasks/taskdetails/addprogression/<token>/<etoken>`
- Personal Tasks: `GET /personaltasks`, `GET|POST /personaltasks/personaltaskdetail/<Ptoken>`, `GET|POST /personaltasks/personaltaskdetail/addpersonalprogression/<Ptoken>`, edit/toggle/delete endpoints
- Teams: `GET /teams`
- Projects: `GET /team_projects`, `GET /team_projects/project_details/<token>`, `GET|POST /team_projects/create_new_task/<project_token>`, `GET|POST /team_projects/update_task/<task_token>`, `GET|POST /team_projects/delete/<id>`, `POST /team_projects/update_project_task_statut`
- Calendar/Events: `GET /calendar`, `GET|POST /team/<team_id>/add_event`, `POST /events/delete/<token>`, `GET|POST /events/update/<token>`
- Profile: `GET|POST /profile/<Utoken>`, `POST /change_full_name/<Utoken>`, `POST /change_email/<Utoken>`, `POST /change_password/<Utoken>`

Note: All Admin routes require an authenticated user with `usertype == 'Admin'`; Employee routes require `usertype == 'employee'`.

### Usage Tips
- Projects and tasks enforce sensible date constraints (e.g., task start cannot precede project start).
- Calendar aggregates tasks, projects, and events; Employee calendar filters to relevant items (assigned, team, supervised).
- Profile pictures accept `png`, `jpg`, `jpeg`, `gif` (see `ALLOWED_EXTENSIONS` in `extensions.py`).

### Troubleshooting
- If login loops back to the login page, ensure the user `status` is `active` and role is set correctly.
- For local development without HTTPS, consider setting `SESSION_COOKIE_SECURE=False` to avoid cookie issues.
- Verify `DATABASE_URL` uses the correct driver: `mysql+pymysql://...` when PyMySQL is installed.

### Screenshots
Add screenshots to the `static/` folder and reference them here.

### License
Add a license of your choice (MIT recommended) if you plan to open-source.
