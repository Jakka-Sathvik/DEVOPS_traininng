TASK 6 — Create an Application Health Endpoint

Training Area: DevOps / Application Monitoring
Time: 1 Hour
Application: Day 21 DevOps Training Application
Endpoint: GET /health
Expected HTTP Status: 200 OK
Deliverable: Health_Check_Documentation.md

1. Objective

The objective of this task is to create a simple application health endpoint that can be used to determine whether the application is running.

The endpoint will be:

GET /health

Expected response:

{
  "status": "healthy"
}

The health endpoint should be:

simple
fast
safe
easy to test
suitable for monitoring
free from sensitive information
2. What Is a Health Endpoint?

A health endpoint is an HTTP endpoint that allows a monitoring system, load balancer, container platform, or DevOps engineer to check whether an application is responding.

Basic architecture:

Application
     ↓
  /health
     ↓
  HTTP Request
     ↓
  HTTP Response
     ↓
   200 OK
     ↓
  healthy

For this task, the endpoint performs a simple application-level check.

3. Why Do We Need a Health Endpoint?

A running process does not always mean that the application is healthy.

For example:

Container Running
       ↓
Application Process Running
       ↓
/health
       ↓
200 OK

A health endpoint gives another system a simple way to check application availability.

Health endpoints can be used by:

Docker
Kubernetes
load balancers
monitoring systems
CI/CD deployment checks
DevOps engineers
uptime monitoring tools
4. Existing Day 21 Application

The Day 21 application is a simple Flask application.

Example:

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

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

We will add the health endpoint to this application.

5. Add the /health Endpoint

Update app.py:

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
            <p>Health Check: <a href="/health">/health</a></p>
        </body>
    </html>
    """


@app.route("/health")
def health():
    return jsonify({
        "status": "healthy"
    }), 200


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
6. Understanding the Code

The route:

@app.route("/health")

creates the HTTP endpoint:

/health

The function:

def health():

runs when the endpoint is requested.

The response:

return jsonify({
    "status": "healthy"
}), 200

returns JSON with HTTP status code 200.

7. Expected Response

When the application receives:

GET /health

it should return:

{
  "status": "healthy"
}

HTTP response:

HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "healthy"
}
8. Health Check Architecture

The required architecture is:

Application
     ↓
 /health
     ↓
 200 OK
     ↓
 healthy

A more detailed version:

                  Client / Monitor
                         │
                         │ GET /health
                         ↓
                 Flask Application
                         │
                         ↓
                  Health Endpoint
                         │
                         ↓
                    200 OK
                         │
                         ↓
                  {"status":
                   "healthy"}
9. Why Keep the Endpoint Simple?

A basic health endpoint should respond quickly.

For this task, it should not perform complicated operations such as:

/health
   ↓
Database Query
   ↓
External API
   ↓
Authentication Service
   ↓
Multiple Network Calls

Instead:

/health
   ↓
Application Responding?
   ↓
200 OK

This makes the endpoint fast and reliable.

10. What Must NOT Be Exposed?

The health endpoint must not return sensitive information.

Never return:

Passwords
Tokens
Database Credentials
Internal Secrets
Private Keys
API Keys

Bad example:

{
  "status": "healthy",
  "database_password": "password123",
  "api_token": "secret-token"
}

This is insecure.

11. Correct Response

The correct response is intentionally simple:

{
  "status": "healthy"
}

It does not expose internal implementation details.

12. Local Testing

Start the Flask application:

python app.py

Expected output will be similar to:

* Running on http://127.0.0.1:5000

The exact startup message may vary depending on the Flask version and environment.

13. Test Using a Browser

Open:

http://127.0.0.1:5000/health

Expected:

{
  "status": "healthy"
}

The browser should display the JSON response.

14. Test Using curl

Open another terminal and run:

curl http://127.0.0.1:5000/health

Expected response:

{
  "status": "healthy"
}

The exact formatting may vary.

15. Verify HTTP Status Code

On systems where curl supports it, run:

curl -i http://127.0.0.1:5000/health

Expected output will contain:

HTTP/1.1 200 OK
Content-Type: application/json

followed by:

{
  "status": "healthy"
}

The important result is:

HTTP Status: 200
Status: healthy
16. Test Flow

The complete local test is:

Browser / curl
      │
      │ GET /health
      ↓
Flask Application
      │
      ↓
Health Endpoint
      │
      ↓
HTTP 200
      │
      ↓
{
  "status": "healthy"
}
17. Docker Testing

If the Day 21 application is containerized, build and start the application:

docker compose up -d --build

Check the container:

docker compose ps

The application should be running.

If the application exposes port 5000, test:

curl http://localhost:5000/health

Expected:

{
  "status": "healthy"
}
18. Docker Architecture
                Host Machine
                     │
                     │ localhost:5000
                     ↓
             Docker Container
                     │
                     ↓
              Flask Application
                     │
                     ↓
                  /health
                     │
                     ↓
                  200 OK
                     │
                     ↓
             {"status":"healthy"}
19. Why Health Checks Are Useful in DevOps

Health endpoints can be used by automated systems.

Example:

Load Balancer
      │
      ↓
GET /health
      │
 ┌────┴────┐
 ↓         ↓
200       Error
 │         │
 ↓         ↓
Healthy   Unhealthy

An orchestration or monitoring platform can use health information as part of its operational decisions.

20. Health Endpoint vs Application Page

The home page:

GET /

may return:

DevOps Training Application
Application Status: Running

The health endpoint:

GET /health

returns:

{
  "status": "healthy"
}

The health endpoint is designed to provide a simple machine-readable check.

21. Health Check vs Detailed Diagnostics

A basic health check should not expose unnecessary internal information.

Basic Health Check
GET /health
        ↓
200 OK
        ↓
healthy
Detailed Diagnostic Endpoint

A detailed diagnostic endpoint, if required, may need authentication and stronger access controls.

It could potentially inspect:

Database
External APIs
Dependencies
Application internals

But such diagnostics should not be unnecessarily exposed publicly.

For this task, only the simple /health endpoint is required.

22. Security Considerations

The health endpoint should not expose:

passwords
API keys
access tokens
database credentials
private keys
environment variables
internal secrets

Avoid responses such as:

{
  "status": "healthy",
  "environment_variables": "...",
  "database_password": "...",
  "api_token": "..."
}

Use:

{
  "status": "healthy"
}
23. Expected Behavior
Healthy Application
GET /health
     ↓
Application responds
     ↓
HTTP 200
     ↓
{"status":"healthy"}
Application Not Running
GET /health
     ↓
No application response
     ↓
Connection failure

The monitoring system can then recognize that the application is unavailable.

24. Troubleshooting
Problem 1 — Connection Refused

Example:

curl: Failed to connect

Check whether Flask is running:

python app.py
Problem 2 — 404 Not Found

If /health returns:

404 Not Found

check that the route exists:

@app.route("/health")
def health():
    return jsonify({
        "status": "healthy"
    }), 200

Also verify that the application was restarted after changing the code.

Problem 3 — Import Error

If you see:

ModuleNotFoundError: No module named 'flask'

install the project's dependencies using the appropriate environment/requirements file.

For example:

pip install -r requirements.txt
Problem 4 — Wrong Port

If the application is running on:

127.0.0.1:5000

but you test:

localhost:8080

the request will fail unless port 8080 is actually mapped to the application.

Verify the application's configured port and, if using Docker, its port mapping.

25. Verification Checklist
[ ] Added /health endpoint.

[ ] Endpoint accepts GET requests.

[ ] Endpoint returns JSON.

[ ] Response contains status=healthy.

[ ] HTTP status is 200.

[ ] Tested locally.

[ ] Tested with browser or curl.

[ ] Tested inside Docker if applicable.

[ ] No passwords are exposed.

[ ] No API tokens are exposed.

[ ] No database credentials are exposed.

[ ] No private keys are exposed.

[ ] No internal secrets are exposed.

[ ] Application was restarted after code changes.

[ ] Health endpoint responds successfully.
26. Suggested Screenshots

For the training record, capture:

Screenshot 1 — Application Running
[SCREENSHOT]
Flask application running on port 5000
Screenshot 2 — Browser Health Check
[SCREENSHOT]
http://localhost:5000/health

{
  "status": "healthy"
}
Screenshot 3 — curl Test
[SCREENSHOT]
curl -i http://localhost:5000/health

HTTP/1.1 200 OK
...
{
  "status": "healthy"
}
Screenshot 4 — Docker Test

If using Docker:

[SCREENSHOT]
docker compose ps

curl http://localhost:5000/health
27. Interview Questions
Q1. What is a health endpoint?

A health endpoint is an HTTP endpoint that provides a simple way to determine whether an application is responding.

Q2. What endpoint did you create?
GET /health
Q3. What HTTP status should a healthy application return?
200 OK
Q4. What should the health endpoint return?
{
  "status": "healthy"
}
Q5. Should a health endpoint expose passwords?

No. It should never expose passwords, tokens, credentials, private keys, or other secrets.

Q6. Why are health endpoints useful in DevOps?

Monitoring systems, load balancers, containers, and orchestration platforms can use them as a simple application availability signal.

28. What I Learned

During this task, I learned:

How to create a Flask health endpoint.
How GET /health works.
How to return JSON from Flask.
How to return HTTP status code 200.
How to test an endpoint locally.
How to test an endpoint using curl.
How health endpoints can support monitoring.
Why health endpoints should remain simple.
Why passwords, tokens, and secrets must never be exposed.
How a Dockerized application can expose a health endpoint.
The difference between a simple health check and detailed diagnostics.
Conclusion

A health endpoint provides a simple and machine-readable way to check application availability.

For this task, the application exposes:

GET /health

and returns:

{
  "status": "healthy"
}

with:

HTTP 200 OK

The complete flow is:

Application
    ↓
/health
    ↓
200 OK
    ↓
healthy