# frontend
masuk ke direktori frontend

pindah ke branch production
```shell
git branch production
```
![](.10setup_production_images/8b1951ad.png)

buat file .dockerignore
![](.10setup_production_images/038d7235.png)

buat file .env
![](.10setup_production_images/969a445a.png)

buat Dockerfile
```shell
FROM node:16-alpine as build
WORKDIR /home/app
COPY . .
RUN npm install --production
ARG REACT_APP_BASEURL="https://api.reiya.my.id/api/v1"
RUN npm run build

FROM node:16-alpine
WORKDIR /home/app
COPY --from=build /home/app /home/app
RUN npm install -g serve
EXPOSE 3000
CMD ["serve","-s","build"]
```
![](.10setup_production_images/510cf666.png)

build Dockerfile
```shell
docker build -t reiya24/dumbmerch-frontend-production . --progress=plain --no-cache
```
![](.10setup_production_images/ec0d1031.png)

buat docker compose
```shell
version: '3.7'
services:
  frontend-production:
    container_name: frontend-production
    image: reiya24/dumbmerch-frontend-production
    stdin_open: true
    restart: unless-stopped
    ports:
      - 3000:3000
```
![](.10setup_production_images/5d8f646c.png)

jalankan docker compose
```shell
docker compose up -d
```
![](.10setup_production_images/455b76e8.png)

simpan perubahan ke git
```shell
git add . && git commit -m "setup docker"
```
![](.10setup_production_images/829428fb.png)

# database

inisialisasi direktori database dengan git
![](.10setup_production_images/5dcc890a.png)

buat docker compose
```shell
version: '3.7'
services:
  database-production:
    image: postgres:alpine
    container_name: database-production
    restart: unless-stopped
    environment:
      - POSTGRES_USER=reiya
      - POSTGRES_PASSWORD=reiya
      - POSTGRES_DB=reiya
    ports:
      - '5432:5432'
    volumes:
      - ~/konfigurasi_posgres_production:/var/lib/postgresql/data
```
![](.10setup_production_images/bc004d7c.png)

jalankan docker compose
```shell
docker compose up -d
```
![](.10setup_production_images/c92e8983.png)

simpan perubahan di git
```shell
git add . && git commit -m "setup compose production"
```
![](.10setup_production_images/fae81adc.png)

# backend

pastikan di branch production

buat .dockerignore
```shell
.git
.gitignore
Dockerfile
docker-compose.yaml
```
![](.10setup_production_images/c9e7dfd7.png)

buat Dockerfile
```shell
FROM golang:1.18-alpine as builder
WORKDIR /home/app
COPY . .
RUN go mod download
ARG DB_HOST=10.116.106.150
ARG DB_USER=reiya
ARG DB_PASSWORD=reiya
ARG DB_NAME=reiya
ARG DB_PORT=5432
ARG PORT=5000
RUN CGO_ENABLED=0 go build

FROM gcr.io/distroless/cc-debian11
WORKDIR /home/app
COPY --from=builder /home/app/dumbmerch /home/app
COPY --from=builder /home/app/.env /home/app
EXPOSE 5000
CMD ["/home/app/dumbmerch"]
```
![](.10setup_production_images/c0281024.png)

build Dockerfile
```shell
docker build -t reiya24/dumbmerch-backend-production . --progress=plain --no-cache
```
![](.10setup_production_images/c5ee3c6d.png)

buat file docker compose
```shell
version: '3.7'
services:
 backend-production:
  image: reiya24/dumbmerch-backend-production
  container_name: backend-production
  stdin_open: true
  restart: unless-stopped
  ports:
   - 5000:5000
```
![](.10setup_production_images/f34fc58d.png)

![](.10setup_production_images/bf8e6ef4.png)

jalankan docker compose
```shell
docker compose up -d
```
![](.10setup_production_images/ac931c85.png)

simpan perubahan di git
![](.10setup_production_images/52482b50.png)

integrasi backend dengan database berhasil berjalan
![](.10setup_production_images/38e9dad1.png)

