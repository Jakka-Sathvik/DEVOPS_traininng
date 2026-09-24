TASK 4 — Understand CloudFront Origin

Time: 45 Minutes
Topic: CloudFront Origin
Training: DevOps Engineer Training

1. Objective

The objective of this task is to understand the role of an Origin in Amazon CloudFront and how CloudFront communicates with an S3 bucket when a user requests content.

The basic architecture is:

CloudFront
    ↓
Origin
    ↓
S3

In this exercise, Amazon S3 is used as the origin that stores the website files delivered through CloudFront.

2. What is an Origin?

An Origin is the location where CloudFront retrieves the original content when the requested content is not already available in the CloudFront cache.

The origin is the source of the content that CloudFront distributes to users.

Examples of CloudFront origins include:

Amazon S3 bucket
Application Load Balancer
EC2-hosted application
API endpoint
Custom HTTP/HTTPS server

For this training exercise:

CloudFront
    ↓
S3 Origin
    ↓
Website Files

The S3 bucket contains the actual files, such as:

index.html
style.css
app.js
logo.png
images/

CloudFront sits between the user and the origin.

3. Why is S3 Being Used as the Origin?

Amazon S3 is being used as the CloudFront origin because S3 is designed to provide highly durable and scalable object storage.

A static website may contain files such as:

HTML
CSS
JavaScript
Images
Videos
Documents
Static JSON files

These files can be stored in S3 and delivered through CloudFront.

Architecture
                    AWS
                     │
                     ↓
              CloudFront CDN
                     │
                     ↓
                 S3 Origin
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      index.html  style.css   app.js
Benefits of using S3 as an origin
Benefit	Explanation
Object storage	S3 stores website files as objects
Scalability	S3 can handle large amounts of stored content
Durability	S3 is designed for highly durable storage
Integration	S3 integrates directly with CloudFront
Cost efficiency	Static files can be stored without running a web server
Security	S3 can remain private when configured with CloudFront access controls
Versioning	S3 Versioning can help retain previous object versions
4. What is the Relationship Between CloudFront and S3?

CloudFront and S3 have different responsibilities.

S3
↓
Stores the content

CloudFront
↓
Delivers the content
S3

S3 is the storage layer.

Example:

S3 Bucket
   │
   ├── index.html
   ├── style.css
   ├── app.js
   └── logo.png
CloudFront

CloudFront is the content delivery layer.

It receives requests from users and determines whether the requested content can be served from its cache or needs to be retrieved from the origin.

User
  ↓
CloudFront
  ↓
S3

Therefore:

S3 stores the website files, while CloudFront delivers those files efficiently to users.

5. What Happens When CloudFront Receives a Request?

When a browser requests content through a CloudFront distribution, CloudFront checks whether the requested object is already available in its cache.

The simplified flow is:

Browser
   ↓
CloudFront
   ↓
Cache?
 ┌─┴─┐
Yes  No
 │    │
 ↓    ↓
Return  S3
        ↓
     Response
        ↓
    CloudFront
        ↓
       User

There are two main scenarios:

Cache Hit
Cache Miss
6. Scenario 1 — Cache Hit

A cache hit occurs when CloudFront already has the requested content available in its cache.

Example:

User requests:
index.html

CloudFront checks its cache.

Browser
   ↓
CloudFront
   ↓
Is index.html cached?
   ↓
  YES
   ↓
Return cached object
   ↓
Browser

The request does not need to retrieve the object from S3.

Benefits

Cache hits can:

reduce origin requests
reduce latency
improve website performance
reduce repeated data retrieval from the origin
improve scalability for frequently requested content
7. Scenario 2 — Cache Miss

A cache miss occurs when CloudFront does not have the requested content available in its cache.

Example:

User requests:
logo.png

CloudFront checks its cache.

Browser
   ↓
CloudFront
   ↓
Is logo.png cached?
   ↓
  NO
   ↓
Request from S3
   ↓
S3 returns logo.png
   ↓
CloudFront receives object
   ↓
CloudFront returns object to User

CloudFront can also cache the retrieved object according to the distribution's caching configuration, so subsequent requests may be served from cache.

8. Complete Request Flow

The complete simplified request flow is:

                         User
                           │
                           ↓
                       Browser
                           │
                           ↓
                    CloudFront CDN
                           │
                           ↓
                      Cache Check
                       /       \
                     YES        NO
                      │          │
                      ↓          ↓
               Cached Object     S3
                      │           │
                      │           ↓
                      │       Website File
                      │           │
                      │           ↓
                      │      CloudFront
                      │           │
                      └─────┬─────┘
                            ↓
                           User
9. Step-by-Step Example

Suppose the S3 bucket contains:

my-training-bucket
│
├── index.html
├── style.css
└── logo.png

CloudFront is configured with this S3 bucket as its origin.

A user opens the CloudFront URL.

Step 1 — Browser sends request
Browser
   ↓
GET /index.html
Step 2 — Request reaches CloudFront
Browser
   ↓
CloudFront
Step 3 — CloudFront checks cache

CloudFront determines whether index.html is already available in the relevant cache.

Step 4A — Cache Hit

If available:

CloudFront Cache
      ↓
index.html
      ↓
Browser
Step 4B — Cache Miss

If not available:

CloudFront
    ↓
S3 Origin
    ↓
index.html
    ↓
CloudFront
    ↓
Browser

CloudFront may then cache the retrieved object according to its cache behavior.

10. What is an S3 Origin?

An S3 origin means that CloudFront uses an Amazon S3 bucket as the source from which it retrieves content.

Example:

CloudFront Distribution
          │
          ↓
     S3 Origin
          │
          ↓
   Training Bucket
          │
          ├── index.html
          ├── CSS
          ├── JavaScript
          └── Images

The S3 bucket is therefore the origin for the CloudFront distribution.

11. Private S3 Origin

For a secure architecture, the S3 bucket does not necessarily need to be publicly accessible.

A common modern architecture is:

User
  │
  ↓
CloudFront
  │
  │ Authorized access
  ↓
Private S3 Bucket

CloudFront can be configured to access a private S3 origin using Origin Access Control (OAC).

This allows:

users to access content through CloudFront
S3 to remain private
direct public access to the bucket to be blocked
Security principle
Internet User
      ↓
CloudFront
      ↓
Authorized Access
      ↓
Private S3

This is generally preferable to making a private training bucket publicly readable just to test CloudFront.

12. Origin vs Cache

It is important not to confuse an origin with a cache.

Component	Purpose
Origin	Source of the original content
CloudFront cache	Temporarily stores content for faster delivery
S3	Stores the actual objects
Browser	Requests and receives content

Example:

S3
 │
 │ Original content
 ↓
CloudFront
 │
 │ Cached copy
 ↓
User

The cached copy does not replace S3 as the origin.

13. Real-World Example

Consider an online shopping website.

The website has:

index.html
products.html
products.js
logo.png
product-images/

These static files can be stored in S3.

Architecture:

                    Internet Users
                    /     |      \
                   /      |       \
                  ↓       ↓        ↓
             CloudFront Distribution
                    │
              Edge Locations
                    │
                    ↓
                S3 Origin
                    │
                    ↓
              Website Files

Users can receive frequently requested content from CloudFront's cache instead of repeatedly retrieving it from S3.

14. CloudFront Origin Request Flow

A simplified sequence is:

1. User requests content
          ↓
2. Request reaches CloudFront
          ↓
3. CloudFront checks cache
          ↓
      ┌───┴───┐
      ↓       ↓
    Hit      Miss
      │       │
      ↓       ↓
   Return    Request
   cached    content
   content   from origin
              ↓
             S3
              ↓
         Return object
              ↓
          CloudFront
              ↓
             User
15. Important CloudFront Terms
Term	Meaning
CloudFront	AWS content delivery network
Origin	Source from which CloudFront retrieves content
S3 Origin	S3 bucket configured as the content source
Edge Location	Distributed location used by CloudFront to serve/process content closer to users
Cache	Stored copy of content used for faster delivery
Cache Hit	Requested object is available in cache
Cache Miss	Requested object is not available in cache
Distribution	CloudFront configuration that controls how content is delivered
16. S3 vs CloudFront
Feature	S3	CloudFront
Primary purpose	Storage	Content delivery
Stores objects	Yes	Temporarily caches content
CDN	No	Yes
Origin	Can be an origin	Uses configured origins
Edge delivery	No	Yes
Caching	Storage/versioning features, not CDN caching	Yes
Static website files	Yes	Delivers them
Global content delivery	Not its primary role	Yes

Simple explanation:

S3        → Store
CloudFront → Deliver
17. DevOps Use Case

A common CI/CD architecture can look like:

Developer
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
Build Website
    ↓
Upload Files
    ↓
S3
    ↓
CloudFront
    ↓
Users

For example, after a new frontend version is built:

React/Vite Application
        ↓
npm run build
        ↓
Build Files
        ↓
S3
        ↓
CloudFront
        ↓
Users

This separates:

application build
storage
content delivery
18. Security Considerations

When using S3 as a CloudFront origin:

Recommended practices
Keep private training data private.
Keep S3 Block Public Access enabled unless a reviewed public architecture explicitly requires otherwise.
Use CloudFront Origin Access Control for private S3 origins.
Apply least-privilege permissions.
Do not upload passwords or API keys.
Do not upload AWS access keys.
Do not store .env files containing real secrets.
Review CloudFront and S3 access configuration carefully.
Avoid this architecture for private data
Internet
   ↓
Public S3 Bucket
   ↓
Private Files

Prefer:

Internet
   ↓
CloudFront
   ↓
Authorized Access
   ↓
Private S3
19. Practical Understanding Exercise

Assume:

S3 Bucket:
devops-training-bucket

Files:
index.html
style.css
logo.png

CloudFront distribution:

CloudFront
     ↓
devops-training-bucket
Request 1

User requests:

/index.html

If the object is cached:

User
 ↓
CloudFront
 ↓
Cache HIT
 ↓
User
Request 2

User requests:

/new-image.png

If the object is not cached:

User
 ↓
CloudFront
 ↓
Cache MISS
 ↓
S3
 ↓
new-image.png
 ↓
CloudFront
 ↓
User

This demonstrates the basic purpose of the CloudFront origin.

20. Key Questions and Answers
Q1. What is an Origin?

An origin is the source location from which CloudFront retrieves content when it is not available in the relevant cache.

Examples include S3 buckets, load balancers, EC2 applications, APIs, and custom HTTP servers.

Q2. Why is S3 being used as the Origin?

S3 is being used because it provides scalable object storage for static website files such as HTML, CSS, JavaScript, images, and other objects.

Q3. What happens when CloudFront receives a request?

CloudFront checks whether the requested content is available in its cache.

If it is a cache hit, CloudFront can return the cached content.

If it is a cache miss, CloudFront retrieves the content from the configured origin, such as S3, and then returns the content to the user. The object may also be cached according to the configured caching behavior.

21. What I Learned

From this task, I learned:

An origin is the source of content for CloudFront.
S3 can be configured as a CloudFront origin.
S3 is responsible for storing website files.
CloudFront is responsible for delivering content.
CloudFront checks its cache before contacting the origin.
A cache hit can serve content without contacting S3.
A cache miss causes CloudFront to retrieve the content from the origin.
CloudFront can cache content retrieved from the origin according to its configuration.
S3 can remain private while CloudFront provides controlled access.
Origin Access Control can be used for private S3 origins.
CloudFront improves content delivery performance by using distributed edge infrastructure.
S3 and CloudFront have different but complementary responsibilities.
22. Final Architecture
                         USER
                           │
                           ↓
                       BROWSER
                           │
                           ↓
                  CLOUDFRONT CDN
                           │
                           ↓
                     CACHE CHECK
                      /         \
                   HIT           MISS
                    │              │
                    ↓              ↓
              Cached Object       S3
                    │              │
                    │              ↓
                    │        Website Files
                    │              │
                    │              ↓
                    │         CloudFront
                    │              │
                    └──────┬───────┘
                           ↓
                          USER