# DEVOPS-009 — Task 7: Docker Logs & Troubleshooting

## Setup — Intentionally Broken Container

**Dockerfile used (broken version):**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app/app.py .

# BROKEN: references a script name that doesn't exist (typo)
CMD ["python", "main.py"]
```

The app file on disk is `app.py`, but the `CMD` tells the container to run
`main.py` — a **wrong startup command** (the file simply doesn't exist under
that name inside the image).

Build and run:
```
docker build -t broken-app:v1 .
docker run -d --name broken-app broken-app:v1
```

---

## Mandatory Troubleshooting Report

**Problem:**
Container does not start.

**Expected:**
Application should start.

**Actual:**
Container exits.

**Investigation:**
Checked `docker ps -a`
Checked `docker logs`
Checked Dockerfile

### Investigation Detail

```
$ docker ps
CONTAINER ID   IMAGE            COMMAND         STATUS    NAMES
(empty — no running containers)

$ docker ps -a
CONTAINER ID   IMAGE            COMMAND            STATUS                     NAMES
a1b2c3d4e5f6   broken-app:v1    "python main.py"   Exited (2) 3 seconds ago   broken-app

$ docker logs broken-app
python3: can't open file '/app/main.py': [Errno 2] No such file or directory

$ docker inspect broken-app
...
"Cmd": ["python", "main.py"],
"State": {
    "Status": "exited",
    "ExitCode": 2,
    "Error": ""
}
...

$ docker exec -it broken-app sh
Error response from daemon: Container a1b2c3d4e5f6 is not running
```

`docker ps` shows nothing because the container already exited. `docker ps
-a` confirms it exited with code 2 immediately after starting. `docker
logs` gives the exact reason: Python cannot find `main.py`. `docker
inspect` confirms the `Cmd` that was actually run. Since the container
isn't running, `docker exec` can't attach a shell — that only works on
live containers, so log/inspect output is the primary diagnostic here.

**Root Cause:**
Incorrect startup command.

The Dockerfile's `CMD` pointed to `main.py`, but the file copied into the
image is named `app.py`. The mismatch between the `COPY` source and the
`CMD` target caused Python to fail immediately with a "file not found"
error, which made the container exit right after starting.

**Solution:**
Corrected command.

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app/app.py .

# FIXED: correct filename
CMD ["python", "app.py"]
```

Rebuild and rerun:
```
docker build -t broken-app:v2 .
docker run -d --name fixed-app broken-app:v2
```

**Verification:**
Container running successfully.

```
$ docker ps
CONTAINER ID   IMAGE            COMMAND          STATUS         NAMES
b7c8d9e0f1a2   broken-app:v2    "python app.py"  Up 10 seconds  fixed-app

$ docker logs fixed-app
App starting...
App is running
App is running
```

---

## Other Common Breakage Patterns (Reference)

The task lists several other ways a container can be intentionally broken.
The same `docker ps -a` → `docker logs` → `docker inspect` → `docker exec`
workflow applies to each:

| Break Type | Symptom | Diagnostic Command | Typical Root Cause |
|---|---|---|---|
| Wrong startup command | Exits immediately, "file not found" | `docker logs` | CMD/ENTRYPOINT references wrong file/binary |
| Wrong port | App runs, but unreachable from host | `docker inspect` (check `ExposedPorts`), `docker port <container>` | `EXPOSE` or `-p` mapping doesn't match the port the app actually listens on |
| Missing file | Crash on startup, "No such file or directory" | `docker logs` | `COPY` source path wrong, or file not built into image context |
| Incorrect environment variable | App starts but misbehaves/crashes on a config step | `docker logs`, `docker exec -it <container> env` | `ENV`/`-e` value missing, misspelled, or wrong type |
| Incorrect volume path | App can't read/write expected data, "No such file or directory" for a data path | `docker inspect` (check `Mounts`), `docker exec -it <container> ls <path>` | `-v` host:container path mismatch, or container path doesn't match what the app expects |

## Key Takeaways

- `docker ps -a` is the first step whenever a container isn't showing up
  in `docker ps` — it reveals exit status and exit code for stopped
  containers.
- `docker logs` is almost always the fastest way to find the actual error
  message a crashing app produced.
- `docker inspect` is useful for confirming the exact `Cmd`, environment
  variables, port mappings, and volume mounts that were actually applied
  — useful when the Dockerfile "looks right" but something upstream
  (build args, `docker run` flags) changed it.
- `docker exec -it <container> sh` only works on a **running** container —
  it's most useful for interactively poking around when the app starts but
  behaves incorrectly, not when it crashes immediately.
