# stable-diffusion-webui-docker

Originally https://github.com/AbdBarho/stable-diffusion-webui-docker

## Usage
#### Comfy
```sh
git clone https://github.com/thirdscam/stable-diffusion-webui-docker.git
cd stable-diffusion-webui-docker
cp examples/docker-compose.comfy.yml docker-compose.yml
docker compose up -d
```
#### A1111
```sh
git clone https://github.com/thirdscam/stable-diffusion-webui-docker.git
cd stable-diffusion-webui-docker
cp examples/docker-compose.a1111.yml docker-compose.yml
docker compose up -d
```

By default, The model and configuration files are stored in `./data` and the results are stored in the `./output` folder.

## Compose Setting Guide
```yml
version: '3.9'
services:
  # (a1111, comfy) == Default Service Name,
  # (sd-a1111, sd-comfy) == Container Name.
  # You can write a name whatever you want.
  a1111:
    container_name: sd-a1111
    # Image (a1111 or comfy)
    image: ghcr.io/thirdscam/stable-diffusion-webui-a1111:latest
    # Ports (Default == 7890)
    # Ports follow the following rules: "External:Internal(Docker Container)"
    # You can change port number without change CLI_ARGS,
    # Like This: "8888:7890".
    ports:
      - "7890:7890"
    # Volumes
    volumes:
      - ./data:/data
      - ./output:/output
    stop_signal: SIGINT
    environment:
      - CLI_ARGS=--listen --port 7890 --enable-insecure-extension-access --api --theme=dark --no-half-vae
    deploy:
      resources:
        reservations:
          devices:
              - driver: nvidia
                # If it's a multi-GPU environment,
                # you can use the GPU with the number you want to use.
                # Example - device_ids: ['1']
                # In addition, sd-webui does not support Multi-GPU,
                # so it is right to assign only one graphics card.
                device_ids: ['0']
                capabilities: [gpu,compute,utility]
```

## Advanced Compose Setting Guide
#### Google Drive (For Outputs)
```yml
version: '3.9'
services:
  # (a1111, comfy) == Default Service Name,
  # (sd-a1111, sd-comfy) == Container Name.
  # You can write a name whatever you want.
  a1111:
    container_name: sd-a1111
    # Image (a1111 or comfy)
    image: ghcr.io/thirdscam/stable-diffusion-webui-a1111:latest
    # Ports (Default == 7890)
    # Ports follow the following rules: "External:Internal(Docker Container)"
    # You can change port number without change CLI_ARGS,
    # Like This: "8888:7890".
    ports:
      - "7890:7890"
    # Volumes
    volumes:
      - ./data:/data
      - mydrive:/output
    stop_signal: SIGINT
    environment:
      - CLI_ARGS=--listen --port 7890 --enable-insecure-extension-access --api --theme=dark --no-half-vae
    # GPU Setting (For Multi-GPU, etc..)
    # If you are using only one graphics card, leave it as it is.
    deploy:
      resources:
        reservations:
          devices:
              - driver: nvidia
                device_ids: ['0']
                capabilities: [gpu,compute,utility]

volumes:
  mydrive:
    driver: rclone
    driver_opts:
      remote: 'MyDrive:MyOutputFolderName'
      allow_other: 'true'
      vfs_cache_mode: full
      poll_interval: 0
```