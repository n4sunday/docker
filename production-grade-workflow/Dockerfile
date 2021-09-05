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