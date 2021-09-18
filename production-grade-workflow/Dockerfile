FROM node:alpine as builder
WORKDIR '/app'
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

RUN npm install -g serve
CMD ["serve", "-p", "5000", "-s", "./dist"]
