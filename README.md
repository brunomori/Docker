📘 Guia Docker para SRE
(Em construção — focado em uso prático no dia a dia)

---

## 🐳 Conteúdo 1 — Docker Básico 

### 📌 O que é Docker

Docker é uma plataforma que permite empacotar aplicações e suas dependências em **containers**, garantindo que rodem da mesma forma em qualquer ambiente.

* **Imagem** → molde da aplicação
* **Container** → imagem em execução

---

### 📦 Instalação do Docker

#### Ubuntu / Debian

```bash
sudo apt update
sudo apt install docker.io -y
```

Habilitar Docker no boot:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Adicionar usuário ao grupo docker:

```bash
sudo usermod -aG docker $USER
```

*(é necessário logout/login)*

---

## 🧱 Conteúdo 2 — Imagens Docker

### 📄 Dockerfile (criador de imagem)

```Dockerfile
FROM node:alpine
COPY . /app
WORKDIR /app
CMD ["node", "app.js"]
```

Explicação:

* `FROM` → imagem base
* `COPY` → copia arquivos
* `WORKDIR` → diretório de trabalho
* `CMD` → comando executado ao iniciar o container

---

### 🛠️ Build da imagem

```bash
docker build -t nome-da-imagem .
```

* `-t` → define a tag/nome da imagem
* `.` → diretório atual (onde está o Dockerfile)

---

## ▶️ Conteúdo 3 — Containers

### Executar container

```bash
docker run nome-da-imagem
```

Rodar em background:

```bash
docker run -d nome-da-imagem
```

Mapear portas:

```bash
docker run -p 3000:3000 nome-da-imagem
```

---

### 📋 Gerenciamento de containers

Listar containers rodando:

```bash
docker ps
```

Listar todos:

```bash
docker ps -a
```

Parar container:

```bash
docker stop ID_CONTAINER
```

Remover container:

```bash
docker rm ID_CONTAINER
```

---

## 🖼️ Conteúdo 4 — Gerenciamento de Imagens

Listar imagens:

```bash
docker images
```

Remover imagem:

```bash
docker rmi nome-da-imagem
```

---

## 📂 Conteúdo 5 — Volumes (Persistência)

Criar volume:

```bash
docker volume create meu-volume
```

Usar volume:

```bash
docker run -v meu-volume:/dados nome-da-imagem
```

---

## 🌐 Conteúdo 6 — Docker Network (Básico)

Listar redes:

```bash
docker network ls
```

Criar rede:

```bash
docker network create minha-rede
```

Usar rede:

```bash
docker run --network minha-rede nome-da-imagem
```

---

## 🧹 Conteúdo 7 — Limpeza (Dia a Dia SRE)

Remover containers parados:

```bash
docker container prune
```

Remover imagens não usadas:

```bash
docker image prune
```

Limpeza geral:

```bash
docker system prune
```

⚠️ Atenção: remove recursos não utilizados.

---

## 📌 Boas Práticas

* Use imagens pequenas (alpine)
* Nomeie bem imagens e containers
* Evite rodar containers como root
* Documente Dockerfiles

---

🧠 Objetivo: servir como **cola rápida** e base sólida para SRE Jr / DevOps.
