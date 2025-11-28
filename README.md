# **CRUD MEAN Application – Dockerized Deployment with CI/CD & NGINX Reverse Proxy**

This repository contains a fully containerized **CRUD MEAN Application** deployed on an EC2 instance with automated CI/CD using GitHub Actions and image hosting on Docker Hub.
All services (frontend, backend, database) are routed through a production-ready NGINX reverse proxy.

---

##  Project Structure

```
crud-dd-task-mean-app/
│
├── docker-compose.yml          # Compose file for prod deployment
├── nginx.conf                  # NGINX reverse proxy config
│
├── frontend/                   # Angular App (Dockerized)
│   └── Dockerfile
│
└── backend/                    # Node.js API + MongoDB Driver
    └── Dockerfile
```

---

#  Step-by-Step Setup & Deployment Instructions

## 1️⃣ Clone the Repository

```bash
git clone https://github.com/raju200539/dd_task.git
cd dd_task
```

---

## 2️⃣ Build & Push Docker Images to Docker Hub

### Backend:

```bash
docker build -t <dockerhub_username>/crud-backend:latest ./backend
docker push <dockerhub_username>/crud-backend:latest
```

### Frontend:

```bash
docker build -t <dockerhub_username>/crud-frontend:latest ./frontend
docker push <dockerhub_username>/crud-frontend:latest
```

GitHub Actions CI/CD pipeline automates this process.

---

## 3️⃣ Configure NGINX Reverse Proxy

`nginx.conf`:

```nginx
events {}

http {
    upstream frontend_upstream {
        server frontend:8081;
    }

    upstream backend_upstream {
        server backend:8080;
    }

    server {
        listen 80;
        server_name _;

        # API → backend
        location /api/ {
            proxy_pass http://backend_upstream;
            proxy_http_version 1.1;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Everything else → frontend
        location / {
            proxy_pass http://frontend_upstream;
            proxy_http_version 1.1;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

This setup routes:

* `/api` → backend
* `/` → frontend

Removes all CORS issues.

---

## 4️⃣ Deploy Using Docker Compose

On your EC2 instance:

```bash
docker compose pull
docker compose down
docker compose up -d
```

This automatically pulls and launches:

* crud-frontend
* crud-backend
* mongo
* nginx

---

## 5️⃣ Access the Application

```
http://<EC2_PUBLIC_IP>/
```

---

#  Screenshots

## 1. CI/CD Configuration (GitHub Actions)


<img width="1006" height="563" alt="image" src="https://github.com/user-attachments/assets/4c250fc6-0b6d-4340-9873-4c71b91382eb" />

## 2. CI/CD Execution Logs


<img width="1362" height="543" alt="image" src="https://github.com/user-attachments/assets/3d9cad47-3b10-4c1b-a25e-7d24c75d9354" />


## 3. Dockerhub Images pushing and deploying

### **Frontend Dockerhub Registry:**
<img width="639" height="626" alt="image" src="https://github.com/user-attachments/assets/97aa50d6-ca4a-4af5-83c5-f0bf354a0abb" />

### **backend Dockerhub Registry:**
<img width="645" height="564" alt="image" src="https://github.com/user-attachments/assets/5168fba0-ef57-4c6a-b322-5a7d6d75a43b" />

### **Container running in EC2:**
<img width="1097" height="84" alt="image" src="https://github.com/user-attachments/assets/8bfe6da8-efa5-46d2-adfd-52283b5ccfca" />


## 4. Application Running on EC2 (UI)

### **Home:**

<img width="1366" height="685" alt="home" src="https://github.com/user-attachments/assets/fc9675b3-6184-49df-9559-343c58f31720" />

### **Add Page:**
<img width="1361" height="710" alt="crud_add" src="https://github.com/user-attachments/assets/37c926aa-4427-472b-b959-db355050b1a9" />

### **Add Success:**
<img width="1365" height="683" alt="crud_add_suc" src="https://github.com/user-attachments/assets/0609818b-a682-48c6-b516-d0d8c1259fec" />


## 5. NGINX Infrastructure & Routing

### **Nginx as Container:**
<img width="1085" height="31" alt="image" src="https://github.com/user-attachments/assets/2497abec-540b-4421-9111-8866c339d348" />

### **EC2 Instance:**
<img width="1119" height="211" alt="image" src="https://github.com/user-attachments/assets/a72c3e2f-a0d1-4f53-8508-4cbff02042a4" />

### **Inbound rules:**
<img width="1092" height="197" alt="image" src="https://github.com/user-attachments/assets/6d8d47be-46dc-4614-9d73-2f7c1d415fdd" />

- All we need to install in EC2 is docker because nginx also running as container in the EC2 we have to allow only http traffic in the EC2.
Therefore, Application is secure by not exposing directly to the outside world, It can only accesable through nginx that is http:80
---

# ⚙️ Infrastructure Overview

This deployment uses:

* Angular Frontend (Docker container)
* Node.js Backend (Docker container)
* MongoDB Database (Docker container)
* NGINX Reverse Proxy (Docker container)
* EC2 as the host machine
* GitHub Actions for CI/CD
* Docker Hub as container registry
* Docker Compose as orchestrator

Everything runs inside a private Docker bridge network.

---

#  docker-compose.yml

```yaml
services:
  mongo:
    image: mongo:6
    container_name: mongo
    restart: unless-stopped
    volumes:
      - mongo-data:/data/db
    networks:
      - app-net

  backend:
    image: rajupadidapu/crud-backend:latest
    container_name: backend
    restart: unless-stopped
    environment:
      MONGO_URI: mongodb://mongo:27017/dd_db
      PORT: 8080
    depends_on:
      - mongo
    expose:
      - "8080"
    networks:
      - app-net

  frontend:
    image: rajupadidapu/crud-frontend:latest
    container_name: frontend
    restart: unless-stopped
    depends_on:
      - backend
    expose:
      - "8081"
    networks:
      - app-net

  nginx:
    image: nginx:alpine
    container_name: nginx
    depends_on:
      - frontend
      - backend
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - app-net

volumes:
  mongo-data:

networks:
  app-net:
    driver: bridge
```

---

# ✔️ Deployment Flow Summary

1. Push to `main`
2. GitHub Actions builds & pushes Docker images
3. Only `docker-compose.yml` and `nginx.conf` are deployed to EC2
4. EC2 runs `docker compose up -d`
5. App becomes live on port **80**
