# 🚀 Automated CI/CD Pipeline — Python Flask on AWS EC2

> **Phase 1 · Project 1** of a DevOps learning roadmap  
> Deploy a Flask web app to AWS EC2 using Docker, Nginx, and GitHub Actions — fully automated on every `git push`.

---

## 📋 What This Project Does

Every time you push code to the `main` branch, GitHub Actions automatically:

1. Builds a Docker image of your Flask app
2. SSH's into your AWS EC2 server
3. Pulls down the latest image and restarts the container

The result is a live web app at `http://YOUR_EC2_IP` — deployed without touching a server manually.

---

## 🛠️ Tech Stack

| Tool | Role |
|---|---|
| **Python Flask** | Web framework — handles HTTP routes and responses |
| **Gunicorn** | Production WSGI server — serves Flask in production |
| **Docker** | Containerizes the app for consistent, portable deployment |
| **Nginx** | Reverse proxy — forwards public traffic to the Flask container |
| **AWS EC2** | Cloud virtual machine where the app runs |
| **GitHub Actions** | CI/CD pipeline — automates build and deploy on push |

---

## 📁 Project Structure

```
python-flask/
├── app.py                          # Flask web application
├── requirements.txt                # Python dependencies
├── Dockerfile                      # Container build instructions
├── .gitignore                      # Files excluded from Git
└── .github/
    └── workflows/
        └── deploy.yml              # GitHub Actions CI/CD pipeline
```

---

## 🚀 Quick Start

### Prerequisites

- AWS account with an EC2 instance (Ubuntu 22.04, t2.micro)
- GitHub account
- Docker installed locally
- Python 3.x installed locally

### Step 1 — Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/python-flask.git
cd python-flask
```

### Step 2 — Run Locally (Optional)

```bash
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

Visit `http://localhost:5000` to see the app running locally.

### Step 3 — Run with Docker Locally

```bash
docker build -t my-devops-app .
docker run -p 5000:5000 my-devops-app
```

Visit `http://localhost:5000` — same result, now in a container.

### Step 4 — Set Up AWS EC2

1. Launch an EC2 instance (Ubuntu 22.04, t2.micro — free tier eligible)
2. In the Security Group, open inbound ports: **22** (SSH), **80** (HTTP), **5000** (app)
3. Download your `.pem` key pair

SSH into your instance and install Docker and Nginx:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_IP

sudo apt update && sudo apt upgrade -y
sudo apt install docker.io nginx -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
# Log out and back in after this step
```

### Step 5 — Configure Nginx on EC2

```bash
sudo nano /etc/nginx/sites-available/default
```

Replace the file contents with:

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
sudo systemctl restart nginx
```

### Step 6 — Add GitHub Secrets

Go to your GitHub repo → **Settings → Secrets and variables → Actions** → **New repository secret**

| Secret Name | Value |
|---|---|
| `EC2_HOST` | Your EC2 public IP address |
| `EC2_SSH_KEY` | Full contents of your `.pem` private key file |

### Step 7 — Deploy

Push any change to the `main` branch:

```bash
git add .
git commit -m "initial deployment"
git push origin main
```

Watch the **Actions tab** on GitHub — the pipeline will run automatically. Once it turns green, visit `http://YOUR_EC2_IP` in your browser. 🎉

---

## ⚙️ How the CI/CD Pipeline Works

```
git push origin main
        │
        ▼
GitHub Actions triggers (deploy.yml)
        │
        ├── ① Checkout repository code
        ├── ② Build Docker image
        └── ③ SSH into EC2 and run:
                 ├── Stop existing container
                 ├── Remove existing container
                 ├── Build new Docker image from GitHub
                 └── Start new container on port 5000
        │
        ▼
Nginx forwards port 80 → port 5000 → Flask app
        │
        ▼
Live at http://YOUR_EC2_IP ✅
```

---

## 🔧 Nginx Configuration Explained

Nginx acts as a **reverse proxy** — it sits between the internet and your Flask app:

```
Internet (port 80) → Nginx → Flask/Gunicorn (port 5000)
```

**Why use Nginx instead of exposing Flask directly?**
- Clean URLs without port numbers (`http://yoursite.com` instead of `:5000`)
- Nginx handles thousands of concurrent connections efficiently
- Easy to add HTTPS/SSL later with Let's Encrypt
- Required for running multiple apps on the same server

---

## 📂 Key Files Reference

### `app.py`
The Flask application. Add more routes here as your app grows:

```python
@app.route("/health")
def health():
    return {"status": "ok"}, 200
```

### `Dockerfile`
Builds a production-ready image using Python 3.14-slim base and Gunicorn:

```dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

### `deploy.yml`
Triggered on push to `main`. Builds the image and deploys via SSH using `appleboy/ssh-action`.

---

## 🔒 Security Notes

- Never commit your `.pem` key file — it is in `.gitignore`
- Store all sensitive values (EC2 IP, SSH key) in GitHub Secrets only
- Consider restricting EC2 Security Group inbound SSH (port 22) to your IP only
- For production, add HTTPS via Let's Encrypt + Certbot

---

## 🗺️ What's Next

This project is **Phase 1 · Project 1** of a DevOps roadmap:

| Phase | Projects |
|---|---|
| **Phase 1 — Foundation** | ✅ CI/CD Pipeline · Infrastructure as Code · Logging & Monitoring |
| **Phase 2 — Intermediate** | Kubernetes on EKS · GitOps with ArgoCD · Secrets Management |
| **Phase 3 — Advanced** | Coming soon |
| **Phase 4 — MLOps Bridge** | Coming soon |

---

## 📚 Resources

- [Flask Documentation](https://flask.palletsprojects.com/)
- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- [AWS EC2 Getting Started](https://docs.aws.amazon.com/ec2/index.html)

---

## 📝 License

MIT License — feel free to use this as a learning template.
