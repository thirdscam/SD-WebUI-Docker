FROM alpine/git:2.36.2 as download

COPY clone.sh /clone.sh

RUN . /clone.sh taming-transformers https://github.com/CompVis/taming-transformers.git \
  && rm -rf data assets **/*.ipynb

RUN . /clone.sh stable-diffusion-stability-ai https://github.com/Stability-AI/stablediffusion.git \
  && rm -rf assets data/**/*.png data/**/*.jpg data/**/*.gif

RUN . /clone.sh CodeFormer https://github.com/sczhou/CodeFormer.git \
  && wget -O misc.py https://pastebin.com/raw/1AKZ9kGy \
  && mv misc.py basicsr/utils/misc.py \
  && rm -rf assets inputs

RUN . /clone.sh BLIP https://github.com/salesforce/BLIP.git
RUN . /clone.sh k-diffusion https://github.com/crowsonkb/k-diffusion.git
RUN . /clone.sh clip-interrogator https://github.com/pharmapsychotic/clip-interrogator

FROM nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04

RUN apt-get update && apt install -y python3 python3-pip

ENV DEBIAN_FRONTEND=noninteractive PIP_PREFER_BINARY=1

RUN --mount=type=cache,target=/root/.cache/pip \
  pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118

RUN apt install fonts-dejavu-core rsync git jq moreutils wget -y && apt-get clean

RUN --mount=type=cache,target=/root/.cache/pip \
  git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git apt && \
  cd apt && \
  pip install -r requirements_versions.txt && \
  pip install --pre xformers

RUN --mount=type=cache,target=/root/.cache/pip \
  pip install triton

ENV ROOT=/apt

COPY --from=download /repositories/ ${ROOT}/repositories/
RUN mkdir ${ROOT}/interrogate && cp ${ROOT}/repositories/clip-interrogator/clip_interrogator/data/* ${ROOT}/interrogate
RUN --mount=type=cache,target=/root/.cache/pip \
  pip install -r ${ROOT}/repositories/CodeFormer/requirements.txt

RUN --mount=type=cache,target=/root/.cache/pip \
  pip install pyngrok \
  git+https://github.com/TencentARC/GFPGAN.git \
  git+https://github.com/openai/CLIP.git \
  git+https://github.com/mlfoundations/open_clip.git

RUN apt-get install -y libgoogle-perftools-dev libgl1-mesa-glx libglib2.0-0 libsm6 libxrender1 libxext6 && apt-get clean
ENV LD_PRELOAD=libtcmalloc.so

RUN --mount=type=cache,target=/root/.cache/pip \
  pip install -U opencv-python-headless scikit-learn send2trash mediapipe ultralytics dynamicprompts[attentiongrabber,magicprompt]

COPY config.py /docker/config.py
COPY config.json /docker/config.json
COPY entrypoint.sh /docker/entrypoint.sh
RUN chmod +x /docker/entrypoint.sh

RUN \
  mv ${ROOT}/style.css ${ROOT}/user.css && \
  sed -i 's/in_app_dir = .*/in_app_dir = True/g' /usr/local/lib/python3.10/dist-packages/gradio/routes.py

WORKDIR ${ROOT}
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility
ENV NVIDIA_VISIBLE_DEVICES=all
ENV CLI_ARGS=""
ENTRYPOINT ["/docker/entrypoint.sh"]

# Cleanup
RUN cp webui.py core.py
RUN rm webui-macos-env.sh webui-user.bat webui-user.sh webui.bat webui.sh webui.py screenshot.png CODEOWNERS README.md environment-wsl2.yaml
RUN wget -O misc.py https://pastebin.com/raw/1AKZ9kGy \
  && mv misc.py /usr/local/lib/python3.10/dist-packages/basicsr/utils/misc.py

CMD test -z "${APT_ARGS}" || apt-get -y install ${APT_ARGS} \
    && test -z "${PIP_ARGS}" || pip install ${PIP_ARGS} \
    && python3 -u core.py ${CLI_ARGS}