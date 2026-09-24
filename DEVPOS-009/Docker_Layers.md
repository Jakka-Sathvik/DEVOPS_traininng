# DEVOPS-009 — Task 3: Dockerfile Layer Understanding

## Sample Dockerfile Used

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

Build command:
```
docker build -t devops-app:v2 .
```

---

## Step 1 — First Build (cold cache)

On the very first build, Docker has no cached layers to reuse, so **every
instruction runs from scratch**:

```
[1/5] FROM python:3.11-slim
[2/5] WORKDIR /app
[3/5] COPY requirements.txt .
[4/5] RUN pip install --no-cache-dir -r requirements.txt   <-- slow, downloads packages
[5/5] COPY app.py .
```

Each instruction produces a new layer, and the final image is the sum of
all layers stacked on top of the base image.

---

## Step 2 — Rebuild With No Changes

Running `docker build -t devops-app:v2 .` again, without editing anything,
produces:

```
[1/5] FROM python:3.11-slim            CACHED
[2/5] WORKDIR /app                     CACHED
[3/5] COPY requirements.txt .          CACHED
[4/5] RUN pip install ...              CACHED
[5/5] COPY app.py .                    CACHED
```

Every single step says `CACHED`. The build finishes almost instantly
because Docker recognizes that nothing changed and reuses every existing
layer instead of re-executing the instructions.

---

## Step 3 — Modify One Instruction and Rebuild

Now `app.py` is changed (e.g. the print statement is edited) and the
image is rebuilt:

```
[1/5] FROM python:3.11-slim            CACHED
[2/5] WORKDIR /app                     CACHED
[3/5] COPY requirements.txt .          CACHED
[4/5] RUN pip install ...              CACHED
[5/5] COPY app.py .                    REBUILT
```

Only the step that changed (`COPY app.py .`) — and anything **after** it
in the file — gets rebuilt. Everything before that line is untouched and
still comes from cache.

If instead `requirements.txt` had been changed, the cache would break one
step earlier: `COPY requirements.txt .`, `RUN pip install...`, and
`COPY app.py .` would all rebuild, because a cache miss invalidates that
layer **and every layer after it**, not just the one that changed.

---

## Explain

### What is a layer?
A layer is a read-only, immutable snapshot of filesystem changes produced
by a single Dockerfile instruction (mainly `FROM`, `RUN`, `COPY`, `ADD`).
Docker stacks these layers on top of each other to form the final image.
Layers are content-addressed (identified by a hash of their contents plus
the instruction that produced them), which is what makes caching and reuse
possible — identical layers are stored and shared only once, even across
different images.

### What is build cache?
The build cache is Docker's mechanism for skipping instructions it has
already executed before. Before running an instruction, Docker checks
whether a layer already exists that was produced by the *same instruction*
applied to the *same input state* (same base layer + same instruction
text/files). If a match is found, Docker reuses that existing layer
instead of re-executing the instruction — this is what shows up as
`CACHED` in the build output.

### Why does Docker reuse layers?
- **Speed**: skipping unchanged steps (like a `pip install` or `apt-get
  install`) can turn a multi-minute build into a few seconds.
- **Efficiency**: identical layers are stored once on disk and shared
  across images/containers, rather than duplicated.
- **Consistency**: reusing a layer guarantees the exact same output as
  last time, since nothing about that step's inputs changed.
- **Reduced network/CPU use**: cached steps don't re-download packages or
  redo expensive computation.

Caching stops at the first instruction whose inputs changed — that layer
and everything after it must rebuild, because each layer depends on the
one below it.

### Why should Dockerfiles be organized properly?
Ordering directly determines how much of the cache is usable:

- **Put things that change rarely first** (base image, OS packages,
  dependency manifests) and **things that change often last** (application
  source code). This way, routine code edits only invalidate the last one
  or two layers instead of the whole image.
- **Separate dependency installation from source copy**, e.g. `COPY
  requirements.txt .` + `RUN pip install ...` *before* `COPY . .`. If you
  copy all source code first, then install dependencies, any single-line
  code change forces a full dependency reinstall on every build — slow and
  wasteful.
- **Combine related RUN commands** where sensible (e.g. `apt-get update &&
  apt-get install -y ...` in one layer) to avoid stale package lists and
  reduce the number of layers.
- **Smaller, well-ordered images** build faster, deploy faster, and are
  easier to debug since each layer has a clear, predictable purpose.

In short: a well-ordered Dockerfile maximizes cache hits, minimizes build
time, and produces smaller, more predictable images — poor ordering
throws away most of the benefit Docker's layer caching is designed to
give you.
