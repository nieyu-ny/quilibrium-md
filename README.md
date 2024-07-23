# quilibrium-md
1、检查是否安装docker

docker --version

未安装则执行docker安装命令：

curl -fsSL https://get.docker.com | bash -s docker

2、创建docker构建文件

mkdir -p $HONE/quili

cd $HONE/quili

vim docker-compose.yml

# 将下面的内容复制到docker-compose.yml中

name: quilibrium

services:
  node:
    image: mscurtescu/ceremonyclient:latest
    restart: unless-stopped
    command: ["--signature-check=false"]
    deploy:
      resources:
        limits:
          cpus: "6"
          memory: "16G"
        reservations:
          cpus: "6"
          memory: "16G"
    environment:
      - DEFAULT_LISTEN_GRPC_MULTIADDR=/ip4/0.0.0.0/tcp/8337
      - DEFAULT_LISTEN_REST_MULTIADDR=/ip4/0.0.0.0/tcp/8338
      - DEFAULT_STATS_MULTIADDR=/dns/stats.quilibrium.com/tcp/443
    ports:
      - '${QUILIBRIUM_P2P_PORT:-8336}:8336/udp' # p2p
      - '127.0.0.1:${QUILIBRIUM_GRPC_PORT:-8337}:8337/tcp' # gRPC
      - '127.0.0.1:${QUILIBRIUM_REST_PORT:-8338}:8338/tcp' # REST
    healthcheck:
      test: ["CMD", "grpcurl", "-plaintext", "localhost:8337", "list", "quilibrium.node.node.pb.NodeService"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 1m
    volumes:
      - ./.config:/root/.config
    logging:
      driver: "json-file"
      options:
        max-file: "5"
        max-size: 2048m

# 复制完成后 ctr + c 再 :wq  保存文件

5、启动quilibrium

docker compose up -d

6、查看启动日志

docker ps -a

![image](https://github.com/user-attachments/assets/e7a12ce0-585e-4a29-a220-525303295e83)

docker logs --tail 50 -f 容器id

日志中出现下面的信息则表示安装成功并正在同步（首次启动需要一点时间，请耐心等待）

![image](https://github.com/user-attachments/assets/a6a01e5d-0d6f-408e-aabc-b7a928810d8b)

