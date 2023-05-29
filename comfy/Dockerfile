FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

RUN apt-get update && apt-get install -y git python3 python3-pip && apt-get clean

RUN --mount=type=cache,target=/root/.cache/pip pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118

ENV ROOT=/apt
RUN --mount=type=cache,target=/root/.cache/pip \
  git clone https://github.com/comfyanonymous/ComfyUI.git ${ROOT} && \
  cd ${ROOT} && \
  pip install -r requirements.txt \
  pip install -U xformers

RUN --mount=type=cache,target=/root/.cache/pip  \
  pip install triton

WORKDIR ${ROOT}

# add info
COPY entrypoint.sh /docker/entrypoint.sh
RUN chmod +x /docker/entrypoint.sh

ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility NVIDIA_VISIBLE_DEVICES=all
ENV PYTHONPATH="${PYTHONPATH}:${PWD}" CLI_ARGS=""
ENTRYPOINT ["/docker/entrypoint.sh"]
CMD python3 -u main.py ${CLI_ARGS}