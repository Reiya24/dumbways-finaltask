# frontend

pindah ke branch staging

buat file .dockerignore
![](.12setup_staging_images/2d7683f4.png)

buat Dockerfile
```shell
FROM node:12.16-alpine3.11 as build
WORKDIR /home/app
COPY . .
RUN npm install

FROM node:12.16-alpine3.11
WORKDIR /home/app
COPY --from=build /home/app /home/app
EXPOSE 3000
CMD ["npm","start"]
```
![](.12setup_staging_images/ed1b00f8.png)

build Dockerfile
![](.12setup_staging_images/5b52fd36.png)

buat file docker compose
![](.12setup_staging_images/d5f5e220.png)

jalankan docker compose
![](.12setup_staging_images/9b3759e2.png)

simpan perubahan di git
![](.12setup_staging_images/56abdac3.png)

# database
pindah ke branch staging

buat docker compose
```shell
version: '3.7'
services:
  database-staging:
    image: postgres:alpine
    container_name: database-staging
    restart: unless-stopped
    environment:
      - POSTGRES_USER=reiya
      - POSTGRES_PASSWORD=reiya
      - POSTGRES_DB=reiya
    ports:
      - '4400:5432'
    volumes:
      - ~/konfigurasi_posgres_staging:/var/lib/postgresql/data
```
![](.12setup_staging_images/13463020.png)

jalankan docker compose
![](.12setup_staging_images/3e534aec.png)

# backend

buat file .dockerignore
![](.12setup_staging_images/ad01d8cb.png)

buat Dockerfile
buat Dockerfile
```shell
FROM golang:1.18-alpine as builder
WORKDIR /home/app
COPY . .
RUN CGO_ENABLED=0 go build

FROM gcr.io/distroless/cc-debian11
WORKDIR /home/app
COPY --from=builder /home/app/dumbmerch /home/app
COPY --from=builder /home/app/.env /home/app
EXPOSE 5000
CMD ["/home/app/dumbmerch"]
```
![](.12setup_staging_images/22b25d84.png)

build Dockerfile
![](.12setup_staging_images/000e67e5.png)

buat docker compose
![](.12setup_staging_images/cb3d048a.png)

jalankan docker compose
![](.12setup_staging_images/ad0dfd8e.png)

simpan perubahan di git
![](.12setup_staging_images/699ebba0.png)

setup berhasil
![](.12setup_staging_images/f502ec48.png)
![](.12setup_staging_images/a60daf33.png)