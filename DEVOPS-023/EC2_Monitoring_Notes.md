# EC2 Monitoring Notes

## EC2 Instance Monitoring Review

Amazon EC2 provides monitoring information that helps DevOps engineers understand the health, performance, and availability of an instance.

---

## 1. CPU Utilization

### What is it?

CPU Utilization shows the percentage of CPU resources currently being used by the EC2 instance.

### What does this information tell a DevOps Engineer?

- Determines whether the instance has enough processing power.
- Identifies performance bottlenecks.
- Helps detect unusual spikes in resource consumption.
- Assists in capacity planning and scaling decisions.
- Indicates whether an application is overloading the server.

### Example

- CPU = 20% → Instance is operating normally.
- CPU = 90%+ for a long period → Application may be overloaded and require optimization or scaling.

---

## 2. Network

### What is it?

Network monitoring shows the amount of incoming and outgoing traffic on the EC2 instance.

Common Metrics:
- Network In
- Network Out
- Packets In
- Packets Out

### What does this information tell a DevOps Engineer?

- Measures application traffic.
- Detects traffic spikes and unusual behavior.
- Helps identify network bottlenecks.
- Assists in troubleshooting connectivity issues.
- Can reveal possible security concerns such as unexpected traffic increases.

### Example

- Normal traffic: 100 MB/hour
- Sudden traffic increase: 5 GB/hour

This may indicate:
- Increased user activity
- Load testing
- Potential DDoS attack
- Application issue

---

## 3. Instance Status

### What is it?

Instance Status checks whether the EC2 instance and AWS infrastructure are functioning correctly.

Status Checks:
- System Status Check
- Instance Status Check

### What does this information tell a DevOps Engineer?

- Verifies that the EC2 instance is healthy.
- Detects operating system failures.
- Identifies hardware or infrastructure problems.
- Helps determine whether issues originate from AWS or the application.

### Example

Status Check Results:

- 2/2 Checks Passed → Healthy Instance
- 1/2 Checks Passed → Potential issue requiring investigation
- 0/2 Checks Passed → Instance may be unavailable

---

## 4. Monitoring Period

### What is it?

The Monitoring Period determines how frequently EC2 metrics are collected and reported to CloudWatch.

Types:

#### Basic Monitoring
- Metrics collected every 5 minutes

#### Detailed Monitoring
- Metrics collected every 1 minute

### What does this information tell a DevOps Engineer?

- Determines how quickly issues can be detected.
- Provides greater visibility into system performance.
- Helps with real-time troubleshooting.
- Supports faster incident response.

### Example

Basic Monitoring:
- CPU spike may be visible after several minutes.

Detailed Monitoring:
- CPU spike detected within one minute.
- Faster alerting and response.

---

# Summary Table

| Monitoring Item | What It Tells a DevOps Engineer |
|----------------|----------------------------------|
| CPU Utilization | Server workload and performance health |
| Network | Traffic patterns and connectivity status |
| Instance Status | Overall health of the EC2 instance and AWS infrastructure |
| Monitoring Period | Frequency of metric collection and speed of issue detection |

---

# Conclusion

EC2 monitoring helps DevOps engineers maintain application availability, identify performance issues, troubleshoot incidents, and make informed scaling decisions. Regularly reviewing CPU utilization, network activity, instance status, and monitoring periods is essential for maintaining a stable and reliable production environment.