# Allianz Student Job Scraper

[![Python](https://img.shields.io/badge/Python-3.13-blue?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Selenium](https://img.shields.io/badge/Selenium-4.44-green?style=for-the-badge&logo=selenium&logoColor=white)](https://www.selenium.dev/)
[![Pandas](https://img.shields.io/badge/Pandas-2.2-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

An automated web scraper that collects Werkstudent and Internship job listings from the [Allianz Careers Portal](https://careers.allianz.com/global/en/search-results), filters them by country, and exports the results to a date-stamped Excel file. Built as a portfolio project demonstrating browser automation, dynamic page handling, and structured data extraction.

---

## Table of Contents

1. [About the Project](#about-the-project)
2. [Built With](#built-with)
3. [Getting Started](#getting-started)
   - [Prerequisites](#prerequisites)
   - [Installation](#installation)
4. [Usage](#usage)
5. [Output](#output)
6. [Known Issues](#known-issues)
7. [Roadmap](#roadmap)
8. [Contact](#contact)

---

## About the Project

The Allianz careers portal lists thousands of jobs globally but does not provide URL-level filtering by job level (e.g. Student/Intern roles). This means you cannot simply bookmark a filtered URL and return to it later — filters are applied through JavaScript state in the browser and disappear when you navigate away.

This project solves that problem by automating the entire search workflow:

- Launches a real Chrome browser via Selenium to handle the JavaScript-rendered page
- Programmatically clicks the **Student** job level filter before scraping begins
- Reads the filtered job count and calculates the exact number of pages to loop through
- Extracts title, link, location, category, and job level from every job card across all pages
- Handles the edge case where some jobs have multiple locations (stored as `MULTIPLE: Available in N locations`)
- Filters results to Germany-based positions and saves them to a date-stamped `.xlsx` file

**Why this approach is non-trivial:** The page is a JavaScript Single Page Application (SPA). Raw HTTP requests return an empty HTML shell — the actual job cards are injected into the DOM after JavaScript executes. Selenium solves this by waiting for specific DOM elements to appear before reading the page, ensuring data is only captured once the page has fully rendered.

---

## Built With

- **[Selenium 4.44](https://www.selenium.dev/)** — Browser automation; handles JavaScript rendering and filter interaction
- **[webdriver-manager](https://github.com/SergeyPirogov/webdriver_manager)** — Automatically downloads the correct ChromeDriver version to match your Chrome installation
- **[BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/)** — HTML parsing; extracts structured data from the rendered page source
- **[Pandas](https://pandas.pydata.org/)** — DataFrame manipulation, deduplication, and Excel export
- **[openpyxl](https://openpyxl.readthedocs.io/)** — Excel engine used by Pandas under the hood

---

## Getting Started

### Prerequisites

You need the following installed on your machine before running the project.

- **Python 3.9+** (tested on Python 3.13.2)
- **Google Chrome** (any recent version — webdriver-manager will match the correct ChromeDriver automatically)
- **Jupyter Notebook** or **JupyterLab** to run the `.ipynb` file

### Installation

**1. Clone the repository:**

```bash
git clone https://github.com/RehaSenel/allianz-job-scraper.git
cd allianz-job-scraper
```

**2. Install the required libraries.** You can run this directly inside the first cell of the notebook, or in your terminal:

```bash
pip install selenium webdriver-manager beautifulsoup4 pandas openpyxl
```

**3. Create the output directory** that the scraper saves results into:

```bash
mkdir -p jobs/jobs_in_germany
```

That's it — no API keys, no accounts, and no additional configuration required.

---

## Usage

Open `main.ipynb` in Jupyter and run the cells in order from top to bottom. Each cell is self-contained and clearly labeled, so you can stop after any step to inspect intermediate results.

The notebook is structured as follows. **Cell 1** installs all dependencies. **Cell 2** imports libraries and confirms everything loaded correctly. **Cell 3** defines the constants (`BASE_URL`, `JOBS_PER_PAGE`, `SLEEP_BETWEEN_PAGES`) — this is the only place you need to make changes if you want to adjust the scraper's behavior. **Cell 4** defines the `parse_job_card()` function which handles data extraction from a single job card. **Cell 5** runs the full scrape: it opens Chrome, applies the Student filter, reads the job count, and loops through all pages. **Cell 6** filters results to Germany and saves the Excel file.

> **Note:** The scraper opens a visible Chrome window by default so you can monitor progress. Once you have confirmed it works correctly, you can switch to headless mode by changing `headless=False` to `headless=True` in the driver options.

The delay between page requests is set to `1.0` second (`SLEEP_BETWEEN_PAGES = 1`) to be respectful to the server. A full scrape of ~200 pages takes approximately 5–7 minutes.

---

## Output

Results are saved to `./jobs/jobs_in_germany/` with the following naming convention:

```
allianz_student_jobs_YYYY-MM-DD.xlsx
```

Each row in the Excel file represents one job listing with the following columns:

| Column | Description | Example |
|--------|-------------|---------|
| `title` | Full job title | `Werkstudent im Bereich Lebensversicherung (m/w/d)` |
| `link` | Direct URL to the job posting | `https://careers.allianz.com/global/en/job/98350/...` |
| `location` | City, country, and postal code | `Unterföhring (bei München), Germany, 85774` |
| `category` | Business area | `Actuarial`, `Data & AI`, `Operations` |
| `job_level` | Seniority level | `Student` |

Jobs with multiple locations are stored as `MULTIPLE: Available in N locations` in the location column, since those locations are only loaded inside a modal popup after a user click and are not present in the initial page HTML.

> **Important:** The `.xlsx` output files are excluded from this repository via `.gitignore`. Only the code is version-controlled — no scraped data is published.

---

## Known Issues

**Filter state is lost on pagination.** When navigating to page 2, 3, etc. via direct URL (`?from=10&s=1`), the Student filter checkbox is no longer active in the new page load. The current workaround is that the filter is applied on the first page load and the scraper reads the filtered total job count at that point. However, subsequent pages load all job levels — the Student filtering is therefore done as a post-processing step on the full DataFrame rather than at scrape time. A future fix would re-apply the filter click on each page load.

**The Student filter checkbox ID is dynamic.** The checkbox element has an ID like `jobLevel_phs_Student188` where `188` is the current job count, which changes daily. The scraper uses a CSS attribute selector (`label[for^='jobLevel_phs_Student']`) that matches on the stable prefix rather than the full ID, so this does not cause failures — but it is worth knowing if you inspect the DOM and the ID looks different.

**Python 3.13 + Windows compatibility.** Playwright (an alternative to Selenium) is not compatible with Python 3.13 on Windows due to a known issue with `asyncio.create_subprocess_exec` and `ProactorEventLoop`. Selenium was chosen specifically because it does not have this limitation.

---

## Roadmap

- [ ] Re-apply the Student filter on each page load to ensure truly filtered pagination
- [ ] Add support for scraping multiple countries in a single run
- [ ] Add a daily change-detection feature: compare today's results with yesterday's file and output a summary of new and removed listings
- [ ] Extend to other job portals (DAAD, Stepstone, LinkedIn) with a unified output format
- [ ] Add a CV-matching module using an LLM API to score each listing against an uploaded CV

---

## Contact

**Reha Senel**
GitHub: [@RehaSenel](https://github.com/RehaSenel)

---

## Disclaimer

This project scrapes publicly available data from the Allianz careers portal for personal, non-commercial use. No scraped data is stored in or distributed through this repository. Please use responsibly and in accordance with the website's Terms of Service.
