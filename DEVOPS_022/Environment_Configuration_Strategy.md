TASK 2 — Development vs Production Configuration

Training Area: DevOps / Docker
Time: 45 Minutes
Deliverable: Environment_Configuration_Strategy.md

1. Objective

The objective of this task is to understand how the same application can run in different environments while using different configuration values.

The core concept is:

Same Application
       ↓
Different Environment
       ↓
Different Configuration

The application source code can remain mostly the same while configuration changes between development, testing, staging, and production.

2. Development vs Production

Development and production environments have different purposes.

Development Environment

Used by developers for:

coding
testing
debugging
local development
experimenting with features
troubleshooting
Production Environment

Used by real users and business workloads.

It requires:

stronger security
controlled logging
reliable infrastructure
production databases
production APIs
protected secrets
controlled debugging
3. Configuration Comparison
Configuration	Development	Production
Environment	development	production
Debug	Enabled/controlled	Disabled
Database	Training DB	Production DB
API URL	Development URL	Production URL
Logging	Detailed	Controlled
Secrets	Test values	Secure secrets
4. Environment

The environment identifies where the application is running.

Development
APP_ENV=development

This tells the application that it is running in a development environment.

Production
APP_ENV=production

This tells the application that it is running in production.

Example:

Same Application
       │
       ├── APP_ENV=development
       │
       └── APP_ENV=production

The application can change its behavior based on the environment when that behavior is intentionally designed and controlled.

5. Debug Configuration

Debugging is useful during development because developers need detailed information when troubleshooting problems.

Development

Example:

DEBUG=true

or:

DEBUG=1

The exact value and behavior depend on the application framework.

Development may provide:

detailed error information
stack traces
development logs
debugging tools
automatic reload features
Production

Production should normally use:

DEBUG=false

or the framework's equivalent secure production setting.

Production applications should avoid exposing:

stack traces
source-code details
internal paths
database information
credentials
sensitive configuration
Important

Debug should never be enabled blindly in production.

6. Database Configuration

Development and production should generally use different databases.

Development

Example:

DATABASE_HOST=localhost
DATABASE_NAME=training_db
DATABASE_USER=training_user

A developer can use a training or local database.

Example architecture:

Developer
    ↓
Application
    ↓
Training Database
Production

Example:

DATABASE_HOST=production-db.example.com
DATABASE_NAME=production_db
DATABASE_USER=application_user

Architecture:

Real Users
    ↓
Production Application
    ↓
Production Database
Important Security Rule

Production data should not be casually copied into development environments.

Production databases usually contain real business or customer information and require appropriate security and access controls.

7. API URL Configuration

Applications often communicate with external APIs.

The API URL can change depending on the environment.

Development
API_URL=https://dev-api.example.com

Architecture:

Development Application
          ↓
Development API
Production
API_URL=https://api.example.com

Architecture:

Production Application
          ↓
Production API

The application source code can remain the same while the runtime configuration changes.

8. Logging Configuration

Logging is important in both environments, but the amount of detail can be different.

Development — Detailed Logging

Example:

LOG_LEVEL=DEBUG

Development logs may contain:

Application started
Database connection initialized
Request received
API request sent
Function executed
Debug information

Detailed logs help developers understand application behavior.

Production — Controlled Logging

Example:

LOG_LEVEL=INFO

or:

LOG_LEVEL=WARNING

depending on the application's logging strategy.

Production logging should be useful without unnecessarily exposing sensitive information.

9. Secrets

Secrets are sensitive values such as:

database passwords
API keys
access tokens
private keys
service credentials

Development and production should not normally use the same secrets.

Development

A developer may use a test credential:

DATABASE_PASSWORD=test-password

This is only an example. Real credentials should still be protected.

Production

Production should use a protected secret:

DATABASE_PASSWORD=<managed-secure-secret>

Production secrets should be stored using an approved secret-management solution rather than committed to source control.

Examples include:

CI/CD secret stores
cloud secret-management services
Docker/Kubernetes secret mechanisms where appropriate
HashiCorp Vault
AWS Secrets Manager
10. Complete Environment Examples
Development Configuration

Example .env:

APP_ENV=development
DEBUG=true

DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=training_db
DATABASE_USER=training_user
DATABASE_PASSWORD=<development-secret>

API_URL=https://dev-api.example.com

LOG_LEVEL=DEBUG

This configuration is designed for development and testing.

Production Configuration

Example conceptual configuration:

APP_ENV=production
DEBUG=false

DATABASE_HOST=production-db.example.com
DATABASE_PORT=5432
DATABASE_NAME=production_db
DATABASE_USER=application_user
DATABASE_PASSWORD=<production-secret>

API_URL=https://api.example.com

LOG_LEVEL=INFO

The actual production secret should come from an approved secure mechanism.

11. Same Application, Different Configuration

The most important concept is:

                    Same Application
                           │
             ┌─────────────┴─────────────┐
             ↓                           ↓
       Development                  Production
             │                           │
             ↓                           ↓
   Development Config          Production Config
             │                           │
             ↓                           ↓
       Training DB                 Production DB
       Dev API                     Production API
       Debug ON                    Debug OFF
       Detailed Logs               Controlled Logs
       Test Secrets                Secure Secrets

The application code does not need to contain separate hard-coded versions for every environment.

12. Configuration Strategy

A good environment configuration strategy separates:

Application Code
       +
Environment Configuration
       +
Secret Management

Architecture:

                  Application Source Code
                           │
                           ↓
                    Build Application
                           │
                           ↓
                      Docker Image
                           │
            ┌──────────────┴──────────────┐
            ↓                             ↓
       Development                    Production
            │                             │
            ↓                             ↓
   Development Config           Production Config
            │                             │
            ↓                             ↓
      Training DB                  Production DB
      Development API             Production API
      Debug Controlled             Debug Disabled
      Detailed Logs                Controlled Logs
      Test Secrets                 Secure Secrets
13. Docker Example

Docker makes this strategy easier because the same image can be used with different environment variables.

Example:

services:
  app:
    image: myapp:1.0
    environment:
      APP_ENV: ${APP_ENV}
      DEBUG: ${DEBUG}
      DATABASE_HOST: ${DATABASE_HOST}
      DATABASE_NAME: ${DATABASE_NAME}
      API_URL: ${API_URL}
      LOG_LEVEL: ${LOG_LEVEL}

Development configuration:

APP_ENV=development
DEBUG=true
DATABASE_HOST=database
DATABASE_NAME=training_db
API_URL=https://dev-api.example.com
LOG_LEVEL=DEBUG

Production deployment can provide different values:

APP_ENV=production
DEBUG=false
DATABASE_HOST=production-db.example.com
DATABASE_NAME=production_db
API_URL=https://api.example.com
LOG_LEVEL=INFO

The Docker image itself can remain the same.

14. Build Once, Configure at Runtime

A useful DevOps principle is:

Source Code
     ↓
Build
     ↓
Application Image
     ↓
Deploy
     ↓
Runtime Configuration

Instead of creating:

myapp-development
myapp-testing
myapp-production

only because configuration differs, a deployment strategy can use one application image and supply environment-specific runtime configuration.

For example:

myapp:1.0

can be deployed with:

Development → development configuration
Testing     → testing configuration
Production  → production configuration
15. Why Configuration Should Be Environment-Specific

Different environments have different requirements.

Requirement	Development	Production
Purpose	Development/testing	Real workload
Database	Training/local	Production
API	Development API	Production API
Debugging	More detailed	Restricted
Logging	Detailed	Controlled
Secrets	Test/development	Protected production secrets
Access	Developer access	Restricted access
Security	Development controls	Strong production controls
16. What Happens If the Wrong Configuration Is Used?

Incorrect environment configuration can cause serious problems.

Example 1 — Production Uses Development Database
Production Application
        ↓
Development Database

This can cause:

incorrect data access
data corruption
unexpected application behavior
Example 2 — Production Debug Enabled
Production
    ↓
DEBUG=true

This may expose sensitive internal information.

Example 3 — Development Uses Production Database
Developer
    ↓
Production Database

This can be dangerous because development testing could affect real production data.

Example 4 — Production Uses Development API
Production Application
        ↓
Development API

Users may receive incorrect or incomplete information.

17. Recommended Environment Separation

A typical DevOps setup is:

Development
    │
    ├── Development Application
    ├── Training Database
    ├── Development API
    └── Development Configuration

Testing / Staging
    │
    ├── Test Application
    ├── Test Database
    ├── Test API
    └── Test Configuration

Production
    │
    ├── Production Application
    ├── Production Database
    ├── Production API
    └── Production Configuration

This reduces the risk of accidentally mixing environments.

18. Security Best Practices
1. Never hard-code production passwords

Bad:

DATABASE_PASSWORD = "production-password"
2. Do not commit real secrets

Avoid committing:

.env

when it contains real secrets.

3. Use .env.example

Example:

DATABASE_HOST=
DATABASE_NAME=
DATABASE_USER=
DATABASE_PASSWORD=
4. Use secret management

Production secrets should be managed using approved secure systems.

5. Disable unnecessary debugging

Production:

DEBUG=false
6. Restrict production access

Only authorized users and services should access production resources.

7. Keep environments separated

Do not casually connect development applications to production resources.

19. Practical Exercise

Create two conceptual configuration files.

Development
APP_ENV=development
DEBUG=true
DATABASE_HOST=database
DATABASE_NAME=training_db
API_URL=https://dev-api.example.com
LOG_LEVEL=DEBUG
Production
APP_ENV=production
DEBUG=false
DATABASE_HOST=production-db.example.com
DATABASE_NAME=production_db
API_URL=https://api.example.com
LOG_LEVEL=INFO

Compare them.

Observe that:

Application Code
       ↓
Same

while:

Configuration
       ↓
Different
20. Verification Checklist
[ ] I understand development environment.

[ ] I understand production environment.

[ ] I understand APP_ENV.

[ ] I understand why DEBUG should normally be disabled in production.

[ ] I understand the difference between training and production databases.

[ ] I understand development vs production API URLs.

[ ] I understand detailed vs controlled logging.

[ ] I understand development/test secrets vs production secrets.

[ ] I understand environment-specific configuration.

[ ] I understand how Docker can use different runtime configuration.

[ ] I understand the "same application, different configuration" principle.

[ ] I understand why production resources should be protected.

[ ] I understand why production data should not be casually used for development.
21. Interview Questions
Q1. What is the difference between development and production configuration?

Development configuration is optimized for coding, testing, and debugging, while production configuration is designed for real workloads, security, reliability, and controlled operation.

Q2. Why should debug mode normally be disabled in production?

Because detailed debugging information can expose internal application details and potentially sensitive information.

Q3. Can the same Docker image be used in development and production?

Yes. A common DevOps approach is to use the same application image while supplying different runtime configuration for each environment.

Q4. Why should development and production databases be separated?

To prevent development/testing activities from accidentally affecting real production data.

Q5. Why should production secrets be protected?

Production secrets can provide access to sensitive systems and data. Unauthorized disclosure can result in security incidents.

22. Key Concept

The most important lesson from this task is:

Same Application
       ↓
Different Environment
       ↓
Different Configuration

For example:

                 Same Application
                        │
             ┌──────────┴──────────┐
             ↓                     ↓
       Development             Production
             │                     │
             ↓                     ↓
      APP_ENV=development   APP_ENV=production
      DEBUG=true            DEBUG=false
      Training DB           Production DB
      Dev API               Production API
      Detailed Logs         Controlled Logs
      Test Secrets          Secure Secrets
23. What I Learned

During this task, I learned:

The difference between development and production environments.
How environment variables control application configuration.
Why debug settings differ between environments.
Why databases should be separated.
Why API URLs change between environments.
Why development logging can be more detailed.
Why production logging should be controlled.
The difference between test values and production secrets.
How Docker supports environment-specific configuration.
Why the same application can use different configuration in different environments.
Why production configuration requires stronger security controls.
Conclusion

A well-designed DevOps environment separates application code from environment-specific configuration.

The application should be able to run in different environments without unnecessary source-code changes.

The overall strategy is:

                 Same Application
                        ↓
                Build Application
                        ↓
                  Docker Image
                        ↓
          ┌─────────────┴─────────────┐
          ↓                           ↓
    Development                  Production
          ↓                           ↓
 Dev Configuration            Prod Configuration
          ↓                           ↓
 Training DB                  Production DB
 Dev API                      Production API
 Debug Controlled             Debug Disabled
 Detailed Logs                Controlled Logs
 Test Secrets                 Secure Secrets

