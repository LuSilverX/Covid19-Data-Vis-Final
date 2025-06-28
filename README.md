# COVID-19 Data Dashboard

![Python](https://img.shields.io/badge/Python-3.9+-blue?style=for-the-badge&logo=python)
![Django](https://img.shields.io/badge/Django-5.1-blue?style=for-the-badge&logo=django)
![Celery](https://img.shields.io/badge/Celery-5.x-blue?style=for-the-badge&logo=celery)
![Selenium](https://img.shields.io/badge/Selenium-4.x-blue?style=for-the-badge&logo=selenium)

A sophisticated Django web application that provides a dual-purpose dashboard for tracking and visualizing COVID-19 data. It features a historical data explorer for early pandemic trends and a live data tracker that performs on-demand web scraping to fetch the latest information from official sources.

## Key Features

- **Historical Data Explorer**: Visualize early pandemic data with interactive, paginated tables and filterable line charts for US National, State, and County levels.
- **Live Data Tracker**: Get the most recent data available from the CDC and WHO.
- **On-Demand Web Scraping**: Users can initiate a background task to scrape the very latest data from the CDC website, which no longer provides a public data API.
- **Asynchronous Task Handling**: Utilizes Celery and Redis to manage long-running scraping tasks in the background without blocking the user interface.
- **Dynamic Frontend**: The frontend polls the server for task completion and dynamically updates data tables without requiring a page reload.
- **Scheduled Updates**: Automatically fetches data from the CDC and WHO on a weekly schedule using Celery Beat.

## How It Works

The project is split into two main user experiences:

1.  **Historical Dashboard (`/` or `/early-data/`)**: This section is powered by data imported from local CSV files (`Data/us.csv`, etc.). It uses a suite of AJAX APIs to provide paginated table data and filterable chart data, offering a rich user experience for exploring historical trends.

2.  **Live Dashboard (`/live-data/`)**: This section provides up-to-the-minute data. When a user requests data for a specific state for the first time or clicks the "Refresh" button, a Celery task is dispatched. This task launches a headless Selenium browser, navigates the CDC website, downloads the data, parses it, and saves it to the database. The user's browser polls the backend until the task is complete and then displays the new data. A similar, simpler process exists for the WHO data, which is fetched from a direct CSV link.

## Technical Stack

- **Backend**: Django, Celery
- **Browser Automation**: Selenium
- **Data Fetching**: Requests
- **Database**: MySQL
- **Frontend**: Django Templates, JavaScript, jQuery, Chart.js
- **Asynchronous Task Queue**: Redis

## Project Structure

- `covid19_project/`: Main Django project configuration, settings, and root URL routing.
- `data_handler/`: The core Django app containing all models, views, tasks, and templates for the dashboard.
- `Data/`: Contains the initial CSV files needed to populate the historical database tables.
- `static/`: Contains the project's static assets (CSS, JavaScript).

## Setup and Installation

### 1. Prerequisites

- Python 3.9+
- MySQL Server
- Redis Server

### 2. Clone the Repository

```bash
# Clone the repository using the HTTPS method
git clone [https://github.com/LuSilverX/Covid19-Data-Vis-Final.git](https://github.com/LuSilverX/Covid19-Data-Vis-Final.git)

# After the clone is complete, navigate into the newly created directory
cd Covid19-Data-Vis-Final

3. Set Up Environment
   
Create and activate a virtual environment:

python3 -m venv venv
source venv/bin/activate
Install the required Python packages:

pip install -r requirements.txt
(Note: A requirements.txt file should be created with packages like django, celery, redis, selenium, requests, mysqlclient)

4. Database Configuration
Log in to your MySQL server and create a new database.
SQL
CREATE DATABASE covid19_data_vis;
Update the DATABASES setting in covid19_project/settings.py with your MySQL username and password.

7. Run Migrations
Apply the database schema:

python manage.py migrate

7. Initial Data Population (Historical Data)
The historical data for the main dashboard comes from the CSV files in the /Data directory. A custom management command is required to import this data.

Create a management command data_handler/management/commands/import_historical_data.py to parse the CSVs and populate the CovidUSData, CovidStateData, and CovidCountyData models.

Once the command is created, run it:

python manage.py import_historical_data

8. Running the Services
This project requires three separate processes to run concurrently in a development environment.
Run the Django Development Server:

python manage.py runserver

Run the Redis Server:
Ensure your Redis server is running. If you installed it via a package manager, it might already be running as a service.

redis-server
Run the Celery Worker:
In a new terminal, start the Celery worker to listen for tasks.

celery -A covid19_project worker -l info

Run the Celery Beat Scheduler (Optional):
To enable the scheduled weekly data fetches, run the Celery Beat scheduler in another terminal.

celery -A covid19_project beat -l info

You can now access the application at http://127.0.0.1:8000/.
