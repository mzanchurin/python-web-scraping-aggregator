# python-web-scraping-aggregator

Web scraping and aggregation service built with **Python**, **Scrapy**, optional **Selenium**, and **PostgreSQL**.  
The project demonstrates how to scrape 2–3 different websites, normalize and store the data, expose it via an API, and run the scraping on a schedule.

---

## Goals

This repository is meant to showcase:

- **Integrations** with multiple external websites
- **Asynchronous / concurrent** scraping (Scrapy)
- Data **normalization and aggregation** into a single schema
- Persisting scraped data in **PostgreSQL**
- Exposing an **API** for searching / filtering aggregated results
- Running scraping jobs via a **task scheduler**

It is designed as a portfolio-ready example, not a full production system.

---

## Tech stack

- **Language:** Python
- **Scraping:** Scrapy
- **Browser automation (optional):** Selenium (for JS-heavy pages)
- **Database:** PostgreSQL
- **API:** (e.g.) FastAPI / Django REST Framework / Flask-RESTX (implementation-dependent)
- **Scheduling:** cron / APScheduler / simple CLI wrapper
- **Containerization (optional):** Docker & docker-compose

> The exact web framework and scheduler can be chosen during implementation.  
> This project focuses on the scraping + aggregation patterns.

---

## Features

### 1. Multi-site web scraping

- Scrapes **2–3 different websites** within the same domain/theme.
  - Example themes (to be chosen during implementation):
    - Job postings
    - Product prices
    - Real estate listings
    - News headlines
- Each site has its own Scrapy spider:
  - `SiteOneSpider`
  - `SiteTwoSpider`
  - `SiteThreeSpider` (optional)
- Extracts:
  - Common fields (title, url, price, location, published date, etc.)
  - Site-specific metadata (stored separately or in JSON)

### 2. Optional Selenium integration

- For pages with heavy JavaScript rendering:
  - Optional spider variants using Selenium (or Splash/Playwright, depending on setup).
- Demonstrates:
  - When pure Scrapy is enough
  - When headless browser automation is needed

### 3. Data normalization & aggregation

- Scraped items are converted to a **common internal schema** before storage:
  - e.g. `AggregatedItem` model with:
    - `source` (site identifier)
    - `external_id`
    - `title`
    - `url`
    - `description`
    - `price` (normalized to a base currency)
    - `metadata` (JSON for extra fields)
- Normalization layer:
  - Maps site-specific fields (e.g. `cost`, `amount`, `salary`) into a standard `price` field.
  - Parses and normalizes dates, currencies, locations.

### 4. PostgreSQL persistence

- All normalized items are stored in a PostgreSQL database.
- Suggested tables:
  - `sources` – configuration/metadata for each scraped site
  - `items` – main normalized records
  - `raw_items` (optional) – raw JSON dumps for debugging/auditing
  - `scrape_jobs` – when a scraping run started, ended, its status
- Provides:
  - Historical data retention
  - Ability to run analytics on top of the aggregated dataset.

### 5. REST API for consuming aggregated data

- Read-only API (for demo purposes) for:
  - Listing aggregated items
  - Filtering by:
    - Source
    - Date range
    - Price range
    - Search query (title/description)
- Example endpoints (implementation-specific):

  - `GET /api/items/`  
    List paginated items, with filters via query params.

  - `GET /api/items/{id}/`  
    Detail view for a single item.

  - `GET /api/sources/`  
    List configured sources.

  - `GET /api/scrape-jobs/`  
    See the history of scraping runs.

- Output format:
  - JSON (default)
  - Optional CSV export endpoint if needed.

### 6. Task scheduler (recurring scraping)

- Scraping runs are triggered on a schedule, for example:
  - Every hour
  - Every day at midnight
- Possible strategies:
  - **System cron** calling a CLI command:  
    `python manage.py run_spiders` or `python -m scraping.run_all`
  - **APScheduler** or similar in-process scheduler
  - Docker + cron image that runs Scrapy commands periodically

- Each run is recorded as a `scrape_jobs` entry with:
  - Start/end timestamps
  - Sites included
  - Number of items scraped
  - Status (success/failed)
  - Error message (if any)

---

## Architecture & Components

> File/module names are suggestions and can be adapted in implementation.

### 1. Scrapy project

- `scraping_project/`
  - `scraping_project/settings.py`
  - `scraping_project/pipelines.py`
  - `scraping_project/spiders/site_one.py`
  - `scraping_project/spiders/site_two.py`
  - `scraping_project/spiders/site_three.py` (optional)
- Pipelines:
  - Validation & cleaning
  - Normalization to the internal schema
  - Writing to PostgreSQL (via an ORM or direct SQL/SQLAlchemy)

### 2. API service

- `api_service/` (could be FastAPI, Django, Flask, etc.)
  - `models.py` – DB models / ORM definitions
  - `schemas.py` – Pydantic/serializer schemas
  - `routers.py` or `views.py` – API endpoints
  - `deps.py` – DB session / dependencies (if using FastAPI)

- DB models (example):

  - `Source`
  - `AggregatedItem`
  - `RawItem` (optional)
  - `ScrapeJob`

### 3. Scheduler

- `scheduler/`
  - `runner.py` – script that:
    - Triggers spiders (e.g. via Scrapy’s `CrawlerProcess` or command-line)
    - Logs job start/end in the DB
- System-level cron or APScheduler configuration to call `runner.py` periodically.

### 4. Configuration

- `.env` for environment variables:
  - DB connection (`DATABASE_URL`)
  - Scraping-related config (timeouts, user-agent, proxies)
- `config.py` or settings module to centralize configuration.

---

## Typical flow

1. **Scheduler triggers a scrape job**
   - Cron or APScheduler calls the scraping runner.
   - A `ScrapeJob` record is created (status: `running`).

2. **Scrapy spiders run**
   - Each configured spider crawls its target website.
   - Items are passed through pipelines:
     - Clean & validate
     - Normalize fields
     - Persist to PostgreSQL.

3. **Job completion**
   - On success:
     - `ScrapeJob` is updated with counts and status `success`.
   - On failure:
     - Status set to `failed`, error message stored.

4. **API consumption**
   - A client (frontend app / another service / CLI) calls the API:
     - `GET /api/items/?source=site_one&min_price=100`
   - API fetches from PostgreSQL and returns JSON results.

5. **Monitoring & debugging (optional)**
   - Use DB tables and logs to:
     - See how many items are ingested per run
     - See which runs failed and why
     - Compare data across sites.

---

## Getting started (conceptual)

> This is a suggested flow for when the project is implemented.  
> Exact commands and paths will depend on the chosen framework and final structure.

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/python-web-scraping-aggregator.git
cd python-web-scraping-aggregator
