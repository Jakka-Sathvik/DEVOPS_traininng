TASK 3 — Build a Small Application

For this task, I recommend using Python + Flask because it is simple, lightweight, and easy to containerize with Docker later.

Step 1 — Enter the Project

From your final project:

cd devops-docker-final-project

Create the application directory:

mkdir -p app
cd app
Step 2 — Create app.py
nano app.py

Add:

from flask import Flask, jsonify

app = Flask(__name__)


@app.route("/")
def home():
    return """
    <html>
        <head>
            <title>DevOps Training Application</title>
        </head>
        <body>
            <h1>DevOps Training Application</h1>

            <p><strong>Application Status:</strong> Running</p>
            <p><strong>Environment:</strong> Development</p>
            <p><strong>Database:</strong> Connected</p>

            <p>Health Check: <a href="/health">/health</a></p>
        </body>
    </html>
    """


@app.route("/health")
def health():
    return jsonify({
        "status": "success",
        "application": "running",
        "database": "connected"
    }), 200


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

Save with:

Ctrl + O → Enter → Ctrl + X

Step 3 — Create requirements.txt

Inside the app directory:

nano requirements.txt

Add:

Flask

Save and exit.

Your application now looks like:

devops-docker-final-project/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── scripts/
└── docs/
Step 4 — Install Flask

For a quick local test:

pip3 install flask

If Ubuntu prevents system-wide installation, use a virtual environment:

python3 -m venv venv
source venv/bin/activate
pip install flask
Step 5 — Run the Application

From the app directory:

python3 app.py

You should see something similar to:

* Running on http://127.0.0.1:5000
* Running on http://172.x.x.x:5000

The application is now running on port 5000.

Step 6 — Test /

Open:

http://localhost:5000/

You should see:

DevOps Training Application

Application Status: Running

Environment: Development

Database: Connected

Health Check: /health
Step 7 — Test /health

Open:

http://localhost:5000/health

Expected response:

{
  "application": "running",
  "database": "connected",
  "status": "success"
}

The HTTP response should be 200 OK.

You can also test it from the terminal:

curl http://localhost:5000/health

Expected:

{"application":"running","database":"connected","status":"success"}
Step 8 — Test Both Endpoints
Home endpoint
curl http://localhost:5000/
Health endpoint
curl http://localhost:5000/health

Both should respond successfully.

Step 9 — Stop the Application

Go to the terminal where Flask is running and press:

Ctrl + C
Step 10 — Commit the Application

Go back to the project root:

cd ..

Check:

git status

You should see:

app/app.py
app/requirements.txt

Stage:

git add app/

Commit:

git commit -m "Added Flask training application"

Push:

git push
Final Project Structure
devops-docker-final-project/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── scripts/
└── docs/
Task 3 Deliverables

You should capture screenshots of:

Application running

python3 app.py

Home page

http://localhost:5000/

Health endpoint

http://localhost:5000/health

Successful Git commit

git log --oneline -1

This application is now ready for the next Docker task, where you can create a Dockerfile, build the image, and run the application inside a container.