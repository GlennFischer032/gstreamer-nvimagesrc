# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM ubuntu:jammy

RUN echo "APT::Install-Recommends \"0\";" >> /etc/apt/apt.conf
RUN echo "APT::Install-Suggests \"0\";" >> /etc/apt/apt.conf

RUN addgroup --gid 1000 group &&  adduser --gid 1000 --uid 1000 --disabled-password --gecos user user

# Install essentials
RUN \
    apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        curl \
        build-essential \
        ca-certificates \
        git \
        vim wget && apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /opt

RUN chown 1000:1000 /opt

ARG GSTREAMER_VERSION=1.20.3
ARG GST_PLUGINS_BASE_VERSION=1.20.3
ARG GST_PLUGINS_GOOD_VERSION=1.20.3
#ARG GST_PLUGINS_BAD_VERSION=28bd479ea2996752d25feded96d41f9a4e468eeb
ARG GST_PLUGINS_BAD_VERSION=1.20.3
ARG GST_PLUGINS_UGLY_VERSION=1.20.3
ARG GST_PYTHON_VERSION=1.20.3

USER 1000
# cloner repo for each gstreamer module
RUN git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git && cd gstreamer && git checkout ${GSTREAMER_VERSION}
#RUN git clone https://gitlab.freedesktop.org/gstreamer/gst-plugins-base.git && cd gst-plugins-base && git checkout ${GST_PLUGINS_BASE_VERSION}
#RUN git clone https://gitlab.freedesktop.org/gstreamer/gst-plugins-good && cd gst-plugins-good && git checkout ${GST_PLUGINS_GOOD_VERSION}
#RUN git clone https://gitlab.freedesktop.org/gstreamer/gst-plugins-bad && cd gst-plugins-bad && git checkout ${GST_PLUGINS_BAD_VERSION}
#RUN git clone https://gitlab.freedesktop.org/gstreamer/gst-plugins-ugly && cd gst-plugins-ugly && git checkout ${GST_PLUGINS_UGLY_VERSION}
#RUN git clone https://gitlab.freedesktop.org/gstreamer/gst-python && cd gst-python && git checkout ${GST_PYTHON_VERSION}

USER 0

WORKDIR /opt

# Install base build deps
RUN \
    apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        autopoint \
        autoconf \
        automake \
        autotools-dev \
        libtool \
        gettext \
        bison \
        flex \
        gtk-doc-tools \
        libtool-bin \
        libgtk2.0-dev \
        libgl1-mesa-dev \
        libopus-dev \
        libpulse-dev \
        libgirepository1.0-dev \
        gobjc++ \
        gobjc \
        libx11-xcb-dev \
        python3-dev \
        python3-pip python-gi-dev ninja-build \
        valac libgtk-3-dev libwebrtc-audio-processing-dev \
        libssl-dev \
        libsrtp2-dev \
        libx264-dev \
        va-driver-all:amd64 \
        vainfo \
        libva-dev:amd64 \
        libva-x11-2:amd64 \
        libva-glx2:amd64 \
        libxml2 \
        && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install meson build deps
RUN \
    pip3 install meson

USER 1000

# Build gstreamer
# Install usrsctp from source
ARG USRSCTP_VERSION=0.9.5.0
RUN \
    git clone https://github.com/sctplab/usrsctp.git && \
    cd usrsctp && git checkout ${USRSCTP_VERSION} && \
    ./bootstrap && ./configure --prefix=/usr && \
        make 

USER 0

RUN cd usrsctp && make install

USER 1000

# Install libnice from source
#ARG LIBNICE_VERSION=0.1.18
ARG LIBNICE_VERSION=master
RUN \
    git clone https://gitlab.freedesktop.org/libnice/libnice.git && \
    cd libnice && git checkout ${LIBNICE_VERSION} && \
    meson build --prefix=/usr  && ninja -C build -j4

USER 0

RUN cd libnice && ninja -C build install

USER 1000

COPY patch/gstwebrtbin.patch /opt/gstreamer

RUN \
    cd /opt/gstreamer && \
    patch -sp1 < gstwebrtbin.patch && rm -f gstwebrtbin.patch && \
    meson build --prefix=/usr && ninja -C build -j4

USER 0

RUN cd /opt/gstreamer && ninja -C build install

#RUN sed -i 's/^Libs:/Libs: -lpython3.8/' /usr/lib/x86_64-linux-gnu/pkgconfig/python-3.8.pc

COPY --chown=1000:1000 nvimage /opt/nvimage

RUN apt-get update && apt-get -y install libnvidia-decode-510 libnvidia-fbc1-510 libnvidia-encode-510 && cd /opt/nvimage && ./build.sh && cp libgstnvimagesrc.so /usr/lib/x86_64-linux-gnu/gstreamer-1.0/ && apt-get remove --purge -y libnvidia-decode-510 libnvidia-fbc1-510 libnvidia-encode-510 libnvidia-compute-510 && apt-get clean &&  rm -rf /var/lib/apt/lists/*

COPY vulkan /usr/share/vulkan
COPY libgstnvimagesrchevc.so /usr/lib/x86_64-linux-gnu/gstreamer-1.0/
COPY libgstvaapi.so /usr/lib/x86_64-linux-gnu/gstreamer-1.0/
RUN chown -R 1000:1000 /usr/lib/x86_64-linux-gnu/gstreamer-1.0/
COPY NvFBCToGLEnc /usr/local/bin/
RUN apt-get update && apt-get -y install gdb 
