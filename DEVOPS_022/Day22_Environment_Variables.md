Day22 — Environment Variables
TASK 1 — Understand Environment Variables

Training Area: DevOps / Docker
Time: 45 Minutes
Deliverable: Day22_Environment_Variables.md

1. What is an Environment Variable?

An environment variable is a named value provided to an application by the operating system or its execution environment.

Applications use environment variables to read configuration without putting every configuration value directly into the source code.

Basic architecture:

Application
     ↓
Environment Variable
     ↓
Configuration

For example:

APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb
API_URL=https://api.example.com

The application can read these values while it is running.

Simple Example

Instead of writing:

port = 5000

the application can read:

PORT=5000

from the environment.

This allows the same application code to run with different configurations.

For example:

Development
PORT=5000

Testing
PORT=5001

Production
PORT=8080

The application code does not need to change.

2. Why Do Applications Use Environment Variables?

Applications commonly use environment variables for configuration management.

Different environments usually require different configuration values.

For example:

Development
    ↓
DATABASE_HOST=localhost

Testing
    ↓
DATABASE_HOST=test-db

Production
    ↓
DATABASE_HOST=production-db

The application can use the same source code in all three environments.

Common Environment Variables
Variable	Purpose	Example
APP_ENV	Application environment	development
PORT	Application port	5000
DATABASE_HOST	Database hostname	database
DATABASE_PORT	Database port	5432
DATABASE_NAME	Database name	devopsdb
API_URL	External API URL	https://api.example.com
LOG_LEVEL	Logging level	INFO
3. Configuration Flow

A typical application can use environment variables like this:

Operating System / Docker / CI-CD
              │
              ↓
      Environment Variables
              │
              ↓
         Application
              │
              ↓
        Configuration
              │
              ↓
       Application Behavior

For example:

PORT=5000
        ↓
Application reads PORT
        ↓
Application starts on port 5000
4. Why Configuration Should Not Always Be Hard-Coded

Hard-coding configuration means putting values directly inside source code.

Example:

DATABASE_HOST = "production-db.example.com"
PORT = 8080
API_URL = "https://production-api.example.com"

This can create several problems.

Problem 1 — Environment Changes

Development, testing, and production environments usually have different configuration.

If values are hard-coded:

Source Code
     ↓
Production Configuration

Changing the environment may require modifying the source code.

With environment variables:

Same Application
       │
       ├── Development Variables
       ├── Testing Variables
       └── Production Variables

The application code can remain unchanged.

Problem 2 — Deployment Flexibility

Suppose an application uses:

PORT=5000

during development.

Production may require:

PORT=8080

With environment variables, the deployment environment can provide the appropriate value.

Problem 3 — Configuration Changes

If configuration is hard-coded, a small configuration change may require:

Edit Code
   ↓
Test
   ↓
Commit
   ↓
Build
   ↓
Deploy

Externalized configuration can allow configuration to be supplied separately from application code, subject to the deployment system and application design.

5. Configuration vs Secret

Configuration and secrets are related, but they are not the same thing.

Configuration

Configuration controls how an application behaves.

Examples:

APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb
LOG_LEVEL=INFO

These values are generally not secrets, although some configuration values may still be sensitive.

Secret

A secret is sensitive information that should be protected from unauthorized access.

Examples:

DATABASE_PASSWORD
API_KEY
ACCESS_TOKEN
PRIVATE_KEY

Example:

DATABASE_PASSWORD=********

Secrets require stronger protection.

They should not be committed to Git repositories or placed directly in source code.

6. Important Security Rule

Never commit real credentials or secrets to Git.

Bad example:

DATABASE_PASSWORD = "MyRealPassword123"

Another bad example:

API_KEY=real-production-api-key

inside a committed .env file.

Better approach:

Application
     ↓
Secret Management / Secure Environment
     ↓
Runtime
     ↓
Application

Depending on the environment, secrets can be supplied using approved mechanisms such as:

CI/CD secret stores
Docker/Compose secret mechanisms where appropriate
Cloud secret-management services
Kubernetes Secrets
AWS Secrets Manager
HashiCorp Vault
approved environment variables

The important principle is:

Do not expose secrets in source code or public repositories.

7. Environment Variables in Docker

Environment variables are especially useful with Docker because the same container image can be deployed with different configurations.

Example:

Docker Image
     │
     ├── Development Environment
     │       ↓
     │   APP_ENV=development
     │
     ├── Testing Environment
     │       ↓
     │   APP_ENV=testing
     │
     └── Production Environment
             ↓
         APP_ENV=production

The application image can remain the same.

8. Docker Example

Suppose a Python application needs:

APP_ENV
PORT
DATABASE_HOST
DATABASE_NAME

A Docker Compose configuration can provide these values:

services:
  app:
    image: myapp:1.0
    environment:
      APP_ENV: development
      PORT: 5000
      DATABASE_HOST: database
      DATABASE_NAME: devopsdb

The application receives these values when the container starts.

9. Using a .env File with Docker Compose

Docker Compose can also load values from a .env file.

Example:

APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb

Compose can reference them:

services:
  app:
    image: myapp:1.0
    environment:
      APP_ENV: ${APP_ENV}
      PORT: ${PORT}
      DATABASE_HOST: ${DATABASE_HOST}
      DATABASE_NAME: ${DATABASE_NAME}

This separates configuration from the Compose file.

10. .env and .env.example

A common project structure is:

project/
│
├── app/
│   └── app.py
│
├── docker-compose.yml
│
├── .env
├── .env.example
└── .gitignore
.env

Contains local configuration.

Example:

APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb
DATABASE_USER=devopsuser
DATABASE_PASSWORD=change-me

Real secret values should not be committed.

.env.example

Contains the variable names without real secrets:

APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb
DATABASE_USER=devopsuser
DATABASE_PASSWORD=

This helps another developer understand which variables are required.

11. .gitignore

The .env file containing secrets should generally be excluded from Git.

Example:

.env

Then:

Developer
    ↓
.env
    ↓
Docker Compose
    ↓
Container

while:

.env.example
    ↓
Git Repository

contains only safe example configuration.

Important

Adding .env to .gitignore does not remove a secret that was already committed to Git history.

If a real credential has been exposed, it should be treated as compromised and handled through the appropriate credential revocation/rotation process.

12. Docker Environment Architecture

A typical Docker application can look like this:

                 Docker Host
                     │
             ┌───────┴────────┐
             │                │
             ↓                ↓
        Environment       Environment
        Variables         Variables
             │                │
             └───────┬────────┘
                     ↓
                Application
                  Container
                     │
                     ↓
              Application Code

For a multi-container application:

                  Docker Compose
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
       App Service             Database Service
          │                         │
          │                         ↓
          │                    PostgreSQL
          │
          ↓
     Environment
      Variables
13. Example — Application + Database

Suppose we have:

Application
     │
     ↓
PostgreSQL

The application needs to know where the database is located.

Environment variables can provide:

DATABASE_HOST=database
DATABASE_PORT=5432
DATABASE_NAME=devopsdb
DATABASE_USER=devopsuser

Docker Compose service discovery allows the application to reach the database using its service name:

database:5432

rather than:

localhost:5432

Inside the application container, localhost refers to the application container itself.

14. Environment Variables Across Environments

One major benefit is environment-specific configuration.

Development
APP_ENV=development
DATABASE_HOST=database
DATABASE_NAME=devops_dev
LOG_LEVEL=DEBUG
Testing
APP_ENV=testing
DATABASE_HOST=test-database
DATABASE_NAME=devops_test
LOG_LEVEL=INFO
Production
APP_ENV=production
DATABASE_HOST=production-database
DATABASE_NAME=devops_prod
LOG_LEVEL=WARNING

The application can remain the same:

Same Application Image
        │
        ├── Development Configuration
        ├── Testing Configuration
        └── Production Configuration
15. Environment Variables in CI/CD

Environment variables are also commonly used in CI/CD pipelines.

Example:

Developer
    ↓
Git Push
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
Environment Variables / Secrets
    ↓
Docker Build
    ↓
Deployment

Possible configuration:

APP_ENV=production
API_URL=https://api.example.com

Sensitive values should be stored using the CI/CD platform's protected secret mechanism rather than committed to the repository.

16. Environment Variables in DevOps

Environment variables are useful throughout the DevOps lifecycle.

Development
     ↓
Testing
     ↓
CI/CD
     ↓
Docker
     ↓
Cloud Deployment
     ↓
Production

Each environment can provide appropriate configuration.

For example:

Development
DATABASE_HOST=localhost

Docker
DATABASE_HOST=database

Production
DATABASE_HOST=prod-db.example.com

The application does not necessarily need different source code for each environment.

17. Advantages of Environment Variables
1. Configuration Separation

Configuration can be separated from application source code.

2. Environment Flexibility

The same application can run in development, testing, and production.

3. Easier Deployment

Deployment systems can provide environment-specific values.

4. Container Support

Docker and container platforms can inject runtime configuration.

5. CI/CD Support

Pipelines can provide configuration and protected secrets at deployment time.

6. Reduced Hard-Coding

Developers do not need to embed environment-specific values directly in source code.

18. Limitations and Security Considerations

Environment variables are useful, but they are not automatically secure.

Potential risks include:

accidental exposure in logs
accidental exposure in debugging output
incorrect container configuration
insecure CI/CD configuration
unauthorized access to the process/environment
committing .env files to Git
passing secrets through unsafe command-line history

Therefore:

Environment Variable
        ≠
Automatically Secure Secret

Sensitive values should use an appropriate secret-management mechanism.

19. Hard-Coded vs Environment-Based Configuration
Feature	Hard-Coded	Environment Variables
Configuration location	Source code	Runtime environment
Environment changes	Often requires code changes	Usually configuration change
Reusability	Lower	Higher
Docker support	Possible	Excellent
CI/CD support	Less flexible	Strong
Secret handling	Poor if embedded	Better when combined with secret management
Deployment flexibility	Lower	Higher
20. Real-World Example

Imagine a web application:

Frontend
    ↓
Backend API
    ↓
PostgreSQL Database

The backend may require:

APP_ENV=production
PORT=5000
DATABASE_HOST=database
DATABASE_PORT=5432
DATABASE_NAME=applicationdb
DATABASE_USER=appuser
DATABASE_PASSWORD=<secret>
API_URL=https://api.example.com

The architecture becomes:

                 Deployment Environment
                         │
                         ↓
                Environment Variables
                         │
                         ↓
                  Backend Container
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
          API Server            Database
              │                     │
              └──── Configuration ──┘
21. Practical Exercise
Step 1 — Create a .env file

Example:

APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb

Do not place real production credentials in this training file.

Step 2 — Create .env.example
APP_ENV=development
PORT=5000
DATABASE_HOST=database
DATABASE_NAME=devopsdb
DATABASE_USER=
DATABASE_PASSWORD=
Step 3 — Add .env to .gitignore
.env
Step 4 — Use Variables in Docker Compose

Example:

services:
  app:
    image: myapp:1.0
    environment:
      APP_ENV: ${APP_ENV}
      PORT: ${PORT}
      DATABASE_HOST: ${DATABASE_HOST}
      DATABASE_NAME: ${DATABASE_NAME}
Step 5 — Validate the Compose Configuration

Run:

docker compose config

This helps verify that Docker Compose can resolve the configuration.

For a quieter validation:

docker compose config --quiet

If there is no error, the Compose configuration is syntactically valid.

22. Verification Checklist

Check the following:

[ ] I understand what an environment variable is.

[ ] I understand APP_ENV.

[ ] I understand PORT.

[ ] I understand DATABASE_HOST.

[ ] I understand DATABASE_NAME.

[ ] I understand the difference between configuration and secrets.

[ ] I understand why hard-coded configuration can be problematic.

[ ] I understand how Docker can provide environment variables.

[ ] I understand .env files.

[ ] I understand .env.example.

[ ] I understand why .env should normally be excluded from Git when it contains secrets.

[ ] I understand that environment variables are not automatically secure.

[ ] I understand how environment-specific configuration works.

[ ] I can validate a Docker Compose configuration.
23. Key Interview Questions
Q1. What is an environment variable?

An environment variable is a named runtime value used by an application to obtain configuration from its execution environment.

Q2. Why do applications use environment variables?

They allow configuration to be supplied outside the source code and make the same application easier to run across different environments.

Q3. Why should configuration not always be hard-coded?

Hard-coded configuration reduces flexibility and can require source-code changes when deploying to different environments.

Q4. What is the difference between configuration and a secret?

Configuration controls application behavior, while a secret is sensitive information such as a password, API key, or access token that requires protection.

Q5. Why are environment variables useful in Docker?

Docker can provide environment-specific configuration to containers at runtime, allowing the same image to be used in different environments.

24. Important DevOps Concept

Remember:

Application Code
       │
       │ stays mostly the same
       ↓
Environment Configuration
       │
       ├── Development
       ├── Testing
       └── Production

This is one of the important principles of configuration externalization.

25. Summary

Environment variables provide applications with configuration values at runtime.

Common examples include:

APP_ENV
PORT
DATABASE_HOST
DATABASE_NAME
API_URL

They help separate application code from environment-specific configuration.

Docker makes environment variables particularly useful because the same container image can be deployed with different runtime configurations.

However:

Configuration
      ≠
Secret

and:

Environment Variable
      ≠
Automatically Secure

Sensitive information should be protected using appropriate secret-management practices.

The key DevOps principle is:

Build Once
    ↓
Configure at Runtime
    ↓
Deploy to Different Environments