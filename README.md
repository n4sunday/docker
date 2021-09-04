## 🐳 Docker

---

#### 🚀 Basic Command

##### Create and Running a Container from as Image

```sh
docker run <image name>
```

##### List all running containers

```sh
docker ps
```

##### Create a Container

```sh
docker create <image name>
```

##### Start a Container

```sh
docker start <container id>
```

##### Stop All

```sh
docker system prune
```

##### Get logs from a container

```sh
docker logs <container id>
```

##### Stop container

```sh
docker stop <container id>
```

##### Kill container

```sh
docker kill <container id>
```

##### Execute an additional command in a container

```sh
docker exec -it <container id> <command>
```

Command

- `bash`
- `powershell`
- `zsh`
- `sh`

---

#### 🚀 Building and Custom Image

##### Basic Build

📄 Dockerfile

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

###### Tagging an Image

```sh
docker build -t n4sunday/redis:latest .
docker build -t <your docker id>/<repo | project name>:<version> .
```

---

#### 🚀 Making Real Project with Docker

##### Basic Node Project

🗃️ Structure

```sh
project
  ├── index.js
  ├── package.json
  └── Dockerfile
```

📄 index.js

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

📄 Dockerfile

```Dockerfile
# Specify a base image
FROM alpine

# Install some depenendencies
RUN npm install

# Default command
CMD ["npm","start"]
```