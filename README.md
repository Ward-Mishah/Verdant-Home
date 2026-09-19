# Verdant Home

Verdant Home is a Django web prototype for exploring household resource-use data. It presents one daily record at a time and gives friendly sustainability prompts for high-use conditions.

> **Prototype scope:** The interface describes a future sensor- and AI-assisted home system. The checked-in code currently reads a bundled Excel workbook and applies deterministic threshold rules; it does not connect to physical sensors, a live data source, or an AI API.

## Features

- Displays electricity, water, air-pollution, air-conditioner, plugged-outlet, and room-brightness measurements.
- Selects a random sample record on dashboard load or refresh.
- Produces tailored conservation prompts when a measurement exceeds a threshold.
- Includes a simple landing page describing the sustainable-home concept.

## Methodology

The active `posts` application uses an explainable, rule-based workflow:

1. **Acquire sample data** — each `/posts/` request reads `static/data/Verdant_Home_Updated_Example_Data.xlsx` with pandas and selects one row at random.
2. **Present measurements** — the view maps the six fields to a Django template context, which renders “Today’s Usage.”
3. **Evaluate thresholds** — selecting **Want Some Tips?** evaluates the chosen row.
4. **Respond clearly** — one conservation prompt is returned per breached rule, followed by a randomly selected encouragement message. The alternative action displays a goodbye message.

| Measurement | Prompt shown above |
| --- | ---: |
| Electricity usage | 100 kWh |
| Water usage | 300 L |
| Air-pollution level | 50 PPM |
| AC usage | 8 hours |
| Plugged-in outlets | 15 |
| Room brightness | 80% |

This approach keeps each suggestion traceable to a clear condition; it should not be represented as a live measurement or model prediction.

## System design

```text
Browser
  │
  ├── /                 → VerdantHome.views.homepage → templates/home.html
  └── /posts/           → posts.views.posts_list
                             │
                             ├── pandas + openpyxl read Excel fixture
                             ├── random row selection
                             └── Django template context
                                  │
                                  ▼
                         posts/templates/posts/posts_list.html
                                  │
               ┌──────────────────┼──────────────────┐
               ▼                  ▼                  ▼
       /posts/refresh/    /posts/getrec/     /posts/goodbye/
       new random row     threshold advice   farewell message
```

| Component | Responsibility |
| --- | --- |
| `manage.py` | Django command-line entry point. |
| `VerdantHome/settings.py` | Django, template, SQLite, and static-file configuration. |
| `VerdantHome/urls.py` | Root routes and the `posts` route inclusion. |
| `VerdantHome/views.py` | Landing and about-page rendering; contains an unused earlier recommendation implementation. |
| `posts/views.py` | Active dashboard: Excel loading, random sampling, and rule-based recommendations. |
| `posts/urls.py` | Dashboard actions: refresh, recommendation, and goodbye. |
| `posts/templates/posts/posts_list.html` | Dashboard UI and action forms. |
| `static/data/Verdant_Home_Updated_Example_Data.xlsx` | Bundled prototype dataset. |
| `db.sqlite3` | Default local SQLite database; no custom models are currently defined. |

## Install in user-managed storage (Windows)

Store the project somewhere owned and writable by the user, such as `%USERPROFILE%\Documents\VerdantHome`. Avoid `Program Files` or other system-managed folders.

1. Clone the repository into user storage:

   ```powershell
   git clone https://github.com/Ward-Mishah/Verdant-Home.git "$env:USERPROFILE\Documents\VerdantHome"
   Set-Location "$env:USERPROFILE\Documents\VerdantHome"
   ```

   Alternatively, download the ZIP from GitHub and extract its contents there.

2. Create and activate a virtual environment:

   ```powershell
   py -3.13 -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   Python 3.11 or later is recommended; the checked-in bytecode indicates prior use of Python 3.13.

3. Install dependencies:

   ```powershell
   python -m pip install --upgrade pip
   python -m pip install "Django>=5.2,<5.3" pandas openpyxl
   ```

4. Create Django’s built-in tables and run the development server:

   ```powershell
   python manage.py migrate
   python manage.py runserver
   ```

5. Visit `http://127.0.0.1:8000/` and choose **See Today’s Usage?**.

Run commands from the project root because the Excel data is loaded through the relative path `./static/data/Verdant_Home_Updated_Example_Data.xlsx`. Runtime SQLite data remains in `db.sqlite3` under the user-managed project folder; back it up if you later add stored data.

## Production and development notes

- Rotate the development `SECRET_KEY`, set `DEBUG = False`, and remove environment-specific values from tracked source before deployment. Do not use Django’s development server in production.
- The Excel file is read on every dashboard request; a database, cache, or ingestion service would be better for production-scale data.
- Module-level globals hold recommendation state. They are not safe for concurrent users or multiple workers; use request/session state or database-backed models instead.
- Thresholds and messages are hard-coded. Move them into configuration or models for calibration.
- No dependency lockfile or automated tests are included. Add `requirements.txt` (or a modern dependency file) and test coverage before deployment.
