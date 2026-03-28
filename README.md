# Full-Stack App Deployment on AWS EC2 using Docker

This project demonstrates deploying a full-stack React + Node.js app in Docker containers on AWS EC2, using Nginx as a reverse proxy for frontend-backend communication.

## Project Overview

- **Frontend:** React.js
- **Backend:** Node.js / Express
- **Containerization:** Docker
- **Web Server / Reverse Proxy:** Nginx
- **Cloud Deployment:** AWS EC2

The frontend and backend are deployed in separate Docker containers. Nginx routes traffic to the frontend on `/` and the backend API on `/api/`.

## Project Structure

**Backend**
- Node.js/Express server
- Runs on port 5000
- Dockerfile included
- `.dockerignore`, `.env`, and `dist` folder present

**Frontend**
- React application
- Runs on port 3000
- Dockerfile included
- `.dockerignore`, `.env`, and `build` folder present

**Nginx**
- Reverse proxy container
- Forwards requests to frontend (`/`) and backend (`/api/`)
- Custom configuration in `default.conf`

**Root**
- `docker-compose.yml` (optional)
- `.dockerignore`
- Project README

---
## Docker Hub Repository Setup

1. **Create a Docker Hub Account**  
   - Go to [https://hub.docker.com](https://hub.docker.com)  
   - Sign up for a free account or log in if you already have one.

2. **Create a New Repository**  
   - Click **Create Repository**  
   - Enter a **name** (e.g., `fullstack-deploy-docker-on-ec2`)  
   - Set **Visibility**: Public (for resume/demo projects) or Private  
   - Click **Create**  

3. **Login to Docker Hub from your machine**  
```bash
docker login
# Enter your Docker Hub username and password when prompted

## Project Setup in VS Code
1. **Create folders:**
```bash
mkdir backend
cd backend
npm init -y        # create Node.js backend
mkdir frontend
cd ../frontend
npx create-react-app .   # create React frontend
mkdir nginx

2. Add .dockerignore in frontend & backend:
node_modules
build
dist
.env
.git
.gitignore

3. Add Dockerfile in each folder:
Backend Dockerfile (Node.js/Express)
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "index.js"]

Frontend Dockerfile (React)
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

4. Add nginx/default.conf

server {
    listen 80;

    location / {
        proxy_pass http://frontend:3000;
    }

    location /api/ {
        proxy_pass http://backend:5000;
    }
}

5. Make dockercompose.yml file
code from git hub repo

6. Docker Build & Push
Build images:
docker build -t your_dockerhub_username/backend:latest ./backend
docker build -t your_dockerhub_username/frontend:latest ./frontend

Login to Docker Hub:
docker login

Push images:
docker push your_dockerhub_username/backend:latest
docker push your_dockerhub_username/frontend:latest

## Deployment Steps on AWS EC2
1. Launch an **EC2 instance** (Ubuntu) and open ports: 22 (SSH), 80 (HTTP), 3000, 5000.  

2. Connect via SSH:
```bash
cd Downloads
ssh -i "e.pem" ec2-user@<EC2-Public-IP>

3.Install Docker and Docker Compose:
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo docker --version

4. Install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64" -o /usr/local/bin/docker-compose

Give permissin
sudo chmod +x /usr/local/bin/docker-compose

Verrfy
docker-compose --version

5. Pull Docker images from Docker Hub:
sudo docker pull ajay2929/fullstack-deploy-docker-on-ec2:01  # Backend
sudo docker pull ajay2929/fullstack-deploy-docker-on-ec2:02  # Frontend

6. Create a custom Docker network:
sudo docker network create fullstack-net

7. Run backend and frontend containers:
sudo docker run -d --name backend --network fullstack-net -p 5000:5000 ajay2929/fullstack-deploy-docker-on-ec2:01
sudo docker run -d --name frontend --network fullstack-net -p 3000:3000 ajay2929/fullstack-deploy-docker-on-ec2:02

8. Configure Nginx:
sudo mkdir -p /home/ec2-user/nginx
sudo nano /home/ec2-user/nginx/default.conf
Paste the following:
server {
    listen 80;

    location / {
        proxy_pass http://frontend:3000;
    }

    location /api/ {
        proxy_pass http://backend:5000;
    }
}

9.Set folder and file permissions:
sudo chmod 755 /home/ec2-user/nginx        # Folder

Run Nginx container with mounted config:
sudo docker run -d --name nginx-server \
  --network fullstack-net \
  -p 80:80 \
  -v /home/ec2-user/nginx:/etc/nginx/conf.d:ro \
  nginx

10. **Access in browser**  
   - Open a browser and enter your EC2 public IP:
(http://<EC2_PUBLIC_IP> # Shows frontend via Nginx
http://<EC2_PUBLIC_IP>/api # Shows backend via Nginx)

11. Notes
Backend runs on port 5000, frontend on 3000, and Nginx exposes 80.
Docker network ensures communication between containers.
Folder mount allows Nginx to read your custom configuration.
Use chmod 755 for directories and chmod 644 for config files.

12. GitHub Integration
Initialize Git repository locally:
git init
git add .
git commit -m "Initial commit"

Push to GitHub:
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git branch -M main
git push -u origin main


