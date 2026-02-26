# Lab 1: Docker Basics and Cgroups Limits
## PART 1
### 1) Download and Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

## 2) Pull nginx:alpine Image
```bash
docker pull nginx:alpine
```
![Docker pull Output](image.png)

## 3) Run nginx:alpine in the Background
```bash
docker run -d --name Tarek-ITI-46 nginx:alpine
```
<img width="585" height="85" alt="image" src="https://github.com/user-attachments/assets/bb55583d-c37b-45bf-a2f6-78f6f51263fa" />


## 4) Run Apache Container with Resource Limits
```bash
docker run -d --name apache-limit --memory="70m" --cpus="1.0" httpd
```
<img width="720" height="271" alt="image" src="https://github.com/user-attachments/assets/2ea8446d-0389-4643-9ea0-74cd8a208947" />

**Command Breakdown:**

**`-d`**: Runs the container in Detached mode (background).

**`--name`** apache-limit: Assigns a custom name to the container for easier management.

**`--memory="50m"`**: Sets a hard limit on the RAM usage using Cgroups.

**`--cpu="1.0"`**: Restricts the container to use a maximum of 1 CPU core

**Cgroups Effect (Resource Control):**

When this command runs, the Linux Kernel creates a directory in the host's control group hierarchy: **/sys/fs/cgroup/system.slice/docker-02c1eee187a9ea421d5d1c5f83622f4a63572cba5770c11464085f259f7fb3ba.scope**

Inside this directory, the file memory.max is updated with the value **73400320** (which is exactly 70MB in bytes)

The Kernel monitors the container's processes; if they attempt to exceed **70MB**, the Kernel will intervene (Throttling or OOM Kill).

<img width="896" height="241" alt="image" src="https://github.com/user-attachments/assets/68532c13-bdae-40fe-b857-8d2d290590ca" />



## PART 2

### 1) Run an alpine container called memory-eater with memory limit of 50M
```bash
docker run -d --name memory-eater --memory="50m" --memory-swap="50m" alpine sh -c "tail /dev/zero"
```
<img width="923" height="83" alt="image" src="https://github.com/user-attachments/assets/d54ccc23-6bc1-4bf0-bcd7-bb782eb70ba0" />

**Command Breakdown:**
* : Sets a strict limit of 50MB of physical RAM via Cgroups.
* **`--memory-swap="50m"`**: By setting swap equal to memory, we effectively
**disable swap** ($50 - 50 = 0$), leaving the container with no escape when RAM runs out.
* **`sh -c "tail /dev/zero"`**: This command continuously reads from `/dev/zero`, a special file that provides null characters, forcing the container to fill its memory buffers almost instantly.

---

### 2. Proof of OOM Killer Intervention

**The Mechanism:**
When the `tail` process tries to allocate memory beyond the 50MB limit, the Linux Kernel's **OOM Killer** monitors the Cgroup. Since no swap is available, the Kernel sends a `SIGKILL` signal to the offending process to protect the host system's stability.

**How to Verify:**
After running the container, use the following commands to prove it was killed:

1. **Check Status:**
   ` ``bash
   docker ps -a | grep memory-eater
   ` ``
   *Result: You will see the status as `Exited`.*
   <img width="1110" height="86" alt="image" src="https://github.com/user-attachments/assets/2d3c5a7f-9ee1-4cc7-86f0-d1417fcc1d85" />


2. **Verify OOM Kill:**
   ` ``bash
   docker inspect -f '{{.State.OOMKilled}}' memory-eater
   ` ``
   *Result: Should return `true`.*
   
   <img width="512" height="92" alt="image" src="https://github.com/user-attachments/assets/4189bd4a-c6bc-49df-bfae-f567d5a8da77" />


---
