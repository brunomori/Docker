📘 Guia Docker para SRE

## 📌 Índice
- Docker Básico
- Dockerfile
- Containers
- Logs
- Exec
- Portas
- Volumes
- Network
- Docker Hub
- Compose

---

## 🐳 Conteúdo 1 — Docker Básico (Sobrevivência)

### 📌 O que é Docker

Docker é uma plataforma que permite empacotar aplicações e suas dependências em **containers**, garantindo que rodem da mesma forma em qualquer ambiente.

---

### 🧠 Diferença entre Imagem e Container

* **Imagem Docker** → é o *modelo* (template) da aplicação. Ela é **imutável** e contém tudo que a aplicação precisa para rodar (código, dependências, configurações).
* **Container Docker** → é a **imagem em execução**. Ele é criado a partir de uma imagem e representa a aplicação rodando de fato.

Exemplo prático:

* Imagem = classe
* Container = objeto da classe

Ou ainda:

* Imagem = receita
* Container = prato pronto

Resumo rápido:

* Uma imagem pode gerar **vários containers**
* Se o container for apagado, a imagem continua existindo

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
FROM node:12-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
```

Explicação:

* `FROM` → define a imagem base
* `WORKDIR` → define o diretório de trabalho dentro do container
* `COPY . .` → copia **todos os arquivos do diretório atual (host)** para o **diretório atual do container** (definido pelo WORKDIR)
* `CMD` → comando executado ao iniciar o container

CMD` → comando executado ao iniciar o container

---

### ➕ COPY vs ADD (Quando e por que usar)

### COPY (recomendado na maioria dos casos)

Usado para copiar arquivos e diretórios do **host** para o **container** de forma simples e previsível.

```Dockerfile
COPY . .
```

Use quando:

* estiver copiando arquivos locais
* quiser comportamento simples e controlado
* seguir boas práticas

---

### ADD (uso específico)

O `ADD` faz tudo que o `COPY` faz **e mais**, porém com comportamentos extras.

Exemplo:

```Dockerfile
ADD app.tar.gz /app
```

O que o `ADD` faz de diferente:

* descompacta automaticamente arquivos `.tar`
* permite copiar arquivos a partir de uma URL

Use `ADD` quando:

* precisar descompactar arquivos automaticamente
* precisar baixar um arquivo remoto (caso muito específico)

⚠️ Atenção:

* Evite `ADD` quando `COPY` resolver
* `ADD` pode gerar comportamentos inesperados

📌 Boa prática SRE:

> Use **COPY por padrão**. Use **ADD somente quando precisar das funcionalidades extras**.

---

## 🛠️ Build da imagem

### Build básico da imagem

```bash
docker build -t app .
```

* `-t app` → nome/tag da imagem
* `.` → diretório atual (onde está o Dockerfile)

---

### Executar container em modo interativo (debug)

```bash
docker run -it app sh
```

Usado para:

* depurar erros
* inspecionar arquivos
* testar comandos dentro do container

---

## 🧱 Dockerfile — Exemplos Organizados por Nível

### 🔹 Exemplo 1 — Dockerfile simples (base)

```Dockerfile
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN apk add --no-cache python2 g++ make
```

Usado quando:

* projeto simples
* sem preocupação inicial com usuário ou otimização

---

### 🔹 Exemplo 2 — Dockerfile com variável de ambiente

```Dockerfile
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN apk add --no-cache python2 g++ make

ENV API_URL=https://api.bruno.com/
```

Usado quando:

* aplicação depende de configurações externas
* ambientes diferentes (dev, stage, prod)

---

### 🔹 Exemplo 3 — Dockerfile mais próximo de produção (boas práticas)

```Dockerfile
FROM node:12-alpine
WORKDIR /app

RUN addgroup bruno && adduser -S -G bruno dev
USER dev

COPY . .
RUN apk add --no-cache python2 g++ make
RUN yarn install --production

EXPOSE 3000
CMD ["node", "src/index.js"]
```

### 📖 Explicação linha por linha

* `FROM node:12-alpine`
  → Define a imagem base. Usa Node.js versão 12 sobre Alpine Linux, que é uma distribuição leve e comum em containers.

* `WORKDIR /app`
  → Define `/app` como diretório de trabalho dentro do container. Todos os comandos seguintes usam esse diretório como base.

* `RUN addgroup bruno && adduser -S -G bruno dev`
  → Cria um grupo chamado `bruno` e um usuário `dev` pertencente a esse grupo. Isso evita rodar a aplicação como root.

* `USER dev`
  → Define que, a partir daqui, os comandos e a aplicação rodarão como o usuário `dev`.

* `COPY . .`
  → Copia todos os arquivos do diretório atual do host para o diretório `/app` dentro do container.

* `RUN apk add --no-cache python2 g++ make`
  → Instala dependências necessárias para compilar módulos nativos. O `--no-cache` evita deixar arquivos temporários na imagem.

* `RUN yarn install --production`
  → Instala apenas dependências de produção, ignorando dependências de desenvolvimento.

* `EXPOSE 3000`
  → Documenta que a aplicação escuta na porta 3000. Não abre a porta, apenas informa.

* `CMD ["node", "src/index.js"]`
  → Define o comando principal que será executado quando o container iniciar.

---

## 🛠️ Build da imagem

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

Modo interativo (acesso ao shell do container):

```bash
docker run -it nome-da-imagem sh
```

Explicação:

* `-i` → mantém a entrada padrão aberta (interativo)
* `-t` → cria um terminal (TTY)
* `sh` → shell dentro do container (em imagens alpine)

---

### 📋 Gerenciamento de containers

Listar Iniciar, listar containers rodando:

```bash
docker run -d --name mycontainer brunomanzini/app:v1 (inciado no modo interativo onde fica ativo mesmo depois de sair Ctr+C dando um name)
```

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
docker rm ID_CONTAINER ( usar stop e depois rm , caso queira matar direto usar: "docker rm -f container_name")
```
## 🐳 Logs Containers

```bash
# Ver logs
docker logs <container>

# Ver logs em tempo real
docker logs -f <container>

# Ver últimas linhas
docker logs --tail 50 <container>

# Container morreu?
docker ps -a
docker logs <container>
```
## 🧠 Docker — docker exec

```bash
# CASO 1: Abrir um shell (mais liberdade, debug manual)
docker exec -it <container> sh

# → entra no container como se fosse um terminal
# → permite usar cd, ls, export, etc.

# CASO 2: Rodar comando direto (rápido, script, produção)
docker exec <container> ls

# → executa apenas o comando
# → não entra no shell
# → ideal para automação e CI/CD

# Observações
# - <container> é NOME ou ID (não é imagem)
# - o container precisa estar RODANDO
```

## 🧠 Docker — Conceito de Portas

```bash
# Formato
-p HOST:CONTAINER

# Exemplo real
# Aplicação roda na porta 3000 dentro do container
docker run -d -p 80:3000 app:v1

# CONTAINER (3000)
# → onde a aplicação escuta

# HOST (80)
# → onde você acessa

# Acesso no navegador
http://localhost:80

# Fluxo
# navegador → localhost:80 → docker → container:3000
```

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

```bash
docker volume inspect meu-volume
```

```bash
docker volume ls
Para listar os volumes disponíveis no Docker
```

```bash
docker volume rm app
Para remover volume disponíveil no Docker
```


Usar volume:

```bash
docker run -it -v meu-volume:/dados brunomanzini/app:v1 sh

docker run → cria e executa um container

-it → modo interativo (terminal)

-v meu-volume:/dados → monta o volume no caminho /dados

brunomanzini/app:v1 → imagem usada

sh → abre um shell dentro do container

```

---

📂 Copiando Arquivos do Host para o Container
```bash
Usando Dockerfile (build):

COPY arquivo.txt /app/arquivo.txt


Usando volume (bind mount):

docker run -v $(pwd)/arquivos:/app nginx


Copiar com container rodando:

docker cp arquivo.txt meu-container:/tmp/arquivo.txt
```

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

## 🌍 Conteúdo 8 — Docker Hub (Subir e Baixar Imagens)

### 📌 O que é Docker Hub

Docker Hub é um **repositório de imagens Docker**, parecido com o GitHub, usado para armazenar e compartilhar imagens.

---

### 🔐 Login no Docker Hub

```bash
docker login
```

* Use seu usuário e senha do Docker Hub

---

### 🏷️ Taguear imagem para o Docker Hub

Formato obrigatório:

```
usuario/nome-da-imagem:versao
```

Exemplo:

```bash
docker tag app:v1.0 brunomori/app:v1.0
```

---

### ⬆️ Enviar imagem para o Docker Hub (push)

```bash
docker push brunomori/app:v1.0
```

Após o push, a imagem ficará disponível no Docker Hub.

---

### ⬇️ Baixar imagem do Docker Hub (pull)

```bash
docker pull brunomori/app:v1.0
```

---

### ▶️ Rodar imagem baixada

```bash
docker run -dp 3000:3000 brunomori/app:v1.0
```

Resumo rápido:

* `docker push` → envia imagem para o repositório
* `docker pull` → baixa imagem para o ambiente local

---

## 📌 Boas Práticas

* Use imagens pequenas (alpine)
* Nomeie bem imagens e containers
* Evite rodar containers como root
* Documente Dockerfiles

---

## 🧪 Exemplo Prático — Do Dockerfile ao Container Rodando

### Cenário

Subir uma aplicação simples em Node.js usando Docker e acessar pelo navegador.

### Estrutura do projeto

```
app/
 ├─ app.js
 └─ Dockerfile
```

### app.js

```js
const http = require('http');

http.createServer((req, res) => {
  res.end('Docker funcionando!');
}).listen(3000);
```

### Dockerfile

```Dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
```

### Build da imagem

```bash
docker build -t app-node .
```

### Executar o container

```bash
docker run -p 3000:3000 app-node
```

Acesse no navegador:

```
http://localhost:3000
```

Resultado esperado:

```
Docker funcionando!
```

# Containers – Comandos Essenciais

| Ação                          | Comando Exemplo                                      |
|-------------------------------|------------------------------------------------------|
| Criar container               | `docker run ubuntu`                                  |
| Nomear container              | `docker run --name meu_container ubuntu`             |
| Ver logs                      | `docker logs meu_container`                          |
| Publicar portas               | `docker run -p 8080:80 nginx`                        |
| Executar interativo           | `docker run -it ubuntu bash`                         |
| Iniciar container             | `docker start meu_container`                         |
| Parar container               | `docker stop meu_container`                          |
| Remover container             | `docker rm meu_container`                            |
| Usar volume persistente       | `docker run -v /meu_dir:/dados ubuntu`               |
| Copiar arquivo host → container| `docker cp arquivo.txt meu_container:/home`         |

---

# Docker Compose – Comandos Essenciais

| Ação                          | Comando Exemplo                                      |
|-------------------------------|------------------------------------------------------|
| Criar arquivo Compose         | `docker-compose.yml` (definir serviços, volumes, redes) |
| Subir serviços                | `docker-compose up -d`                               |
| Derrubar serviços             | `docker-compose down`                                |
| Ver logs                      | `docker-compose logs`                                |
| Escalar serviços              | `docker-compose up -d --scale web=3`                 |
| Listar serviços ativos        | `docker-compose ps`                                  |
| Executar comando em serviço   | `docker-compose exec web bash`                       |
| Recriar containers            | `docker-compose up -d --force-recreate`              |
| Atualizar imagens             | `docker-compose pull`                                |
| Construir imagens             | `docker-compose build`                               |

# Explicação do docker-compose.yml

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: exemplo123
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
  ```

  Explicação:

* `version: '3'` → define a versão da sintaxe do Docker Compose.  
* `services:` → lista os serviços (containers) que serão criados.  
* `web:` → serviço que usa a imagem oficial do Nginx.  
* `image: nginx` → especifica a imagem usada.  
* `ports: "8080:80"` → mapeia a porta 80 do container para a porta 8080 do host.  
* `db:` → serviço que usa a imagem oficial do MySQL.  
* `image: mysql` → especifica a imagem usada.  
* `environment:` → define variáveis de ambiente.  
* `MYSQL_ROOT_PASSWORD: exemplo123` → senha do usuário root do banco.  
* `volumes: db_data:/var/lib/mysql` → cria persistência de dados, mapeando o volume `db_data` para o diretório interno do MySQL.  
* `volumes:` → declara volumes persistentes.  
* `db_data:` → volume nomeado usado pelo serviço `db` para manter os dados mesmo após reiniciar ou remover o container.  


🧠 Objetivo: servir como **cola rápida** e base sólida para SRE Jr / DevOps.
