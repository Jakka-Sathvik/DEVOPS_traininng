TASK 6 — Enable S3 Versioning

Training Area: AWS / DevOps
Time: 1 Hour
Deliverable: S3_Versioning.md
Status: Practical Task

1. Objective

The objective of this task is to understand and practically test Amazon S3 Versioning.

In this exercise:

Enable versioning on the training bucket.
Create version-test.txt.
Upload the file.
Modify its content.
Upload it again using the same object key.
Modify it again.
Upload it a third time.
Inspect the different object versions.

Expected result:

version-test.txt
       │
       ├── Version 1
       ├── Version 2
       └── Version 3

The main purpose is to understand how S3 preserves multiple versions of the same object.

2. What is S3 Versioning?

S3 Versioning is a bucket-level feature that allows multiple versions of an object to be maintained.

Without versioning:

Upload version-test.txt
          ↓
Current Object

Upload version-test.txt again
          ↓
New content becomes current

With versioning:

version-test.txt
       │
       ├── Version 1
       ├── Version 2
       └── Version 3

Each upload using the same object key creates another object version when versioning is enabled.

3. Why Versioning is Important

Applications and users frequently modify files.

For example:

config.json

may be changed several times.

Without versioning, an accidental overwrite could remove the previous content.

With versioning:

config.json
   │
   ├── Version 1
   ├── Version 2
   └── Version 3

Previous versions can be retained and potentially restored when appropriate.

4. Prerequisites

Before starting, make sure:

You have access to the training AWS account.
You are using the approved training IAM identity.
Your S3 training bucket already exists.
You have permission to manage the bucket's versioning configuration.
You know the name of the training bucket.

Example:

Bucket:
sathvik-devops-training-2026

Do not use real credentials in notes or screenshots.

5. Step 1 — Open Amazon S3
Sign in to the AWS Management Console.
Search for:
S3
Open Amazon S3.
Open the training bucket.

Example:

S3
 │
 └── sathvik-devops-training-2026
6. Step 2 — Enable Versioning

Inside the bucket:

Open the Properties tab.
Find the Bucket Versioning section.
Select Edit.
Select:
Enable
Save the change.

Expected configuration:

Bucket Versioning
       ↓
    Enabled
7. Verify Versioning

After saving, return to the bucket's Properties page.

Confirm:

Bucket Versioning:
Enabled

This is important because the test will not demonstrate the expected behavior unless versioning is enabled before the uploads.

8. Step 3 — Create version-test.txt

Create a file on your computer named:

version-test.txt

For the first version, enter:

This is Version 1.

Save the file.

The local file is:

version-test.txt
9. Step 4 — Upload Version 1

Open the S3 bucket.

Select:

Upload

Add:

version-test.txt

Then select:

Upload

The bucket now contains:

sathvik-devops-training-2026
│
└── version-test.txt

At this point, there should be one current version.

Conceptually:

version-test.txt
       │
       └── Version 1
10. Step 5 — Modify the File

Open the local file:

version-test.txt

Change its content to:

This is Version 2.
The file was modified for S3 versioning testing.

Save the file.

Important:

Keep the filename exactly the same.

The object key must remain:

version-test.txt
11. Step 6 — Upload Version 2

Return to the same S3 bucket.

Select:

Upload

Upload the modified:

version-test.txt

Use the same object key:

version-test.txt

Complete the upload.

Because versioning is enabled, S3 keeps the previous version and creates another version.

Conceptually:

version-test.txt
       │
       ├── Version 1
       └── Version 2

The latest upload becomes the current version.

12. Step 7 — Modify the File Again

Modify the local file a second time.

Change the content to:

This is Version 3.
The file was uploaded for the third time.

Save the file.

The filename must remain:

version-test.txt
13. Step 8 — Upload Version 3

Return to the S3 bucket.

Select:

Upload

Upload the modified:

version-test.txt

Again, use the same object key:

version-test.txt

After the upload:

version-test.txt
       │
       ├── Version 1
       ├── Version 2
       └── Version 3

Version 3 should be the current version.

14. Expected Versioning Result

The final object structure should conceptually look like:

S3 Bucket
   │
   └── version-test.txt
          │
          ├── Version 1
          │    └── "This is Version 1."
          │
          ├── Version 2
          │    └── "This is Version 2."
          │
          └── Version 3
               └── "This is Version 3."

The important concept is that the object key remains the same:

version-test.txt

while S3 maintains separate versions behind that key.

15. Step 9 — Inspect Object Versions

Open the S3 bucket and locate:

version-test.txt

In the S3 Console, use the object/version viewing controls to display previous versions.

You should be able to distinguish:

Version 1
Version 2
Version 3

The console may display version IDs rather than literally labeling them "Version 1", "Version 2", and "Version 3."

For example:

version-test.txt
│
├── Version ID: abc123...
├── Version ID: def456...
└── Version ID: ghi789...

The exact Version IDs are generated by S3.

16. Version ID

Each version of an object receives a Version ID when versioning is enabled.

Conceptually:

Object Key:
version-test.txt

Version IDs:
├── ID-A
├── ID-B
└── ID-C

Therefore:

Same Object Key
       │
       ├── Version ID A
       ├── Version ID B
       └── Version ID C

The Version ID allows S3 to distinguish one version from another.

17. Current Version

At any point, one version is the current version of the object.

After uploading Version 3:

version-test.txt
       │
       ├── Version 1
       ├── Version 2
       └── Version 3 ← Current

The latest uploaded version becomes the current version.

Older versions remain available according to the bucket's versioning state and lifecycle/deletion behavior.

18. Versioning Demonstration

The entire practical exercise can be represented as:

              S3 Bucket
                  │
          Versioning Enabled
                  │
                  ↓
          version-test.txt
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
    Version 1  Version 2  Version 3
        │         │         │
        ↓         ↓         ↓
    Content 1   Content 2  Content 3

Upload workflow:

Create File
    ↓
Upload
    ↓
Version 1
    ↓
Modify File
    ↓
Upload Same Key
    ↓
Version 2
    ↓
Modify File
    ↓
Upload Same Key
    ↓
Version 3
19. Why is Versioning Useful?

S3 Versioning is useful because it protects against accidental changes and provides a history of object versions.

At least five practical use cases are:

Use Case 1 — Accidental File Overwrite

A developer accidentally uploads a new file using the same object key.

Example:

application-config.json

Original:

Version 1

Developer accidentally uploads incorrect configuration:

Version 2

With versioning:

application-config.json
       │
       ├── Version 1 ← Previous configuration
       └── Version 2 ← Incorrect configuration

The previous version can potentially be recovered.

Use Case 2 — Accidental Deletion

An object may be accidentally deleted.

With versioning enabled, S3 uses delete markers for versioned objects rather than simply eliminating all previous versions.

Conceptually:

Object
 │
 ├── Version 1
 ├── Version 2
 └── Delete Marker

Previous versions can remain available and can potentially be restored by removing the delete marker or otherwise managing the object versions appropriately.

Use Case 3 — Application Configuration Recovery

Configuration files can change over time.

Example:

config/
└── production.json

Versions:

Version 1 → Original configuration
Version 2 → Updated configuration
Version 3 → New configuration

If a new configuration causes a problem, the previous version can help with recovery.

Use Case 4 — CI/CD Artifact Management

DevOps pipelines can upload application artifacts to S3.

Example:

releases/
└── application.zip

With versioning:

application.zip
   │
   ├── Version 1
   ├── Version 2
   └── Version 3

This can provide additional protection against accidental overwrites of the same object key.

For stronger release identification, teams should also use explicit application/build versioning in object keys or artifact metadata rather than relying only on S3 versioning.

Example:

releases/
├── 1.0.0/application.zip
├── 1.0.1/application.zip
└── 1.1.0/application.zip
Use Case 5 — Data Recovery

Versioning can help recover earlier versions of files.

Example:

report.pdf
   │
   ├── Version 1
   ├── Version 2
   └── Version 3

If Version 3 contains an error, an earlier version can potentially be retrieved.

Use Case 6 — Protection Against Human Error

A user may accidentally modify or replace an important file.

Example:

backup/database.sql

Versioning can preserve earlier versions, providing an additional recovery mechanism.

Use Case 7 — Audit and Change History

Versioning can provide a history of object versions.

For example:

Object
 │
 ├── Older Version
 ├── Modified Version
 └── Current Version

This can help teams understand how an object changed over time.

However, S3 Versioning should not be treated as a complete audit system. For detailed API/user activity auditing, services such as AWS CloudTrail are used.

20. Five Main Use Cases Summary
#	Use Case	Benefit
1	Accidental overwrite	Recover previous object version
2	Accidental deletion	Previous versions can remain available
3	Configuration recovery	Restore an earlier configuration
4	CI/CD artifacts	Protect against accidental artifact replacement
5	Data recovery	Retrieve an earlier version
6	Human error	Additional protection from mistakes
7	Change history	Maintain object-version history
21. Important Difference: Versioning vs Backup

Versioning is useful, but it should not automatically be considered a complete backup strategy.

Versioning helps maintain previous versions of objects in the same S3 bucket.

A broader backup/disaster recovery strategy may require:

Separate backup locations
Cross-Region replication
Lifecycle policies
Retention policies
Backup services
Access controls
Monitoring
Recovery testing

Simple distinction:

Versioning
    ↓
Multiple versions of an object

versus:

Backup / DR
    ↓
Broader recovery strategy
22. Storage Consideration

Versioning can increase storage usage because previous versions are retained.

Example:

Version 1 = 10 MB
Version 2 = 10 MB
Version 3 = 10 MB

Potential stored data:

Approximately 30 MB

Therefore, organizations should consider lifecycle management and retention requirements.

For example:

Object Versions
      ↓
Lifecycle Rules
      ↓
Transition / Expiration

Lifecycle rules should be designed carefully so that required versions are not removed prematurely.

23. Security Considerations

Versioning does not replace access control.

Use:

IAM
 ↓
Least Privilege
 ↓
S3 Permissions

Do not assume that enabling versioning makes the bucket secure.

A secure architecture should combine:

Block Public Access
        +
IAM Least Privilege
        +
Encryption
        +
Versioning
        +
Monitoring
24. Screenshot Requirements

For your training documentation, capture screenshots of:

Screenshot 1 — Versioning Enabled

Show:

Bucket
 → Properties
 → Bucket Versioning
 → Enabled
Screenshot 2 — Version 1

Show the uploaded:

version-test.txt
Screenshot 3 — Version 2

Show that the same object key was uploaded again.

Screenshot 4 — Version 3

Show the third upload.

Screenshot 5 — Object Versions

Show the version listing for:

version-test.txt

Do not expose:

Access keys
Secret keys
Session tokens
Passwords
MFA recovery information
25. Final Architecture
                         AWS S3
                            │
                            ↓
                         Bucket
                            │
                    Versioning Enabled
                            │
                            ↓
                    version-test.txt
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
          Version 1      Version 2      Version 3
              │             │             │
              ↓             ↓             ↓
          Content 1       Content 2      Content 3

DevOps example:

Developer
    │
    ↓
Git / CI/CD
    │
    ↓
Build Artifact
    │
    ↓
S3
    │
    └── Versioned Object
            │
            ├── Previous Version
            ├── Previous Version
            └── Current Version