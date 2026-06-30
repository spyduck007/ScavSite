# ScavSite

ScavSite is a Django web app for running school scavenger hunt competitions. It was built for class-based school competitions where students sign in with TJ Ion, solve challenges, and earn points for their graduating class.

The app provides a public login/challenge experience for participants, an admin-managed challenge catalog, a live class leaderboard, and analytics pages for organizers.

## Features

- TJ Ion OAuth login for student authentication.
- Class-year teams with automatic leaderboard scoring.
- Challenge categories with sortable challenge order.
- Multiple challenge mechanics:
  - Regular challenges award a fixed point value.
  - Exclusive challenges can only be claimed by the first class to solve them.
  - Decreasing challenges award fewer points after each solve.
  - Dependent challenges unlock after prerequisite challenges are solved.
- Answer validation with optional case sensitivity.
- Submission cooldowns to reduce rapid guessing.
- Hunt start and end windows, with organizer bypass access.
- Organizer analytics for participants, submissions, first bloods, class stats, and challenge stats.
- Discord first-blood webhook notifications.
- Whitenoise static file serving for deployment.

## Tech Stack

- Python
- Django 5
- SQLite by default
- django-environ for environment configuration
- requests-oauthlib for Ion OAuth
- Gunicorn and Whitenoise for production serving

## Project Structure

```text
ScavSite/
|-- core/                  # Main scavenger hunt app
|   |-- models.py          # Participants, challenges, solves, Discord settings
|   |-- views.py           # Login, challenge, submission, analytics views
|   |-- admin.py           # Django admin configuration
|   `-- templates/core/    # App templates
|-- hunt/                  # Django project settings and URL config
|-- static/                # Source static assets
|-- staticfiles/           # Collected static assets
|-- manage.py
|-- requirements.txt
`-- run.sh                 # Gunicorn startup script
```

## How It Works

Participants visit the site and sign in with Ion. On successful OAuth login, ScavSite creates or updates a `Participant` record using the Ion profile, including the student's username, display name, email, graduation year, and admin status.

Each participant is assigned to a team based on graduation year. The configured team years are stored in `SCAV_HUNT_TEAM_YEARS`; if they are not set, the app derives four class years from the hunt end date or current year.

Organizers create challenge categories and challenges in Django admin. A challenge stores its prompt, answer, point value, type, active state, and optional prerequisites. When a student submits a correct answer, the app creates a `ChallengeSolve` for that challenge and class year. A class can solve a challenge once, and the awarded points are added to the class leaderboard.

The challenge page shows available categories, challenge cards, solve state, point values, and the leaderboard. The analytics dashboard gives organizers a deeper view of submissions, class progress, first bloods, and per-challenge performance.

## Getting Started

### Prerequisites

- Python 3.11 or newer recommended
- A TJ Ion OAuth application for real login

### Installation

Clone the repository and install dependencies:

```bash
git clone https://github.com/hsna674/Scav2026.git
cd Scav2026
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create a local environment file:

```bash
cp .env.example .env
```

Update `.env` with your Django secret key, allowed hosts, Ion OAuth credentials, hunt dates, and team years.

Run database migrations:

```bash
python manage.py migrate
```

Create an admin user if needed:

```bash
python manage.py createsuperuser
```

Start the development server:

```bash
python manage.py runserver
```

The site will be available at `http://127.0.0.1:8000/`.

## Configuration

The app reads configuration from environment variables or a local `.env` file.

| Variable | Purpose |
| --- | --- |
| `DJANGO_SECRET_KEY` | Required Django secret key. |
| `DJANGO_DEBUG` | Enables debug mode when true. |
| `DJANGO_ALLOWED_HOSTS` | Comma-separated allowed hosts. |
| `DJANGO_CSRF_TRUSTED_ORIGINS` | Comma-separated trusted CSRF origins. |
| `ION_CLIENT_ID` | Ion OAuth client ID. |
| `ION_CLIENT_SECRET` | Ion OAuth client secret. |
| `ION_REDIRECT_URI` | OAuth callback URL, usually `/complete/ion/`. |
| `ION_SCOPE` | Ion OAuth scopes. |
| `SCAV_HUNT_START` | Optional ISO 8601 hunt opening datetime. |
| `SCAV_HUNT_END` | Optional ISO 8601 hunt closing datetime. |
| `SCAV_HUNT_TEAM_YEARS` | Comma-separated class years competing in the hunt. |
| `SCAV_SUBMISSION_COOLDOWN_SECONDS` | Seconds a participant must wait between answer attempts. |
| `HUNT_YEAR` | Display year shown in the interface. |
| `ENABLE_SNOW_OVERLAY` | Enables or disables the seasonal snow overlay. |

Example:

```env
DJANGO_SECRET_KEY=change-me
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
ION_CLIENT_ID=your-ion-client-id
ION_CLIENT_SECRET=your-ion-client-secret
ION_REDIRECT_URI=http://localhost:8000/complete/ion/
SCAV_HUNT_START=2026-04-10T08:00:00
SCAV_HUNT_END=2026-04-17T22:00:00
SCAV_HUNT_TEAM_YEARS=2027,2028,2029,2030
HUNT_YEAR=2026
```

## Admin Workflow

1. Sign in as a Django superuser or an Ion user marked as an admin.
2. Open `/admin/`.
3. Create active challenge categories.
4. Create challenges inside those categories.
5. Set each challenge's type, answer, point value, and sort order.
6. For dependent challenges, add prerequisite challenges from the same category.
7. Optionally configure Discord first-blood notifications in `Discord Settings`.

Admins can also use the challenge page controls to reorder challenges within a category.

## Analytics

Users marked as admin or scavcomm can open `/analytics/`. The dashboard includes:

- Overall participant, submission, point, and challenge totals.
- Class-by-class scores and recent activity.
- Recent submission logs.
- Challenge solve counts and first-blood details.
- Detail pages for users, submissions, and challenges.

Scavcomm users can view analytics without full Django admin access.

## Deployment

The project includes `run.sh`, which starts Gunicorn with the Django WSGI app:

```bash
gunicorn hunt.wsgi -b $HOST:$PORT -w 1
```

For production, set `DJANGO_DEBUG=False`, configure `DJANGO_ALLOWED_HOSTS` and `DJANGO_CSRF_TRUSTED_ORIGINS`, provide real Ion OAuth credentials, and run:

```bash
python manage.py migrate
python manage.py collectstatic
```

SQLite is used by default. For larger or long-running competitions, consider switching `DATABASES` in `hunt/settings.py` to a managed database such as PostgreSQL.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
