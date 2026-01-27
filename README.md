# Personal Finance Dashboard

A web-based financial dashboard built with Dash and Plotly to track income and expenses over time. It supports categories, payment methods, installments, date filtering, and interactive charts, providing a clear and structured view of personal financial activity.

# Instructions

1) Prerequisites

Before anything else, the user must have the following installed:

Required

- Docker
- Docker Compose (already included in Docker Desktop)

Check in the terminal:

<pre>
docker --version
docker compose version
</pre>

If not installed:

Windows / macOS: https://www.docker.com/products/docker-desktop  
Linux: use your distribution’s package manager

2) Cloning the project

In the terminal:

<pre>
git clone <repository_url>
cd <local_repository>
</pre>

Expected structure (example):

<pre>
.
├── app.py
├── config.py
├── loading.html
├── requirements.txt
├── docker-compose.yml
├── Dockerfile
├── data/
│   ├── transactions.csv
│   ├── categories.csv
│   └── payment_methods.csv
</pre>

The CSV files must exist (even if empty), because the app reads them at startup.

3) Starting the containers

From the project root:

<pre>
docker compose up -d --build
</pre>

This will:

- Build the dashboard container

Check if everything is running:

<pre>
docker ps
</pre>

You should see something like:

<pre>
financial-dashboard
</pre>

5) Accessing the Dashboard

Open your browser:

http://localhost:8050

You will see:

- Financial charts
- Transaction tables

6) Where is the data stored?

The CSV files are stored outside the container, in the data/ folder.

Example:

<pre>
data/
├── transactions.csv
├── categories.csv
└── payment_methods.csv
</pre>

This ensures that:

- Your data is not lost when restarting containers
- You can manually edit the CSV files if you want

7) Stopping the system

To stop everything:

<pre>
docker compose down
</pre>

8) Common issues

❌ Port 8050 is already in use

Edit docker-compose.yml:

ports:

"8080:8050"

Then access:

http://localhost:8080

9) Mental model of how it works

- Dash runs in financial-dashboard
- No external dependencies
