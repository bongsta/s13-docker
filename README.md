# S13 Training Camps

A self-contained student training website that runs as one Docker container.

## Single-container architecture

The `app` container provides everything:

- Responsive HTML, CSS, and JavaScript website
- Course catalog API
- Student registration and login API
- Enrollment API
- File-based demo persistence through the `app-data` Docker volume

The application uses only the Node.js standard library and has no package dependencies. GitHub Actions builds the Docker image on every branch push and pull request, then starts it on a GitHub-hosted runner and checks the health and courses APIs.

## Tools used

- HTML5, CSS3, and vanilla JavaScript
- Node.js 22
- Docker and Docker Compose
- GitHub Actions
- Git and curl

## Run

```bash
docker build -t s13-training-camps:latest .
docker volume create s13-training-camps-data
docker run -d --name s13-training-camps --restart unless-stopped -p 8080:8080 -v s13-training-camps-data:/data s13-training-camps:latest
```

Open <http://localhost:8080>.

## Verify

```bash
docker ps --filter name=s13-training-camps
curl http://localhost:8080/health
curl http://localhost:8080/api/courses
```

## Stop

```bash
docker rm -f s13-training-camps
```

Use `docker volume rm s13-training-camps-data` only when you intentionally want to delete saved demo users and enrollments.

## CI/CD

`.github/workflows/docker-build.yml` builds the image on every branch push and pull request using a GitHub-hosted runner. On branch pushes, it logs in to Docker Hub with the `DOCKERHUB_TOKEN` Actions repository secret and pushes `devopseasylearning/s13-training-camps` tagged with the commit SHA and `latest`. Pull requests build and test without publishing. CI then starts a temporary container and captures a website screenshot artifact; this is not a persistent deployment. Create a Docker Hub access token with write permission and add it in GitHub under **Settings → Secrets and variables → Actions** as `DOCKERHUB_TOKEN`.

## Public deployment on Render

The root `render.yaml` defines a free Docker web service that builds from this repository's `develop` branch, checks `/health`, and deploys new commits automatically. To create it, sign in to Render, choose **New → Blueprint**, connect this GitHub repository, and deploy the `render.yaml` Blueprint. Render will provide a public `*.onrender.com` URL after the first deployment. Free services can spin down when idle, and their filesystem is ephemeral, so demo users and enrollments stored in `/data` are not guaranteed to persist across restarts or deploys.
