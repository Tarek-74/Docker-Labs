# Lab 1: Docker Basics and Cgroups Limits

## 1) Download and Install Docker
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```

## 2) Pull nginx:alpine Image
```bash
docker pull nginx:alpine
![Docker pull Output](image.png)
```

## 3) Run nginx:alpine in the Background
```bash
docker run -d --name Tarek-ITI-46 nginx:alpine
```

## 4) Run Apache Container with Resource Limits
```bash
docker run -d --name apache-limit --memory="70m" --cpus="1.0" httpd
```
