# 🐳 Docker

## 🚀 Basic Command

Create and Running a Container from as Image

```sh
docker run <image name>
```

List all running containers

```sh
docker ps
```

Create a Container

```sh
docker create <image name>
```

Start a Container

```sh
docker start <container id>
```

Stop All

```sh
docker system prune
```

Get logs from a container

```sh
docker logs <container id>
```

Stop container

```sh
docker stop <container id>
```

Kill container

```sh
docker kill <container id>
```

Execute an additional command in a container

```sh
docker exec -it <container id> <command>
```

**Command**

- `bash`
- `powershell`
- `zsh`
- `sh`

## 🚀 Building and Custom Image

📁 `redis-image`

#### Basic Build

📄 **`Dockerfile`**

```Dockerfile
# Use an exising docker image as a base
FROM alpine

# Download and install a dependency
RUN apk add --update redis

# Tell the image what to do when it starts
# as a container
CMD ["redis-server"]
```

```sh
docker build .

docker run <image id>
```

##### Tagging an Image

```sh
docker build -t n4sunday/redis:latest .
docker build -t <your docker id>/<repo | project name>:<version> .
```

## 🚀 Making Real Project with Docker

📁 `real-project`

#### Basic Node Project

🗃️ Structure

```sh
project
  ├── index.js
  ├── package.json
  └── Dockerfile
```

📄 **`index.js`**

```js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000, () => {
  console.log("Listen on port 3000");
});
```

📄 **`package.json`**

```json
{
  "name": "real-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "*"
  },
  "author": "",
  "license": "ISC"
}
```

📄 **`Dockerfile`**

```Dockerfile
# Specify a base image
FROM node:alpine

# Install some depenendencies
RUN npm install

# Default command
CMD ["npm","start"]

```

| Command |                            Description                             |                  Description                  |
| :-----: | :----------------------------------------------------------------: | :-------------------------------------------: |
|  COPY   |                                 ./                                 |                      ./                       |
|         | Path to folder to copy on `your machine` relative to build context | Place to copy stuff to inside `the container` |

update 📄 **`Dockerfile`**

```Dockerfile
# Specify a base image
FROM node:alpine

# Install some depenendencies
COPY ./ ./
RUN npm install

# Default command
CMD ["npm","start"]

```

#### 🔥 Container Port Mapping

docker run -p `<incoming request port>`:`<port inside container>` `<image id>`

```sh
docker run -p 3000:3000 <image id>
```

#### Working Directory

📄 **`Dockerfile`**

```Dockerfile
# Specify a base image
FROM node:alpine

WORKDIR /usr/app

# Install some depenendencies
COPY ./ ./
RUN npm install

# Default command
CMD ["npm","start"]
```

📄 **`Dockerfile`**

```Dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY ./package.json ./
RUN npm install
# ----- cache -----
COPY ./ ./

CMD ["npm","start"]
```

## 🚀 Docker Compose with Multiple Local Container

📁 `multiple-container`

- Used to start up multiple Docker containers
- Automates some of the long-winded arguments we were padding to 'docker run'

🗃️ Structure

```sh
project
  ├── index.js
  ├── package.json
  ├── Dockerfile
  └── docker-compose.yml
```

📄 **`index.js`**

```js
const express = require("express");
const redis = require("redis");

const app = express();
const client = redis.createClient({
  host: "redis-server",
  port: 6379,
});
client.set("visits", 0);

app.get("/", (req, res) => {
  client.get("visits", (err, visits) => {
    res.send("Number of visits is " + visits);
    client.set("visits", +visits + 1);
  });
});

app.listen(3000, () => {
  console.log("Listen on port 3000");
});
```

📄 **`Dockerfile`**

```Dockerfile
FROM node:alpine

WORKDIR /usr/app

COPY ./package.json ./
RUN npm install
COPY ./ ./

CMD ["npm","start"]

```

📄 **`package.json`**

```json
{
  "name": "multiple-container",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "*",
    "redis": "2.8.0"
  },
  "author": "",
  "license": "ISC"
}
```

📄 **`docker-compose.yml`**

```yml
version: "3"
services:
  redis-server:
    image: "redis"
  node-app:
    build: .
    ports:
      - "3000:3000"
```

#### 🔥 Commands

Launch docker compose

```sh
docker-compose up
```

Launch in background

```sh
docker-compose up -d
```

```sh
docker-compose up --build
```

Stop Containers

```sh
docker-compose down
```

#### Automatic Container Restarts

Status Codes
| Status Codes | Description |
| :----------: | :------------------------------: |
| 0 | We exited and everything is OK |
| 1, 2, 3, etc | We exited because something went wrong!|

Restart Policies
default **`no`**
| Status Codes | Description |
| :----------: | :------------------------------: |
| `no` | Never attempt to restart this . container if it stops or crashes |
| `always` | If this container stops `for any reason` always attempt to restart it |
| `on-failure` | Only restart if the container stops with an error code |
| `unless-stopped` | Always restart unless we (the developers) forcibly stop it |

📄 **`docker-compose.yml`**

```yml
version: "3"
services:
  redis-server:
    image: "redis"
  node-app:
    restart: always
    build: .
    ports:
      - "3000:3000"
```

## 🚀 Creating a Production Grade Workflow

📁 `production-grade-workflow`

#### Volumes

use docker

```sh
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app <image_id>
```

use docker compose

📄 **`Dockerfile.dev`**

```Dockerfile
FROM node:alpine

WORKDIR /app

COPY package.json .
RUN npm install

COPY . .

CMD ["npm","run","dev"]

```

📄 **`docker-compose.yml`**

```yml
version: "3"
services:
  node-app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

vite project config `usePolling: true`

📄 **`vite.config.js`**

```js
import { defineConfig } from "vite";
import reactRefresh from "@vitejs/plugin-react-refresh";

export default defineConfig({
  server: {
    host: "0.0.0.0",
    port: 3000,
    watch: {
      usePolling: true,
    },
  },
  plugins: [reactRefresh()],
});
```

## 🚀 Docker Compose for Running Tests

📄 **`docker-compose.yml`**

```yml
version: "3"
services:
  node-app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
  tests:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - /app/node_modules
      - .:/app
    command: ["npm", "run", "test"]
```

## 🚀 Implement Muti-Step Builds

📄 **`Dockerfile`**

```Dockerfile
# STEP 1
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# STEP 2
FROM nginx
COPY --from=builder /app/dist /usr/share/nginx/html
```

Run nginx

```sh
docker run -p 3000:80 <image_id>
```

## 🚀 Travis

📄 **`.travis.yml`**

```yml
sudo: required
services:
  - docker

before_install:
  - docker build -t n4sunday/docker-react -f Dockerfile.dev .

script:
  - docker run n4sunday/docker-react npm run test -- --coverage
```

## 🚀 Kubernetes (K8S)

#### 🔥 What is Kubernetes?

System for running many different containers over multiple different machines

#### 🔥 Why use Kubernetes?

When you need to run many different containers with different images

`install kubectl`
`install minikube`

```sh
minikube start
```

```sh
minikube status
```

## 🚀 NGINX

📁 `production-grade-workflow`

#### 🔥 Web Server

📄 **`Dockerfile`**

```Dockerfile
# STEP 1
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# STEP 2
FROM nginx
COPY --from=builder /app/dist /usr/share/nginx/html
```

📄 **`docker-compose.yml`**

```yml
version: "3"
services:
  front-app:
    build: .
    ports:
      - "3000:80"
```

#### 🔥 Reverse proxy
📄 **`Dockerfile`**
```Dockerfile
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

RUN npm install -g serve
CMD ["serve", "-p", "5000", "-s", "./dist"]
```

📄 **`docker-compose.yml`**

```yml
version: "3"
services:
  nginx:
    image: nginx
    container_name: nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
  front-end:
    build: .
    ports:
      - "3000:5000"
```
