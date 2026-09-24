# Task 5: Docker Build Report

## 1. Core Architectural Concepts
To understand how a local build functions, three core architectural elements must be explicitly tracked:

* **Dockerfile + Application Source:** The `Dockerfile` serves as a step-by-step automation manifest containing sequential build instructions. The application source code comprises the runtime logic, dependencies, and functional configurations that get embedded into the target execution environment.
* **Docker Build Context:** The explicit directory space passed to the build command (indicated by the `.` positional argument). When an engine build is invoked, all recursive file networks inside this folder are compressed and transferred directly to the background Docker daemon engine. Keeping this path optimized prevents structural caching delays and image bloat.
* **Docker Image:** The structural, layered, read-only immutable artifact generated upon successful build execution. It acts as an isolated packaging snapshot ready for high-fidelity deployment inside container instances.

---

## 2. Step-by-Step Execution & Diagnostic Log

### Step 1: Initial Local Build Attempt (Root Context Validation Failure)
The workflow began at the `/home` filesystem path. This space lacked the infrastructure code needed to initiate an application layer compilation.
```bash
(venv) sathvik@DESKTOP-PR53HA3:/home$ sudo docker build -t my-app:v1.0 .
```
* **Engine Result:** `ERROR: failed to build: failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory`
* **Architectural Lesson:** The build context path (`.`) was pointed to an empty system root where no `Dockerfile` manifested. The execution engine safely aborted the process before layer generation.

### Step 2: Absolute Path Navigation Attempt (Directory Resolution Failure)
An attempt was made to access the project folder using an absolute root reference path.
```bash
(venv) sathvik@DESKTOP-PR53HA3:/home$ cd /devops-day20-docker
```
* **Engine Result:** `-bash: cd: /devops-day20-docker: No such file or directory`
* **Architectural Lesson:** The application directory did not reside directly in the physical core storage root (`/`). Instead, it existed inside the local user directory namespace, requiring relative folder traversal.

### Step 3: Context Relocation and Build Re-Execution (Virtual Network Timeout)
The active path was successfully adjusted to match the project directory structure, but execution stalled at the external layer resolution phase.
```bash
(venv) sathvik@DESKTOP-PR53HA3:/home$ cd ~
(venv) sathvik@DESKTOP-PR53HA3:~$ cd devops-day20-docker
(venv) sathvik@DESKTOP-PR53HA3:~/devops-day20-docker$ sudo docker build -t my-app:v1.0 .
```
* **Engine Result:** `ERROR: failed to build: failed to solve: failed to fetch anonymous token... dial tcp: lookup auth.docker.io on 172.30.224.1:53: read udp: i/o timeout`
* **Architectural Lesson:** The build context was correctly target-aligned, but the WSL2 hypervisor's virtual network bridge mapping (`172.30.224.1`) dropped internal UDP routing packets. This blocked the engine from contacting Docker Hub to download the base image `python:3.11-slim`.

### Step 4: System Init Subsystem Invalidation
An attempt was made to restart the background engine using standard system management procedures inside the WSL2 terminal.
```bash
(venv) sathvik@DESKTOP-PR53HA3:~/devops-day20-docker$ sudo service docker restart
```
* **Engine Result:** `Failed to restart docker.service: Unit docker.service not found.`
* **Architectural Lesson:** Because the environment runs Docker Desktop for Windows integrated into WSL2, service controls are managed via the host Windows OS kernel rather than a native Linux init/systemd framework.

### Step 5: Network Remediation & Final Successful Build Execution
To resolve the virtual networking block, the internal Linux environment routing configuration was manually updated to point directly to reliable upstream public nameservers, bypassing the internal virtual router gateway.
```bash
# Injecting public authoritative DNS records directly into the environment runtime
(venv) sathvik@DESKTOP-PR53HA3:~/devops-day20-docker$ sudo sh -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'

# Re-executing the local compilation manifest
(venv) sathvik@DESKTOP-PR53HA3:~/devops-day20-docker$ sudo docker build -t my-app:v1.0 .
```
* **Engine Result:** `Building FINISHED [docker:default] - SUCCESS`
* **Architectural Lesson:** Overwriting `/etc/resolv.conf` forced network translation queries to exit the virtual adapter cleanly. This let Docker pull down the base imagery and assemble the local project layers without further delays.

---

## 3. Post-Build Image Inspection Data
The image was successfully inspected locally by executing the target metadata query:
```bash
sudo docker images my-app:v1.0
```

| Structural Metric | Inspected Value / Output Registry |
| :--- | :--- |
| **Image Name** | `my-app` |
| **Image Tag** | `v1.0` |
| **Image ID** | *[Insert specific Image ID from terminal output, e.g., sha256:e3b0c442]* |
| **Image Size** | *[Insert specific Virtual Size from terminal output, e.g., 124 MB]* |
| **Build Integrity Status** | `SUCCESSFUL COMPILATION` |

---

## 4. Verification Checkpoints
* **Context Isolation:** Aligning execution to `~/devops-day20-docker` prevented host-level system files from bloating the build context transfer payload.
* **Layer Caching:** Re-running the stabilized pipeline automatically optimized performance by matching unmodified manifest commands with local disk storage layers.
* **Subsystem Architecture Awareness:** Troubleshooting confirmed that Docker Desktop manages operations via cross-OS virtualization, which helps explain why native Linux management utilities are restricted inside the WSL2 terminal.
