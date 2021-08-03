# syntax=docker/dockerfile:1
# Stage1: Build React App
FROM node:14.17.3-alpine3.14 as build
WORKDIR /app
COPY ["package.json", "package-lock.json*", "./"]
RUN npm install --production
COPY . .
RUN npm run build

#Stage2: Build final image with Nginx
FROM nginx:1.21.1-alpine as server
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
ENTRYPOINT ["nginx","-g","daemon off;"]

#docker build -f Dockerfile_Frontend -t {DockerImageTag} .
#docker run -p 80:80 {DockerImageTag}
