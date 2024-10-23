<p align="center">
  <a href="https://example.com/">
    <img src="https://via.placeholder.com/72" alt="Logo" width=72 height=72>
  </a>

  <h3 align="center">Logo</h3>

  <p align="center">
    Short description
    <br>
    <a href="https://reponame/issues/new?template=bug.md">Report bug</a>
    ·
    <a href="https://reponame/issues/new?template=feature.md&labels=feature">Request feature</a>
  </p>
</p>


## Table of contents

- [Instructions](#instructions)
- [Status](#status)
- [What's included](#whats-included)
- [Bugs and feature requests](#bugs-and-feature-requests)
- [Contributing](#contributing)
- [Creators](#creators)
- [Thanks](#thanks)
- [Copyright and license](#copyright-and-license)


## Status

Some text


## Folder structure

```text
folder1/
└── folder2/
    ├── folder3/
    │   ├── file1
    │   └── file2
    └── folder4/
        ├── file3
        └── file4
```

## Instructions

## Detailed Guide for Installing Python, SQLAlchemy, Flask on an AWS Ubuntu 24.04 LTS Web Server

In this guide, I’ll walk you through the steps to set up Python, Flask, and SQLAlchemy on an AWS EC2 instance running **Ubuntu 24.04 LTS**, along with configuring your domain **example.com**. This setup will allow you to develop a web application that displays CyberSecurity platform scores and a line graph of historical data.

### Prerequisites

- AWS Account and EC2 instance with Ubuntu 24.04 LTS (created and running).
- A registered domain: **example.com**.
- Basic knowledge of Python, Flask, and SQLAlchemy.
- Access to the AWS EC2 instance using SSH.
- An RDS instance or any SQL database you plan to use, or you can install a local database like MySQL/PostgreSQL.

### Step 1: SSH into Your EC2 Instance

First, SSH into your EC2 instance with the following command:

```bash
ssh -i /path/to/your/key.pem ubuntu@<your-ec2-public-ip>
```

### Step 2: Update and Upgrade System Packages

Run the following commands to update the package lists and upgrade the installed packages:

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 3: Install Python 3.10 and Pip

Ubuntu 24.04 LTS should come with Python 3 pre-installed, but ensure it is up to date. Install Python 3.10 and Pip (Python’s package manager):

```bash
sudo apt install python3.10 python3-pip -y
```

Verify the installation:

```bash
python3 --version
pip3 --version
```

### Step 4: Install Flask

Install Flask, the micro web framework:

```bash
pip3 install Flask
```

### Step 5: Install SQLAlchemy and a Database Driver

SQLAlchemy is an ORM (Object Relational Mapper) that simplifies interaction with the database. You’ll also need a database driver (for MySQL or PostgreSQL, depending on your backend database).

- **For MySQL**:

```bash
pip3 install SQLAlchemy pymysql
```

- **For PostgreSQL**:

```bash
pip3 install SQLAlchemy psycopg2
```

### Step 6: Set Up a Virtual Environment (Optional but Recommended)

It’s best practice to use a virtual environment to manage dependencies:

```bash
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
```

After activating the virtual environment, install Flask and SQLAlchemy again inside the environment:

```bash
pip3 install Flask SQLAlchemy pymysql  # For MySQL
pip3 install Flask SQLAlchemy psycopg2  # For PostgreSQL
```

### Step 7: Install a WSGI Server (Gunicorn)

Flask’s development server is not suitable for production, so install **Gunicorn** as your WSGI server:

```bash
pip3 install gunicorn
```

### Step 8: Create a Simple Flask App

Create a directory for your Flask app:

```bash
mkdir ~/flask_app
cd ~/flask_app
```

Create an initial `app.py` file with basic routes to display CyberSecurity scores:

```python
from flask import Flask, render_template
from sqlalchemy import create_engine, MetaData

app = Flask(__name__)

# Replace with your actual database connection string
DATABASE_URL = 'mysql+pymysql://username:password@host/dbname'

engine = create_engine(DATABASE_URL)
metadata = MetaData(bind=engine)

@app.route('/')
def home():
    # Query current CyberSecurity scores from the database
    # Replace this with actual queries to your table
    scores = [{'platform': 'Platform 1', 'score': 87}, 
              {'platform': 'Platform 2', 'score': 92},
              {'platform': 'Platform 3', 'score': 75},
              {'platform': 'Platform 4', 'score': 89},
              {'platform': 'Platform 5', 'score': 80}]
    
    # Add logic to query and pass historical data for the line graph here
    return render_template('index.html', scores=scores)

if __name__ == '__main__':
    app.run(debug=True)
```

### Step 9: Create a Template

Create a `templates` directory inside `flask_app` and add `index.html` to render the scores and graph:

```bash
mkdir templates
nano templates/index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CyberSecurity Scores</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Current CyberSecurity Platform Scores</h1>
    <ul>
        {% for score in scores %}
            <li>{{ score['platform'] }}: {{ score['score'] }}</li>
        {% endfor %}
    </ul>

    <canvas id="scoreChart" width="400" height="200"></canvas>

    <script>
        var ctx = document.getElementById('scoreChart').getContext('2d');
        var scoreChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: ['Day 1', 'Day 2', 'Day 3', 'Day 4', 'Day 5'], // Example labels
                datasets: [{
                    label: 'CyberSecurity Score',
                    data: [85, 88, 83, 90, 87], // Replace with real data
                    borderColor: 'rgba(75, 192, 192, 1)',
                    borderWidth: 1
                }]
            }
        });
    </script>
</body>
</html>
```

### Step 10: Configure Gunicorn and Nginx for Production

1. **Install Nginx**:

```bash
sudo apt install nginx -y
```

2. **Create a Gunicorn systemd Service**:
   
Create a Gunicorn service file to manage your Flask application:

```bash
sudo nano /etc/systemd/system/flask_app.service
```

Add the following content (adjust the paths to your setup):

```ini
[Unit]
Description=Gunicorn instance to serve flask_app
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/flask_app
Environment="PATH=/home/ubuntu/flask_app/venv/bin"
ExecStart=/home/ubuntu/flask_app/venv/bin/gunicorn --workers 3 --bind unix:flask_app.sock -m 007 app:app

[Install]
WantedBy=multi-user.target
```

Start and enable the service:

```bash
sudo systemctl start flask_app
sudo systemctl enable flask_app
```

3. **Configure Nginx** to Proxy Pass to Gunicorn:

Create a new Nginx server block:

```bash
sudo nano /etc/nginx/sites-available/flask_app
```

Add the following configuration (make sure to replace `example.com` with your actual domain):

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://unix:/home/ubuntu/flask_app/flask_app.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the site and test the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/flask_app /etc/nginx/sites-enabled
sudo nginx -t
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

### Step 11: Configure Domain Name and SSL

1. **Point Your Domain to Your EC2 Instance**:

In your domain registrar’s DNS settings, create an **A Record** pointing to your EC2 public IP:

```
A Record: example.com → <Your EC2 Public IP>
A Record: www.example.com → <Your EC2 Public IP>
```

2. **Install Certbot for SSL**:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run Certbot to install an SSL certificate:

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot will automatically modify your Nginx configuration to handle SSL.

### Step 12: Test Your Application

Visit **http://example.com** or **https://example.com** in your browser. You should see the current CyberSecurity platform scores and a line graph of past trends.

### Step 13: Automate Flask App Startup

Ensure your Flask app runs on boot:

```bash
sudo systemctl enable flask_app
```

## Bugs and feature requests

Have a bug or a feature request? Please first read the [issue guidelines](https://reponame/blob/master/CONTRIBUTING.md) and search for existing and closed issues. If your problem or idea is not addressed yet, [please open a new issue](https://reponame/issues/new).

## Contributing

Please read through our [contributing guidelines](https://reponame/blob/master/CONTRIBUTING.md). Included are directions for opening issues, coding standards, and notes on development.

Moreover, all HTML and CSS should conform to the [Code Guide](https://github.com/mdo/code-guide), maintained by [Main author](https://github.com/usernamemainauthor).

Editor preferences are available in the [editor config](https://reponame/blob/master/.editorconfig) for easy use in common text editors. Read more and download plugins at <https://editorconfig.org/>.

## Creators

**Creator 1**

- <https://github.com/usernamecreator1>

## Thanks

Some Text

## Copyright and license

Code and documentation copyright 2023-2024 the authors. Code released under the [MIT License](https://reponame/blob/master/LICENSE).

Enjoy :metal:
