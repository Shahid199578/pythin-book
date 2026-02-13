# Chapter 22: Python & Docker - The Container Handler
> "It works on my machine... so we'll ship your machine."

## The Goal: Container Automation
You can run `docker run nginx`.
But what if you need to:
1.  Build 50 images?
2.  Wait for a database container to be healthy?
3.  Clean up all stopped containers older than 1 hour?

You need **Python + Docker**.

---

## 22.1 The Docker SDK for Python

While you can use `subprocess.run(["docker", ...])`, the official library is cleaner.

```bash
pip install docker
```

### Listing Containers

```python
import docker

client = docker.from_env()

for container in client.containers.list():
    print(f"ID: {container.short_id} | Image: {container.image.tags} | Status: {container.status}")
```

---

## 22.2 Running Containers

Let's launch a Redis server from Python.

```python
try:
    container = client.containers.run(
        "redis:alpine",
        name="my-redis",
        ports={'6379/tcp': 6379},
        detach=True # Run in background
    )
    print(f"Redis started! ID: {container.short_id}")

except docker.errors.APIError as e:
    print(f"Docker Error: {e}")
```

---

## 22.3 Parsing Dockerfiles

Sometimes you need to analyze a `Dockerfile` as text.

```python
def check_dockerfile(path="Dockerfile"):
    with open(path, "r") as f:
        lines = f.readlines()
    
    for line in lines:
        if line.startswith("FROM"):
            image = line.split()[1]
            if "latest" in image:
                print(f"[WARN] Using 'latest' tag in {image} is dangerous!")

# check_dockerfile()
```

---

## 22.4 DevOps Use Case: The Janitor Script

A script to clean up stopped containers and unused images (Pruning).

```python
import docker

client = docker.from_env()

# Safety Check: Is Docker running?
try:
    client.ping()
except docker.errors.DockerException:
    print("Error: Could not connect to Docker. Is it running? Do you have permission?")
    exit(1)

def prune_system():
    # 1. Remove Stopped Containers
    pruned_containers = client.containers.prune()
    print(f"Deleted Containers: {pruned_containers['ContainersDeleted']}")
    
    # 2. Remove Dangling Images (None:None)
    pruned_images = client.images.prune()
    print(f"Deleted Images: {len(pruned_images.get('ImagesDeleted', []))}")

if __name__ == "__main__":
    confirm = input("Run Docker Prune? (y/n): ")
    if confirm.lower() == 'y':
        prune_system()
```

---

## 22.5 Integration Tests (Waiting for Services)

When testing, you need to wait for the DB to handle connections.

```python
import time

def wait_for_db(container):
    print("Waiting for DB...")
    for i in range(10):
        if "Ready to accept connections" in container.logs().decode("utf-8"):
            print("DB is Ready!")
            return True
        time.sleep(1)
    return False

# container = client.containers.run("postgres", detach=True)
# wait_for_db(container)
```

---

## 22.6 The Laws of Container Scripts

1.  **The Law of Binding:** Use `docker.from_env()` to connect to your local Docker socket.
2.  **The Law of Cleanup:** Always `container.stop()` and `container.remove()` in your test scripts (use `try/finally`).
3.  **The Law of Tags:** Never deploy `latest`. Parse your Dockerfiles to enforce specific versions.

---

## [!] Common Mistakes

### Error #1: Permission Denied
`docker.errors.DockerException: Error while fetching server API version`
**Fix:** Your user is not in the `docker` group. Run `sudo usermod -aG docker $USER`.

### Error #2: Resource Leaks
You started a container in a loop but verify crashed. Now you have 50 zombie containers eating RAM.
**Fix:** Implementing a "Cleanup Hook" (using `atexit` or `finally`) is mandatory.

---

## Practice Problem: The Image Builder

Write a script that:
1.  Takes a directory path as input.
2.  Checks if a `Dockerfile` exists.
3.  Builds the image using `client.images.build()`.
4.  Tags it as `myapp:v1`.

---

## Chapter Summary

-   **Docker SDK:** A powerful Python library for controlling the Docker Daemon.
-   **Automation:** Start, Stop, Build, and Prune containers programmatically.
-   **Testing:** Use Python to spin up ephemeral environments for tests.
-   **Sanity:** Scripts enable complex orchestrations (like "Start DB, Wait, Start App").
