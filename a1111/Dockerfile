FROM nvidia/cuda:12.1.0-cudnn8-devel-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PIP_PREFER_BINARY=1
ENV ROOT=/app

RUN mkdir ${ROOT}
WORKDIR ${ROOT}

# Install Essential Packages (APT)
RUN apt-get update \
 && apt-get install -y python3 python3-pip fonts-dejavu-core rsync git jq moreutils wget \
                       libgoogle-perftools-dev libgl1-mesa-glx libglib2.0-0 libsm6 libxrender1 libxext6 \
 && apt-get clean

# Get SD
RUN git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git .

# Get Modules
RUN mkdir repositories
RUN git clone https://github.com/CompVis/taming-transformers.git repositories/taming-transformers
RUN git clone https://github.com/Stability-AI/stablediffusion.git repositories/stable-diffusion-stability-ai
RUN git clone https://github.com/Stability-AI/generative-models.git repositories/generative-models
RUN git clone https://github.com/sczhou/CodeFormer.git repositories/CodeFormer
RUN git clone https://github.com/salesforce/BLIP.git repositories/BLIP
RUN git clone https://github.com/crowsonkb/k-diffusion.git repositories/k-diffusion
RUN git clone https://github.com/pharmapsychotic/clip-interrogator repositories/clip-interrogator

# https://github.com/pharmapsychotic/clip-interrogator/commit/7a2ac9aa57766748d7dfec8c7cddff8a4b8b143e
RUN mkdir interrogate && cp repositories/clip-interrogator/clip_interrogator/data/* interrogate

# Install Essential Packages (PIP)
RUN --mount=type=cache,target=/root/.cache/pip \
  pip install torch torchvision && \
  pip install -r requirements_versions.txt && \
  pip install -r repositories/CodeFormer/requirements.txt && \
  pip install --pre xformers && \
  pip install triton pyngrok dctorch scikit-learn send2trash mediapipe ultralytics opencv-python-headless dynamicprompts[attentiongrabber,magicprompt] \
  git+https://github.com/TencentARC/GFPGAN.git \
  git+https://github.com/openai/CLIP.git \
  git+https://github.com/mlfoundations/open_clip.git

# Fix Bug (cannot import name 'get_device' by https://github.com/sczhou/CodeFormer/issues/194)
RUN cp repositories/CodeFormer/basicsr/utils/misc.py /usr/local/lib/python3.10/dist-packages/basicsr/utils/misc.py

# Setting up
COPY config.py /docker/config.py
COPY config.json /docker/config.json
COPY entrypoint.sh /docker/entrypoint.sh
RUN chmod +x /docker/entrypoint.sh
ENTRYPOINT ["/docker/entrypoint.sh"]

# Cleaning up
RUN cp webui.py core.py
RUN rm webui-macos-env.sh webui-user.bat webui-user.sh webui.bat webui.sh webui.py screenshot.png CODEOWNERS README.md environment-wsl2.yaml

CMD test -z "${APT_ARGS}" || apt-get -y install ${APT_ARGS} \
    && test -z "${PIP_ARGS}" || pip install ${PIP_ARGS} \
    && find /data/config/auto/extensions -type f -name 'requirements.txt' -exec pip install -q -r '{}' \; \
    && python3 -u core.py ${CLI_ARGS}
