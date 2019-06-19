---
title: Sử dụng Docker để phục vụ ứng dụng NodeJS trên server
date: 2019-06-18 14:34:17
tags: docker nodejs nginx
---

## Mục đích

Ban đầu khi blog này được triển khai trên server mình cài đặt thủ công tất cả các thành phần:

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

Chúng ta đã định nghĩa 3 services theo như thiết kế bên trên, mở 2 cổng 80 và 443 trên service nginx để giao tiếp với web request.

Điểm đặc biệt đó là chúng ta mount 2 volume vào service nginx:

+ /etc/letsencrypt: mount thư mục letsencrypt chứa SSL key vào nginx container
+ nginx.conf: file cấu hình cho nginx để làm cầu nối giữa web request và ứng dụng nodejs

File cấu hình nginx như sau

```nginx
server {
    server_name _;
    return 404;
}
server {
    listen 80;
    server_name midoriki.com www.midoriki.com;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    server_name midoriki.com www.midoriki.com;
    if ($host != "midoriki.com") {
        return 301 https://midoriki.com$request_uri;
    }
    ssl_certificate /etc/nginx/ssl/midoriki.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/midoriki.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH;AES256+EDH';
    location / {
        proxy_pass http://app:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_redirect off;
        proxy_http_version 1.1;
        proxy_cache_bypass $http_upgrade;
        add_header Strict-Transport-Security: max-age=31536000;
        add_header X-Frame-Options: sameorigin;
    }
}
```

Cấu hình này thỏa mãn các tác vụ sau:

+ Trả về 404 cho tất cả các request k trùng với server name bên dưới.
+ Chuyển hướng http sang https
+ Thiết lập SSL với public và secret key từ Letsencrypt
+ Nhận request https và chuyển sang cho ứng dụng nodejs - ở đây là phần

```nginx
proxy_pass http://app:8080;
```

với app là tên service định nghĩa ở file docker-compose.yml.