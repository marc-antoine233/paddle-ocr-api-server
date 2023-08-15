# 部署说明

本部署主要参考 [掘金](https://juejin.cn/post/7194803742032527420)

考虑到国内网络，`wget` 部分均使用国内源

使用的了两个中文模型，检测模型和识别模型：

- ch_PP-OCRv3_det_infer
- ch_PP-OCRv3_rec_infer

其他模型见 [github](https://github.com/PaddlePaddle/PaddleOCR/blob/release/2.5/doc/doc_ch/models_list.md)

## docker 镜像

可以使用 [dockerfile](./dockerfile) 自行 build，也可以拉取现有的镜像：

## docker compose 部署

### 部署前准备

在 [docker-compose.yml](./docker-compose.yml) 所在目录中创建 `.env` 文件，内容为：

```.env
hostip=10.1.1.2
device_ids='0'
```

其中 `hostip` 为给容器分配的 ip，`device_ids` 为给容器分配的宿主机 GPU 的编号

[docker-compose.yml](./docker-compose.yml) 中需要根据需要调整 `network` 部分

!!! tip
    如果上述操作均不想做可以直接使用下面的 `docker-compose.yml`

```yml
version: '3.9'
name: paddle-ocr-serving
services:
paddle-ocr-serving:
    build:
    context: .
    image: marcantoine153/paddle-ocr-serving:latest
    container_name: ocr-serving
    stdin_open: true
    tty: true
    restart: always
    # paddle 官网建议是 64G，可以自行调整
    shm_size: '64g'
    volumes:
      - ./volume/config.yml:/home/PaddleOCR/deploy/pdserving/config.yml
    deploy:
    resources:
        reservations:
        devices:
            - driver: nvidia
                device_ids: ['0']
                capabilities: [gpu]    
```

### 部署

```bash
docker compose up -d
```

## API 调用

`python` 调用脚本参考 [官网脚本](https://raw.githubusercontent.com/PaddlePaddle/PaddleOCR/release/2.7/deploy/pdserving/pipeline_http_client.py)，可以参考 [demo](demo/ocr.ipynb)

`python` 调用不需要额外的包，一般只需要保持 python 版本在3.0以上即可
