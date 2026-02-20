ARG PYTORCH="2.4.1"
ARG CUDA="12.1"
ARG CUDNN="9"

FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0+PTX"
ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

# To fix GPG key error when running apt-get update
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/7fa2af80.pub


RUN apt-get update && apt-get install -y git ninja-build libglib2.0-0 libsm6 libxrender-dev libgl1-mesa-dev wget tar ffmpeg \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


ARG PYTORCH
ARG CUDA

RUN conda clean --all

RUN pip install -U iopath --trusted-host files.pythonhosted.org
RUN pip install torchvision==0.19.1+cu121 --index-url https://download.pytorch.org/whl/cu121 --trusted-host download.pytorch.org 


RUN wget -O cub-1.10.0.tar.gz https://github.com/NVIDIA/cub/archive/1.10.0.tar.gz
RUN tar xzf cub-1.10.0.tar.gz
RUN export CUB_HOME=$PWD/cub-1.10.0


RUN git clone -b main https://github.com/UVA-Computer-Vision-Lab/ovmono3d.git /ovmono3d
WORKDIR /ovmono3d

RUN FORCE_CUDA=1 pip install git+https://github.com/facebookresearch/pytorch3d.git@055ab3a
RUN pip install git+https://github.com/yaojin17/detectron2.git  # slightly modified detectron2 for OVMono3D

RUN pip install cython opencv-python scipy pandas einops open_clip_torch open3d

RUN pip install git+https://github.com/apple/ml-depth-pro.git@b2cd0d5
RUN pip install git+https://github.com/facebookresearch/segment-anything.git@dca509f
RUN pip install git+https://github.com/IDEA-Research/GroundingDINO.git@856dde2

RUN mkdir -p checkpoints
RUN wget -P ./checkpoints https://github.com/IDEA-Research/GroundingDINO/releases/download/v0.1.0-alpha2/groundingdino_swinb_cogcoor.pth
RUN wget -P ./checkpoints https://ml-site.cdn-apple.com/models/depth-pro/depth_pro.pt
RUN wget -P ./checkpoints https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth
RUN wget -P ./checkpoints https://huggingface.co/uva-cv-lab/ovmono3d_lift/resolve/main/ovmono3d_lift.pth

# To fix 'BertModel' object has no attribute 'get_head_mask' error
RUN pip install transformers==4.41.2

# docker build -t ovmono3d .
# docker run -it --rm --network host --gpus all ovmono3d