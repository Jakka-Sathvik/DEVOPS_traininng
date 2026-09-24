## The Core Flow

```
User
 ↓
Authentication
 ↓
"Who are you?"
 ↓
Authorization
 ↓
"What are you allowed to do?"
```

---

## Examples

### Example 1: Developer Logs Into AWS

```
Developer logs into AWS
        ↓
Authentication
        ↓
AWS identifies developer
        ↓
Authorization
        ↓
IAM policy determines permissions
```

### Example 2: AWS CLI Using Access Keys

```
User runs an AWS CLI command
        ↓
Authentication
        ↓
AWS validates the Access Key ID + Secret Access Key
        ↓
Authorization
        ↓
IAM policy checks whether the requested action is allowed
```

### Example 3: EC2 Instance Accessing S3 via IAM Role

```
EC2 instance calls the S3 API
        ↓
Authentication
        ↓
AWS verifies the temporary credentials from the attached IAM Role
        ↓
Authorization
        ↓
Role's policy decides if the S3 action is permitted
```

### Example 4: Authenticated but Not Authorized

```
IAM user tries to delete an S3 bucket
        ↓
Authentication
        ↓
AWS confirms the user's identity — login succeeds
        ↓
Authorization
        ↓
Policy has no "s3:DeleteBucket" permission → Access Denied
```

### Example 5: MFA-Protected Console Login

```
User enters username, password, and MFA code
        ↓
Authentication
        ↓
AWS confirms identity using password + one-time MFA token
        ↓
Authorization
        ↓
IAM policy defines what the now-verified user can access
```

### Example 6: Real-Life Analogy — Airport Boarding

```
Passenger arrives at the airport gate
        ↓
Authentication
        ↓
Staff checks passport/ID → confirms who the passenger is
        ↓
Authorization
        ↓
Boarding pass checked → confirms which flight/seat they can access
```

---

## Quick Comparison

| | Authentication | Authorization |
|---|---|---|
| Question | "Who are you?" | "What are you allowed to do?" |
| Order | Happens first | Happens second, after authentication succeeds |
| Verifies | Identity | Permissions |
| AWS mechanism | Password, access keys, MFA | IAM policies |
| Typical failure | 401 Unauthorized (identity not confirmed) | 403 Forbidden (identity confirmed, action blocked) |

---

## Key Takeaway

Authentication confirms identity; authorization confirms permissions — they're independent checks. Example 4 shows this clearly: a user can pass authentication (a valid, successful login) and still fail authorization (no policy permission for that specific action).
