TASK 1 — Understand CDN & CloudFront

Time: 45 Minutes
Deliverable: Day14_CloudFront_Theory.md

1. Objective

The objective of this task is to understand Content Delivery Networks (CDNs) and Amazon CloudFront before performing practical AWS configuration.

This task covers:

What is a CDN?
Why websites use CDNs
What is Amazon CloudFront?
What is an Origin?
What is an Edge Location?
What is a CloudFront Distribution?
What is caching?
Why caching improves website performance
Difference between Amazon S3 and Amazon CloudFront

The basic architecture is:

                        Users
                          │
                          ↓
                    CloudFront CDN
                          │
                  ┌───────┴───────┐
                  ↓               ↓
            Edge Location     Edge Location
                  │               │
                  └───────┬───────┘
                          ↓
                       Origin
                          │
                          ↓
                    S3 / Web Server
2. What is a CDN?

CDN stands for Content Delivery Network.

A CDN is a distributed network of servers that delivers website content to users from locations that are geographically closer to them.

Instead of every user requesting content directly from one central server:

User
  │
  ↓
Central Server

a CDN can distribute and cache content across many locations:

                         CDN
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Edge A       Edge B       Edge C
             │            │            │
          Users        Users        Users

This can reduce latency and improve the performance of websites and applications.

3. Why Do Websites Use CDNs?

Websites use CDNs primarily to improve:

Performance
Response time
Availability
Scalability
User experience
Delivery of static content

Consider a website whose origin server is located in one geographic region.

A user located far away may experience higher network latency.

Without a CDN:

User
 │
 │ Long Network Distance
 ↓
Origin Server

With a CDN:

User
 │
 ↓
Nearby Edge Location
 │
 ↓
Origin Server

If the requested content is already cached at the edge, the request may not need to travel back to the origin.

4. Benefits of a CDN
4.1 Lower Latency

Content can be delivered from a geographically closer edge location.

User
 ↓
Nearby Edge
 ↓
Content

This can reduce network latency.

4.2 Faster Website Loading

Frequently requested static content can be cached.

Examples include:

HTML
CSS
JavaScript
Images
Fonts
Videos
Downloads

Instead of repeatedly retrieving the same content from the origin, the CDN can serve cached content.

4.3 Reduced Origin Load

Suppose 10,000 users request the same image.

Without caching:

10,000 Requests
       │
       ↓
Origin Server

With CDN caching:

10,000 User Requests
         │
         ↓
      CDN Edge
         │
         ↓
   Cached Content

The origin may receive significantly fewer requests for cacheable content.

4.4 Better Scalability

CDNs are designed to handle large volumes of content requests across distributed infrastructure.

This helps websites handle traffic spikes more efficiently.

5. What is Amazon CloudFront?

Amazon CloudFront is AWS's Content Delivery Network service.

It delivers content to users through a globally distributed network of AWS edge infrastructure.

A simplified architecture is:

                         AWS
                          │
                     CloudFront
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Edge A       Edge B       Edge C
             │            │            │
           Users        Users        Users
                          │
                          ↓
                       Origin
                          │
                    ┌─────┴─────┐
                    ↓           ↓
                   S3        Web Server

CloudFront can work with different origins, including:

Amazon S3
Application Load Balancers
EC2-hosted web servers
API endpoints
Other HTTP/HTTPS origins
6. What is an Origin?

An origin is the source from which CloudFront retrieves content when it is not already available in the appropriate cache.

For example:

CloudFront
    │
    ↓
S3 Bucket

Here:

S3 Bucket = Origin

Another example:

CloudFront
    │
    ↓
Application Load Balancer
    │
    ↓
EC2

Here the load balancer can serve as the CloudFront origin.

7. Common CloudFront Origins
S3 Origin
CloudFront
    ↓
S3 Bucket

This is commonly used for:

Static websites
Images
CSS
JavaScript
Downloads
Static application assets
Web Server Origin
CloudFront
    ↓
Load Balancer
    ↓
Application Servers

This can be used when CloudFront is placed in front of a dynamic application.

8. What is an Edge Location?

An Edge Location is a location in AWS's distributed edge infrastructure where CloudFront can process and serve content closer to users.

Simplified example:

                         CloudFront
                             │
          ┌──────────────────┼──────────────────┐
          ↓                  ↓                  ↓
     Edge Location A    Edge Location B    Edge Location C
          │                  │                  │
       Users              Users              Users

The purpose is to bring content delivery closer to users.

9. User Request and Edge Location

Suppose a user requests:

https://example.com/logo.png

The request reaches CloudFront.

Conceptually:

User
 │
 ↓
CloudFront Edge
 │
 ├── Cache Hit ──→ Return Content
 │
 └── Cache Miss
          ↓
       Origin
          ↓
      Content
          ↓
      CloudFront
          ↓
         User

This is one of the most important CloudFront concepts.

10. What is a CloudFront Distribution?

A CloudFront Distribution is the CloudFront configuration that defines how content should be delivered.

A distribution contains configuration such as:

Origin
Cache behavior
Viewer protocol settings
Allowed HTTP methods
HTTPS/TLS configuration
Cache policies
Distribution domain name
Other delivery/security settings

Simplified architecture:

                    CloudFront Distribution
                             │
              ┌──────────────┼──────────────┐
              ↓              ↓              ↓
           Origin       Cache Behavior    Security
              │
              ↓
          S3 / Server

A CloudFront distribution acts as the configuration layer between users and the origin.

11. CloudFront Distribution Example

Suppose an S3 bucket contains:

website/
├── index.html
├── style.css
├── app.js
└── images/
    └── logo.png

A CloudFront distribution can be configured with:

CloudFront Distribution
        │
        ↓
S3 Bucket
        │
        ├── index.html
        ├── style.css
        ├── app.js
        └── images/logo.png

Users access the application through the CloudFront distribution rather than directly accessing the origin in the intended architecture.

12. What is Caching?

Caching means temporarily storing frequently requested content so that it can be served more quickly when requested again.

For CloudFront:

User
  │
  ↓
CloudFront Edge
  │
  ↓
Cached Content

If content is available in the cache, CloudFront can serve it without retrieving it again from the origin.

13. Cache Hit

A cache hit occurs when the requested content is already available in the relevant CloudFront cache.

Example:

User
 │
 ↓
CloudFront
 │
 ↓
Cache Hit
 │
 ↓
Cached Image
 │
 ↓
User

This is efficient because the request can be satisfied from the edge cache.

14. Cache Miss

A cache miss occurs when the requested content is not currently available in the relevant cache.

Example:

User
 │
 ↓
CloudFront
 │
 ↓
Cache Miss
 │
 ↓
Origin
 │
 ↓
Content
 │
 ↓
CloudFront
 │
 ↓
User

CloudFront can retrieve the content from the origin and cache it according to the applicable configuration.

15. Why Does Caching Improve Website Performance?

Caching can improve performance because it reduces the amount of work and network distance involved in repeatedly retrieving the same content from the origin.

Example:

Without CDN Cache:

User
 ↓
Origin
 ↓
Content

With CDN Cache:

User
 ↓
Nearby Edge
 ↓
Cached Content

Potential benefits include:

Lower latency
Faster content delivery
Reduced origin requests
Reduced origin bandwidth consumption
Better user experience
Better handling of repeated requests
16. Example of Caching

Suppose a website contains a logo:

logo.png

100 users request it.

Without caching:

100 Requests
      │
      ↓
Origin
      │
      ↓
logo.png

With CloudFront caching:

100 User Requests
        │
        ↓
CloudFront Edge
        │
        ↓
Cached logo.png

After the content is cached, repeated requests can often be served directly from the edge.

17. Cache Duration

Cached content is not necessarily stored forever.

CloudFront caching behavior depends on its configured policies and cache-control-related settings.

A simplified example:

Content
   ↓
Cached
   ↓
Cache Lifetime
   ↓
Expiration / Revalidation
   ↓
New Origin Request

For example, static assets such as versioned JavaScript or image files may be suitable for longer cache lifetimes.

Frequently changing content may require shorter caching or revalidation.

18. Static vs Dynamic Content

CDNs are particularly effective for static content.

Static Content

Examples:

Images
CSS
JavaScript
Fonts
PDF files
Static HTML

Architecture:

User
 ↓
CloudFront
 ↓
Cached Static Content
Dynamic Content

Examples:

User-specific dashboards
Shopping cart information
Database-generated responses
Personalized API responses

Dynamic content can also be delivered through CloudFront, but caching must be designed carefully.

19. Difference Between S3 and CloudFront

S3 and CloudFront are different AWS services with different responsibilities.

Amazon S3

Amazon S3 is an object storage service.

It stores objects inside buckets.

Architecture:

S3
 │
 └── Bucket
      ├── Object
      ├── Object
      └── Object

Examples:

index.html
style.css
logo.png
backup.zip
report.pdf
Amazon CloudFront

CloudFront is a content delivery network.

It delivers content through AWS's distributed edge infrastructure.

Architecture:

Users
  │
  ↓
CloudFront
  │
  ↓
Edge Locations
  │
  ↓
Origin

The origin can be S3 or another supported HTTP/HTTPS source.

20. S3 vs CloudFront Comparison
Feature	Amazon S3	Amazon CloudFront
Main purpose	Object storage	Content delivery
Stores objects	Yes	No, primarily serves/caches content
Bucket required	Yes	No
CDN	No	Yes
Edge caching	No	Yes
Origin	Can be an origin	Uses an origin
Primary role	Store data	Deliver content
Common use	Files, backups, static assets	Fast website/content delivery
Global delivery	Through S3 service endpoints	Through CloudFront edge infrastructure

The two services are often used together:

                   Users
                     │
                     ↓
                CloudFront
                     │
               Cached Content
                     │
                     ↓
                   S3
                     │
                  Objects
21. S3 + CloudFront Example

Suppose we have a static website:

S3 Bucket
 │
 ├── index.html
 ├── style.css
 ├── app.js
 └── images/
      └── logo.png

CloudFront can be placed in front:

                     Users
                       │
                       ↓
                 CloudFront
                       │
              ┌────────┴────────┐
              ↓                 ↓
         Edge Cache          S3 Origin
              │                 │
              ↓                 ↓
           Content            Files

This provides a common architecture for delivering static content globally.

22. CloudFront Request Flow

A simplified request lifecycle is:

1. User requests content
          ↓
2. Request reaches CloudFront
          ↓
3. CloudFront checks cache
          ↓
      ┌───┴───┐
      ↓       ↓
   Hit       Miss
      ↓       ↓
 Return     Request
 Content    Origin
              ↓
           Content
              ↓
        CloudFront Cache
              ↓
            User

This flow is fundamental to understanding CDN operation.

23. Why CloudFront is Useful for DevOps

CloudFront is useful in DevOps architectures because it can be integrated with application delivery and deployment workflows.

Examples include:

Static website delivery
Frontend application delivery
Image delivery
JavaScript/CSS delivery
Software downloads
API delivery
TLS/HTTPS termination
Cache management
Global content delivery

Example deployment architecture:

Developer
    │
    ↓
Git Repository
    │
    ↓
CI/CD Pipeline
    │
    ↓
Build
    │
    ↓
S3
    │
    ↓
CloudFront
    │
    ↓
Users
24. CloudFront and Security

CloudFront can also be part of a secure content delivery architecture.

Important considerations include:

HTTPS
TLS certificates
Access control
Origin protection
Cache policies
Restricting direct origin access where appropriate
Monitoring
Logging

For a private S3 origin, a common modern architecture is to allow CloudFront to retrieve objects from S3 while keeping direct public access to the bucket blocked.

Conceptually:

Internet User
      │
      ↓
CloudFront
      │
      ↓
Private S3 Origin

This is preferable to simply making a private data bucket publicly accessible.

25. CDN vs Origin

It is important to understand the difference.

CDN

The CDN delivers content closer to users.

User
 ↓
CloudFront
 ↓
Edge
Origin

The origin is the source of the content.

CloudFront
 ↓
Origin
 ↓
S3 / Web Server

Therefore:

Origin = Source
CloudFront = Delivery Layer
26. Important CloudFront Terms
Term	Meaning
CDN	Distributed content delivery network
CloudFront	AWS CDN service
Origin	Source of content
Edge Location	AWS edge infrastructure used for content delivery
Distribution	CloudFront configuration for delivering content
Cache	Stored copy of content for faster delivery
Cache Hit	Requested content found in cache
Cache Miss	Requested content not found in cache
S3	AWS object storage service
27. Real-World Example

Consider an online shopping website.

It contains:

Product Images
CSS
JavaScript
Product Pages
APIs

Without CDN:

Users
  │
  ↓
Application Server
  │
  ↓
Content

With CloudFront:

                         Users
                           │
                           ↓
                      CloudFront
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       Edge A           Edge B           Edge C
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                    Application Origin

Static content can be cached at the edge, reducing repeated requests to the origin.

28. Key Architecture

The complete concept can be summarized as:

                         USERS
                           │
                           ↓
                    Amazon CloudFront
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Edge A        Edge B        Edge C
             │             │             │
             └─────────────┼─────────────┘
                           │
                     Cache Check
                           │
                     ┌─────┴─────┐
                     ↓           ↓
                  Cache Hit   Cache Miss
                     │           │
                     ↓           ↓
                  Content      Origin
                                 │
                         ┌───────┴───────┐
                         ↓               ↓
                        S3          Web Server