<h1 align="center">SD WebUI Docker</h1>
<p align="center">
  <a href="https://github.com/thirdscam/stable-diffusion-webui-docker/actions">
    <img alt="Build Status" src="https://github.com/thirdscam/stable-diffusion-webui-docker/actions/workflows/release-image.yml/badge.svg">
  </a>
</p>

Originally https://github.com/AbdBarho/stable-diffusion-webui-docker

## Features
- Supports latest A1111, Comfy
- Prebuilt Images
- Using latest `torch`
- (A1111 Only) Replace `xformers` to `opt-sdp-no-mem-attention(torch >= 2.0.0)` (Default)
- apt, pip Packages can be installed (see Compose Setting Guide)
- Auto-installs the requirements.txt included in the extension

## Usage
#### A1111
```sh
git clone https://github.com/thirdscam/SD-WebUI-Docker.git
cd SD-WebUI-Docker
cp examples/docker-compose.a1111.yml docker-compose.yml
docker compose up -d --build
```
#### Comfy
```sh
git clone https://github.com/thirdscam/SD-WebUI-Docker.git
cd SD-WebUI-Docker
cp examples/docker-compose.comfy.yml docker-compose.yml
docker compose up -d --build
```
#### Update
```sh
# In WorkDir
docker compose build --no-cache
docker compose up -d
```
#### On Podman
```sh
# Install
git clone https://github.com/thirdscam/SD-WebUI-Docker.git
cd SD-WebUI-Docker
cp examples/docker-compose.a1111.yml docker-compose.yml # or docker-compose.comfy.yml
podman-compose up -d --build

# Update
podman-compose build --no-cache
podman-compose up -d
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
    image: ghcr.io/thirdscam/sd-webui-docker-a1111:latest
    # Ports (Default == 7890)
    # Ports follow the following rules: "External:Internal(Docker Container)"
    # You can change port number without change CLI_ARGS,
    # Like This: "8888:7890".
    ports:
      - "7890:7890"
    # Volumes
    volumes:
    # Volumes follow the following rules: "External:Internal(Docker Container)"
    # see https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes
      - ./data:/data
      - ./output:/output
    stop_signal: SIGINT
    environment:
      # CLI_ARGS: Command Line Arguments.
      # https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings
      # If you want, you can add other Command Line Arguments by referring to the link above.

      # By default, it runs through `--opt-sdp-no-mem-attention` without using xformers.
      # If you want, you can run it through xformers by deleting `--opt-sdp-no-mem-attention` and inserting `--xformers`.
      - CLI_ARGS=--listen --port 7890 --enable-insecure-extension-access --api --theme=dark --no-half-vae --opt-sdp-no-mem-attention

      # APT_ARGS, PIP_ARGS: This is where you write the apt and pip packages to install.
      # Install some package if you want.
      - APT_ARGS=
      # ex) APT_ARGS=package1 package2 ...
      # equals apt-get install -y package1 package2 ...
      - PIP_ARGS=
      # ex) PIP_ARGS=package1 package2 ...
      # equals pip install package1 package2 ...

      # Tip: If you want to do "pip install -U SomePackage", you can do "-U SomePackage" on the PIP_ARGS.
      # Like This: PIP_ARGS=-U SomePackage
      # Same at "--user SomePackage"
      # PIP_ARGS=--user SomePackage
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
  a1111:
    container_name: sd-a1111
    image: ghcr.io/thirdscam/sd-webui-docker-a1111:latest
    ports:
      - "7890:7890"
    # Volumes
    volumes:
      - ./data:/data
      - mydrive:/output
    stop_signal: SIGINT
    environment:
      - CLI_ARGS=--listen --port 7890 --enable-insecure-extension-access --api --theme=dark --no-half-vae
    deploy:
      resources:
        reservations:
          devices:
              - driver: nvidia
                device_ids: ['0']
                capabilities: [gpu,compute,utility]

# Google Drive Mount (using rclone)
# https://rclone.org/docker/
volumes:
  mydrive:
    driver: rclone
    driver_opts:
      remote: 'MyDrive:MyOutputFolderName'
      allow_other: 'true'
      vfs_cache_mode: full
      poll_interval: 0
```