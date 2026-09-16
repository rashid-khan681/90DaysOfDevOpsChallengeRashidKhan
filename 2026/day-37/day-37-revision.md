# Day 37: Docker Revision & Self-Assessment

## 1. Self-Assessment Checklist
I have marked this honestly based on my actual hands-on execution from Day 29 to Day 36 on my AWS EC2 instance:

- [x] Run a container from Docker Hub (interactive + detached)
- [x] List, stop, remove containers and images
- [x] Explain image layers and how caching works
- [x] Write a Dockerfile from scratch with FROM, RUN, COPY, WORKDIR, CMD
- [x] Explain CMD vs ENTRYPOINT
- [x] Build and tag a custom image
- [x] Create and use named volumes
- [x] Use bind mounts
- [x] Create custom networks and connect containers
- [x] Write a docker-compose.yml for a multi-container app
- [x] Use environment variables and .env files in Compose
- [x] Write a multi-stage Dockerfile
- [x] Push an image to Docker Hub
- [x] Use healthchecks and depends_on

*Self-Check Verdict:* **100% Confident**. (I executed all of these end-to-end while deploying the Nexus Prime dashboard).

## 2. Quick-Fire Questions (From Real Experience)

**1. What is the difference between an image and a container?**
- An image is an immutable blueprint built from read-only filesystem layers (like the heavy 1.8GB Node image I built). A container is the actual live, isolated Linux process running that blueprint with a thin read/write layer on top.

**2. What happens to data inside a container when you remove it?**
- It is permanently destroyed. I experienced this firsthand on Day 32: I created a `demo` table in a Postgres container, ran `docker rm -f`, and spun up a new one. My query through `ERROR: relation "demo" does not exist`. Data must be backed by Volumes to survive!

**3. How do two containers on the same custom network communicate?**
- They talk seamlessly using Docker's embedded DNS by referencing their container/service names. I proved this when the default bridge threw a `ping: bad address` error, but pinging `my-container2` worked instantly with 0% packet loss on my custom `my-app-net`.

**4. What does `docker compose down -v` do differently from `docker compose down`?**
- A standard `down` only removes the containers and networks, keeping your data safe. Adding the `-v` flag violently destroys your persistent named volumes too. I used `-v` specifically when my EC2 EBS volume was choked and I needed to deep clean my server.

**5. Why are multi-stage builds useful?**
- They are absolute lifesavers for cloud storage constraints and security. By discarding the heavy SDK builder stages, I shrank a Golang image from 492MB to 7.6MB (a 98.4% reduction) and my Node.js backend from 1.8GB to 391MB, copying only the compiled artifacts into production.

**6. What is the difference between COPY and ADD?**
- `COPY` strictly copies files/directories from the host into the container. `ADD` does the same but can also extract `.tar` archives and download from URLs. Best practice is to stick to `COPY` for predictable builds, which I used consistently in my Dockerfiles.

**7. What does `-p 8080:80` mean?**
- The flag `-p 8080:80` tells Docker to forward traffic from a specific port on your host machine to a port inside the container. This process is called **port mapping** or **port publishing**.

Here is the exact breakdown of how it works:
* **`-p` (or `--publish`)**: This tells Docker that you want to open a network port so outside traffic can reach the container.
* **`8080` (Host Port)**: The port on your *actual computer* (host machine) that you will connect to in your browser or app (e.g., `http://localhost:8080`).
* **`80` (Container Port)**: The port that the application inside the Docker container is actively *listening on* (usually the default port for web servers like Nginx or Apache).

When a request is made, it follows this exact path:
`Your Browser ──> Host Machine (Port 8080) ──> Docker Bridge ──> Container (Port 80)`

**8. How do you check how much disk space Docker is using?**
- By running `docker system df` to see a high-level summary of the space used by images, containers, local volumes, and the build cache. For a deeper, itemized breakdown showing exactly which specific container or image is consuming space, we append the verbose flag: `docker system df -v`.
