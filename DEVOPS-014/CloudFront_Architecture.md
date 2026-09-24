TASK 2 — Understand the CloudFront Architecture

Time: 45 Minutes
Deliverable: CloudFront_Architecture.md

1. Objective

The objective of this task is to understand the basic architecture of Amazon CloudFront and how users receive website content through CloudFront before the request reaches the S3 origin.

The main components are:

User
CloudFront
Edge Location
Origin
Amazon S3
Website Files

The basic architecture is:

                    User
                     │
                     ↓
              CloudFront CDN
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Edge        Edge       Edge
      Location    Location   Location
          │          │          │
          └──────────┼──────────┘
                     ↓
                  S3 Origin
                     │
                     ↓
              Website Files
2. What is Amazon CloudFront?

Amazon CloudFront is AWS's Content Delivery Network (CDN) service.

It helps deliver website and application content to users through distributed AWS edge infrastructure.

Instead of every user directly requesting content from the S3 origin:

User
 │
 ↓
S3

CloudFront provides a delivery layer:

User
 │
 ↓
CloudFront
 │
 ↓
S3 Origin

CloudFront can cache frequently requested content so that subsequent requests can be served from an appropriate edge location.

3. Complete CloudFront Architecture

The complete simplified architecture is:

                         INTERNET
                            │
                            ↓
                          User
                            │
                            ↓
                    CloudFront CDN
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
        Edge Location  Edge Location  Edge Location
             │              │              │
             └──────────────┼──────────────┘
                            ↓
                        S3 Origin
                            │
                            ↓
                     Website Files
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
         index.html      style.css       app.js

This represents the relationship between the user, CloudFront, edge infrastructure, and S3.

4. Component 1 — User

The User is the person or client accessing the website.

For example, the user may open:

https://example.com

The browser sends an HTTP/HTTPS request for the website.

Architecture:

Browser
   │
   ↓
Website Request
   │
   ↓
CloudFront

The user does not need to know which particular edge location processes the request.

5. Component 2 — CloudFront

Amazon CloudFront is the CDN service between the users and the origin.

Its responsibilities can include:

Receiving user requests
Selecting an appropriate edge location
Checking cached content
Retrieving content from the origin when required
Serving cached content
Applying configured cache behavior
Supporting HTTPS/TLS
Providing a globally distributed delivery layer

Simplified:

User
 │
 ↓
CloudFront
 │
 ├── Cache available → Return content
 │
 └── Cache unavailable → Request origin
6. Component 3 — Edge Location

An Edge Location is part of AWS's distributed edge infrastructure used by CloudFront to process and deliver content closer to users.

Simplified architecture:

                  CloudFront
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Edge A       Edge B       Edge C
          │            │            │
       Users        Users        Users

The purpose of edge infrastructure is to reduce the distance that content needs to travel to users.

7. Multiple Edge Locations

CloudFront uses a distributed network of edge infrastructure.

For example:

                       CloudFront
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
   Edge Location A    Edge Location B    Edge Location C
        │                  │                  │
      Users              Users              Users

A user request is handled through CloudFront's distributed infrastructure.

The exact edge location used is determined by CloudFront based on its routing and network infrastructure.

8. Component 4 — Origin

An origin is the source from which CloudFront retrieves content.

In this task, the origin is Amazon S3.

CloudFront
     │
     ↓
S3 Origin

The S3 bucket contains the website files.

For example:

S3 Bucket
 │
 ├── index.html
 ├── style.css
 ├── app.js
 └── images/
      ├── logo.png
      └── banner.jpg

CloudFront retrieves content from the origin when the content is not available in the applicable cache.

9. Component 5 — Amazon S3

Amazon S3 is an object storage service.

In this architecture, S3 acts as the origin for website content.

CloudFront
     │
     ↓
S3 Bucket
     │
     └── Website Objects

Example:

S3 Bucket
 │
 ├── index.html
 ├── css/style.css
 ├── js/app.js
 └── images/logo.png

S3 is responsible for storing the website files.

CloudFront is responsible for delivering those files through its CDN infrastructure.

10. Component 6 — Website Files

The S3 bucket can contain different types of website files.

Examples:

index.html
style.css
app.js
logo.png
banner.jpg
favicon.ico

Architecture:

S3
 │
 └── Website Files
      │
      ├── HTML
      ├── CSS
      ├── JavaScript
      ├── Images
      └── Other Static Assets

These objects can be delivered through CloudFront.

11. Request Flow

The most important part of the architecture is understanding how a request travels.

Suppose the user requests:

https://example.com/index.html

The simplified flow is:

Step 1
User
 │
 ↓
Website Request

Step 2
CloudFront
 │
 ↓
Edge Infrastructure

Step 3
CloudFront checks cache
 │
 ├── Cache Hit
 │      ↓
 │   Return Content
 │
 └── Cache Miss
        ↓
      S3 Origin
        ↓
   Website File
        ↓
    CloudFront
        ↓
      User
12. Cache Hit

A cache hit occurs when CloudFront has the requested content available in the relevant cache.

Example:

User
 │
 ↓
CloudFront Edge
 │
 ↓
Cache Hit
 │
 ↓
index.html
 │
 ↓
User

The origin does not need to be contacted for that request when the cached response is usable.

This can improve performance and reduce origin traffic.

13. Cache Miss

A cache miss occurs when the requested content is not available in the relevant cache.

Example:

User
 │
 ↓
CloudFront Edge
 │
 ↓
Cache Miss
 │
 ↓
S3 Origin
 │
 ↓
index.html
 │
 ↓
CloudFront
 │
 ↓
User

CloudFront retrieves the content from the origin and can cache it according to the configured caching behavior.

14. Complete Request Flow
                         User
                           │
                           ↓
                    CloudFront CDN
                           │
                           ↓
                    Edge Location
                           │
                    ┌──────┴──────┐
                    ↓             ↓
                Cache Hit     Cache Miss
                    │             │
                    ↓             ↓
              Cached File      S3 Origin
                    │             │
                    │             ↓
                    │        Website File
                    │             │
                    └──────┬──────┘
                           ↓
                          User

This is the core CloudFront request model.

15. Why Use CloudFront with S3?

Using CloudFront with S3 provides several advantages.

Performance

Content can be delivered through CloudFront's distributed edge infrastructure.

Caching

Frequently requested content can be cached.

Reduced Origin Traffic

Cached requests may not need to reach S3.

HTTPS

CloudFront can provide HTTPS delivery using an appropriate TLS certificate configuration.

Global Delivery

CloudFront provides a global content delivery layer for users in different geographic locations.

Security

CloudFront can be part of an architecture where the S3 bucket remains private and CloudFront is authorized to retrieve content from it.

16. S3 vs CloudFront Responsibilities

It is important not to confuse the two services.

Component	Main Responsibility
User	Requests website content
CloudFront	Delivers content
Edge Location	Handles content closer to users
S3	Stores website files
Origin	Source of content
Cache	Stores reusable copies for faster delivery

The simple rule is:

S3
 ↓
Stores Content

CloudFront
 ↓
Delivers Content
17. Example Website Architecture

Consider a static website stored in S3.

S3 Bucket
 │
 ├── index.html
 ├── about.html
 ├── css/
 │    └── style.css
 ├── js/
 │    └── app.js
 └── images/
      ├── logo.png
      └── banner.jpg

CloudFront can be placed in front of the S3 origin:

                         Users
                           │
                           ↓
                     CloudFront
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
           Edge A       Edge B       Edge C
              │            │            │
              └────────────┼────────────┘
                           ↓
                       S3 Origin
                           │
                           ↓
                    Website Files
18. Security Architecture

For private website assets, a secure architecture can keep the S3 bucket from being directly public and allow CloudFront to access the required objects through an appropriate origin access configuration.

Conceptually:

Internet User
      │
      ↓
  CloudFront
      │
      ↓
Private S3 Origin
      │
      ↓
Website Files

This is preferable to making a private S3 bucket publicly accessible simply because a website needs to serve content.

For a modern S3 + CloudFront architecture, Origin Access Control (OAC) is commonly used to authorize CloudFront to access a private S3 origin.

19. CloudFront Distribution

A CloudFront Distribution is the configuration that tells CloudFront how to deliver content.

It can define settings such as:

CloudFront Distribution
        │
        ├── Origin
        ├── Cache Behavior
        ├── Cache Policy
        ├── Viewer Protocol Policy
        ├── Allowed Methods
        ├── HTTPS/TLS Configuration
        └── Other Delivery Settings

For this task:

CloudFront Distribution
        │
        ↓
S3 Origin

The distribution provides the connection between users and the S3 origin.

20. Architecture Summary

The entire architecture can be summarized as:

                    USER
                     │
                     ↓
              CLOUDFRONT CDN
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       EDGE        EDGE       EDGE
      LOCATION    LOCATION   LOCATION
          │          │          │
          └──────────┼──────────┘
                     ↓
                  S3 ORIGIN
                     │
                     ↓
              WEBSITE FILES
21. Real-World Example

Suppose a company hosts its frontend application in S3.

The files are:

index.html
app.js
style.css
logo.png

The architecture is:

Developer
    │
    ↓
Git Repository
    │
    ↓
CI/CD Pipeline
    │
    ↓
S3 Bucket
    │
    ↓
CloudFront
    │
    ↓
Users

CloudFront provides the content delivery layer between the users and the S3 origin.

A more complete architecture is:

                         Developer
                            │
                            ↓
                       Git Repository
                            │
                            ↓
                        CI/CD Build
                            │
                            ↓
                       S3 Origin
                            │
                            ↓
                     CloudFront CDN
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
          Edge A         Edge B         Edge C
             │              │              │
             └──────────────┼──────────────┘
                            ↓
                           Users
22. Important Terms
Term	Explanation
User	Client requesting website content
CloudFront	AWS Content Delivery Network
Edge Location	Distributed CloudFront edge infrastructure used to serve/process content closer to users
Origin	Source from which CloudFront retrieves content
S3	AWS object storage service
S3 Origin	S3 bucket configured as a CloudFront origin
Cache	Stored copy of content used to serve repeated requests efficiently
Cache Hit	Requested content is available in cache
Cache Miss	Requested content is not available in cache
Distribution	CloudFront configuration for delivering content
23. What I Learned

During this task, I learned:

CloudFront is AWS's CDN service.
Users request content through CloudFront.
Edge infrastructure helps deliver content closer to users.
An origin is the source of the content.
Amazon S3 can be used as a CloudFront origin.
S3 stores website files as objects.
CloudFront can cache frequently requested content.
A cache hit can allow content to be served without retrieving it again from the origin.
A cache miss can cause CloudFront to retrieve content from S3.
CloudFront and S3 have different responsibilities.
A CloudFront Distribution controls how content is delivered.
CloudFront can be used with private S3 origins using appropriate access controls such as Origin Access Control.