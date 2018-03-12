FROM nvidia/cuda:9.0-cudnn7-runtime-ubuntu16.04

LABEL maintainer="joeyism"
RUN apt-get update

# Installs basic dependencies
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y software-properties-common
RUN apt-get install -y --no-install-recommends apt-utils
RUN apt-get install -y --no-install-recommends git curl vim unzip openssh-client wget
RUN apt-get install -y --no-install-recommends build-essential cmake
RUN apt-get install -y --no-install-recommends libopenblas-dev
RUN apt-get install -y --no-install-recommends python3.5 python3.5-dev python3-pip python3-tk
RUN pip3 install --no-cache-dir --upgrade pip setuptools
RUN echo "alias python='python3'" >> /root/.bash_aliases
RUN echo "alias pip='pip3'" >> /root/.bash_aliases
RUN apt-get install -y --no-install-recommends libjpeg-dev zlib1g-dev
RUN pip3 --no-cache-dir install Pillow
RUN pip3 --no-cache-dir install \
    numpy scipy sklearn scikit-image pandas matplotlib requests
RUN pip3 --no-cache-dir install Cython
RUN pip3 --no-cache-dir install jupyter
RUN mkdir /root/.jupyter
RUN echo "c.NotebookApp.ip = '*'" \
         "\nc.NotebookApp.open_browser = False" \
         "\nc.NotebookApp.token = ''" \
         > /root/.jupyter/jupyter_notebook_config.py
RUN apt-get install -y --no-install-recommends \
    libjpeg8-dev libtiff5-dev libjasper-dev libpng12-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libgtk2.0-dev \
    liblapacke-dev checkinstall

# CUDA
ENV CUDNN_VERSION 7.1.1.5
LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"
RUN add-apt-repository ppa:graphics-drivers/ppa
RUN add-apt-repository ppa:ubuntu-toolchain-r/test -y
RUN apt-get update
RUN apt-get install -y nvidia-384 nvidia-384-dev
RUN apt-get install -y g++ freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libglu1-mesa libglu1-mesa-dev
RUN apt install -y gcc-6
RUN apt install -y g++-6
RUN rm /usr/bin/gcc
RUN rm /usr/bin/g++
RUN ln -s /usr/bin/gcc-6 /usr/bin/gcc
RUN ln -s /usr/bin/g++-6 /usr/bin/g++
RUN wget 'https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda_9.0.176_384.81_linux-run' -O cuda-run
RUN sh cuda-run

RUN apt-get update && apt-get install -y --no-install-recommends \
            libcudnn7=$CUDNN_VERSION-1+cuda9.0 && \
    rm -rf /var/lib/apt/lists/*

# For CUDA profiling, TensorFlow requires CUPTI.
ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH


# OpenCV
RUN git clone -b 3.2.0 --depth 1 https://github.com/opencv/opencv.git /usr/local/src/opencv
# Compile
RUN cd /usr/local/src/opencv && mkdir build && cd build && \
    cmake -D CMAKE_INSTALL_PREFIX=/usr/local \
          -D BUILD_TESTS=OFF \
          -D BUILD_PERF_TESTS=OFF \
          -D PYTHON_DEFAULT_EXECUTABLE=$(which python3) \
          .. && \
    make -j"$(nproc)" && \
    make install

# Tensorflow 1.5 for python3.5
RUN pip3 --no-cache-dir install "https://github.com/mind/wheels/releases/download/tf1.5-gpu-nomkl/tensorflow-1.5.0-cp35-cp35m-linux_x86_64.whl"

# Keras
RUN pip3 install --no-cache-dir --upgrade h5py pydot_ng keras

# Coco
RUN pip3 install --no-cache-dir git+https://github.com/waleedka/coco.git#subdirectory=PythonAPI

RUN apt-get clean && \
    apt-get autoremove
# TensorBoard
EXPOSE 6006
# IPython
EXPOSE 8888

CMD ["/bin/bash"]
