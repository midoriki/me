---
title: Sử dụng Docker để chạy ứng dụng NodeJS
date: 2019-06-24 12:06:31
tags: docker nodejs
---

## Mục đích

Ban đầu khi blog này được triển khai trên server, mình đã cài đặt thủ công tất cả các thành phần cần thiết:

+ NodeJS
+ PM2
+ Redis
+ Nginx
+ Letsencrypt

Sau đó là đến phần chạy letsencrypt để lấy SSL key, cài đặt PM2 để chạy ứng dụng node, cấu hình nginx để kết nối web request đến ứng dụng node ...

Nói thì nhanh vậy nhưng để hoàn thành mất cả 2 ngày, phần vì mình vẫn còn non tay, nhưng k thể phủ nhận cài đặt một server từ đầu thủ công rất mất thời gian, lại nhiều lỗi vặt liên quan đến các gói phụ thuộc, phân quyền, blah blah.

Vì vậy mình quyết định dùng docker để đóng gói môi trường cho project này.

Một lý do nữa là sử dụng docker có thể dễ dàng cho việc tích hợp triển khai tự động, bằng cách dựng một docker image sau đó triển khai image này thay vì việc phải cập nhật code mới trên server.

## Cấu hình Docker

### Thành phần

Bằng cách nhìn vào các thành phần mà mình phải cài đặt trên server ở phần trên, chúng ta có thể nhận thấy docker cần phải có những services sau:

+ NodeJS server có cài đặt PM2 để chạy ứng dụng
+ Redis server sử dụng cho ứng dụng
+ Nginx server để làm web interface
+ Letsencrypt server để đăng ký SSL

Mình quyết định không cài đặt service Letsencrypt, bởi hiện tại trên server thật đã có sẵn SSL key, hơn nữa mỗi lần rebuild chúng ta lại request key SSL mới xem ra không có lợi cho Letsencrypt.

Như vậy hệ thống của chúng ta bao gồm 3 service.

Với redis và nginx chúng ta có thể sử dụng sẵn các image chính thức của nhà phát triển, như vậy chúng ta chỉ cần build ứng dụng nodejs mà thôi.

### Dockerfile cho ứng dụng NodeJS

Ứng dụng NodeJS của chúng ta gồm 2 thành phần:

+ Server: là một expressjs app, source code nằm trong thư mục ./server
+ Frontend: là tập hợp các app frontend dưới dạng static và serve bởi expressjs, nằm trong thư mục ./app

Như vậy image của chúng ta cần phải có base là nodejs, ngoài ra cần phải được cài đặt PM2, may mắn là PM2 đã dựng sẵn một docker image của họ, chúng ta có thể sử dụng ngay.

Dockerfile để build ứng dụng của chúng ta như sau

```Dockerfile
# PM2 base image
FROM keymetrics/pm2:latest-alpine

# Change working directory
WORKDIR /midoriki

# Bundle APP files
COPY app app/
COPY server server/
COPY package.json .
COPY .env .
COPY ecosystem.config.js .

# Install app dependencies
ENV NPM_CONFIG_LOGLEVEL warn
RUN npm install --production

# Expose the listening port of your app
EXPOSE 8080

# Show current folder structure in logs
RUN ls -al -R

# PM2 start command
CMD [ "pm2-runtime", "start", "ecosystem.config.js" ]
```

### Docker compose

Tiếp theo chúng ta sử dụng docker compose để định nghĩa các services cần thiết

File docker-compose.yml sẽ có nội dung như sau

```yaml
version: '3'
services:
  redis:
    image: "redis:alpine"
    container_name: midoriki_redis
    networks:
      - app-network
  app:
    build:
      context: .
      dockerfile: docker/Dockerfile
    container_name: midoriki_app
    depends_on:
      - redis
    networks:
      - app-network
  web:
    image: "nginx"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx.conf:/etc/nginx/conf.d
      - /etc/letsencrypt:/etc/letsencrypt
    container_name: midoriki_web
    depends_on:
      - app
    networks:
      - app-network
networks:
  app-network:
    driver: bridge
```

Chúng ta đã định nghĩa 3 services theo như thiết kế bên trên, mở 2 cổng 80 và trên service nginx để giao tiếp với web request.

Điểm đặc biệt đó là chúng ta mount 2 volume vào service nginx:

+ /etc/letsencrypt: mount thư mục letsencrypt chứa SSL key vào nginx container
+ nginx.conf: file cấu hình cho nginx để làm cầu nối giữa web request và ứng dụng nodejs

### Cấu hình nginx

```nginx
server {
    server_name _;
    return 404;
}
server {
    listen 80;
    server_name midoriki.com www.midoriki.com;
    return https://$host$request_uri;
}
server {
    listen ssl http2;
    server_name midoriki.com www.midoriki.com;
    ssl_certificate /etc/nginx/ssl/midoriki.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/midoriki.com/privkey.pem;
    location / {
        proxy_pass http://app:8080;
        .
        . <other options here>
        .
    }
}
```

Cấu hình này thỏa mãn các tác vụ sau:

+ Trả về cho tất cả các request k trùng với server name bên dưới.
+ Chuyển hướng http sang https
+ Thiết lập SSL với public và secret key từ Letsencrypt
+ Nhận request https và chuyển sang cho ứng dụng nodejs - ở đây là phần

```nginx
proxy_pass http://app:8080;
```

với app là tên service mà chúng ta đặt ứng dụng nodejs, định nghĩa ở file docker-compose.yml.

## Triển khai

Sau khi đã hoàn thành các cài đặt, chúng ta có thể tiến hành deploy hệ thống.

Trước hết chúng ta cần cài đặt docker trên server

```bash
sudo apt-get update

sudo yum install -y yum-utils device-mapper-persistent-data lvm2

sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker

# add current user into docker group, to bypass sudo when run docker command
sudo groupadd dockerdocker $USER

# enable docker on startup
sudo systemctl enable docker

# verify docker was installed successfully
docker info
```

Sau đó tiếp tục cài đặt docker-compose

```
sudo yum install -y python python-pip

pip install --user docker-compose

docker-compose --version
```

Tiếp theo, chúng ta clone source code từ github và khởi chạy docker-compose

```bash
cd <somewhere you want>

git clone git@github.com:midoriki/midoriki.git

cd midoriki

docker-compose up -d

# check if docker containers are running
docker ps
```

Như vậy chúng ta đã cài đặt thành công một hệ thống chạy trên docker.
