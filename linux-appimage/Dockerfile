FROM ubuntu:18.04
MAINTAINER quriouspixel
WORKDIR /yuzu

ENV QTVER=5.15.2
ENV QTVERMIN=515
ENV LLVMVER=10
ENV GCCVER=11
ENV UBUNTU_VER=bionic

ENV CLANG_BINARY=clang-${LLVMVER}
ENV CLANGXX_BINARY=clang++-${LLVMVER}
ENV LLD_BINARY=lld-${LLVMVER}
ENV GCC_BINARY=gcc-${GCCVER}
ENV GXX_BINARY=g++-${GCCVER}

# Create a user account yuzu (UID 1027) that the container will run as
RUN useradd -m -u 1027 -s /bin/bash yuzu && \
    apt-get update -y && \
    apt-get install -y curl software-properties-common apt-transport-https && \
    add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
    add-apt-repository -y ppa:beineri/opt-qt-${QTVER}-${UBUNTU_VER} && \
    add-apt-repository -y ppa:jonathonf/ffmpeg-4 && \
    apt-mark hold libsdl2-2.0-0 && \
    DEBIAN_FRONTEND=noninteractive apt-get update && apt-get -y full-upgrade && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    aria2 \
    autoconf \
    automake \
    autotools-dev \
    build-essential \
    dpkg \
    fuse \
    $GCC_BINARY $GXX_BINARY \
    libboost-all-dev \
    libtool \
    liblz4-dev \
    liblzma-dev \
    libssl-dev \
    libopus-dev \
    libpthread-stubs0-dev \
    libzstd-dev \
    zlib1g-dev \
    libasound2-dev \
    libpulse-dev \
    pulseaudio \
    p7zip \
    p7zip-full \
    python3 \
    python3-pip \
    python3-setuptools \
    qt${QTVERMIN}base \
    qt${QTVERMIN}declarative \
    qt${QTVERMIN}xmlpatterns \
    qt${QTVERMIN}svg \
    qt${QTVERMIN}tools \
    qt${QTVERMIN}wayland \
    qt${QTVERMIN}webengine \
    libxi-dev \
    libavcodec-dev \
    libavutil-dev \
    libswscale-dev \
    libudev-dev \
    libusb-1.0-0-dev \
    libevdev-dev \
    libc6-dev \
    libavformat-dev \
    libavdevice-dev \
    libfmt-dev \
    libwayland-dev \
    libxrandr-dev \
    libglu1-mesa-dev \
    x11-utils \
    zenity \
    wget \
    curl \
    git \
    ccache \
    ninja-build && \
    pip3 install conan 
    
RUN \
	update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-${GCCVER} 10 && \
	update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-${GCCVER} 10 && \
	ls -al /opt/qt${QTVERMIN}/ && \
	ls -al /opt/qt${QTVERMIN}/bin/ && \
	find . -name "qt${QTVERMIN}-env.sh" -exec sh qt${QTVERMIN}-env.sh {} \; && \
	. /opt/qt${QTVERMIN}/bin/qt${QTVERMIN}-env.sh && \
	rm -rf /opt/qt${QTVERMIN}/examples /opt/qt${QTVERMIN}/doc && \
	gcc --version && \
	g++ --version 
    
ENV CMAKEVER=3.20.3
RUN \
	cd /tmp && \
	curl -sLO https://cmake.org/files/v${CMAKEVER%.*}/cmake-${CMAKEVER}-linux-x86_64.sh && \
	sh cmake-${CMAKEVER}-linux-x86_64.sh --prefix=/usr --skip-license && \
	rm ./cmake*.sh && \
	cmake --version

ENV LIBZIPVER=1.7.3
RUN \
	cd /tmp && \
	curl -sLO https://libzip.org/download/libzip-${LIBZIPVER}.tar.gz && \
	tar -xzf libzip-${LIBZIPVER}.tar.gz && \
	cd libzip-${LIBZIPVER} && \
	mkdir build && cd build && \
	cmake .. -DCMAKE_INSTALL_PREFIX=/usr && \
	make && make install && \
	rm ../../libzip-${LIBZIPVER}.tar.gz 

ENV SDL2VER=2.0.14
RUN \
	cd /tmp && \
	curl -sLO https://libsdl.org/release/SDL2-${SDL2VER}.tar.gz && \
	tar -xzf SDL2-${SDL2VER}.tar.gz && \
	cd SDL2-${SDL2VER} && \
	./configure --prefix=/usr && \
	make && make install && \
	rm ../SDL2-${SDL2VER}.tar.gz 

ENV VULKANVER=1.2.184
RUN \
	cd /tmp && \
	curl -sL -o Vulkan-Headers.tar.gz https://github.com/KhronosGroup/Vulkan-Headers/archive/v${VULKANVER}.tar.gz && \
	tar -xf Vulkan-Headers*.tar.gz && \
	cd Vulkan-Headers*/ && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$GCC_BINARY -DCMAKE_CXX_COMPILER=$GXX_BINARY -DCMAKE_INSTALL_PREFIX=/usr && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./Vulkan-Headers*
RUN \
	cd /tmp && \
	curl -sL -o Vulkan-Loader.tar.gz https://github.com/KhronosGroup/Vulkan-Loader/archive/v${VULKANVER}.tar.gz && \
	tar -xf Vulkan-Loader.tar.gz && \
	cd Vulkan-Loader*/ && \
	mkdir build && cd build && \
	cmake ../ -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_COMPILER=$GCC_BINARY -DCMAKE_CXX_COMPILER=$GXX_BINARY \
	-DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=/usr/lib -DCMAKE_INSTALL_SYSCONFDIR=/etc -DCMAKE_INSTALL_DATADIR=/share && \
	ninja && \
	ninja install && \
	cd ../.. && \
	rm ./*.tar.gz && \
	rm -r ./Vulkan-Loader* 
	
RUN \
	cd /tmp && \
	git clone https://github.com/KhronosGroup/glslang.git && \
	cd glslang && \
	mkdir -p build && cd build && \
	cmake .. -GNinja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="/usr" && \
	ninja && ninja install && \
	glslangValidator --version && \
	cd /tmp

RUN \
        cd /tmp && \
        wget https://github.com/yuzu-emu/ext-linux-bin/raw/main/boost/boost_1_75_0.tar.xz &&\
        tar xvf boost_1_75_0.tar.xz && \
        chown -R root:root boost_1_75_0/ && \
        cp -rv boost_1_75_0/include boost_1_75_0/lib /usr && \
        rm -rf boost*
	
ENV LIBUSBVER=1.0.24
RUN \
	cd /tmp && \
	curl -sLO https://github.com/libusb/libusb/releases/download/v${LIBUSBVER}/libusb-${LIBUSBVER}.tar.bz2 && \
	tar -xvf libusb-${LIBUSBVER}.tar.bz2 && \
	cd libusb-${LIBUSBVER} && \
	./configure && \
	make && make install && \
	rm ../libusb-${LIBUSBVER}.tar.bz2

ENV HIDAPIVER=0.10.1
RUN \
	cd /tmp && \
	curl -sLO https://github.com/libusb/hidapi/archive/refs/tags/hidapi-${HIDAPIVER}.tar.gz && \
	tar -xvf hidapi-${HIDAPIVER}.tar.gz && \
	cd hidapi-hidapi-${HIDAPIVER} && \
	./bootstrap && \
	./configure --prefix=/usr && \
	make && make install && \
	rm ../hidapi-${HIDAPIVER}.tar.gz
	
RUN apt-get clean autoclean && apt-get autoremove --yes && rm -rf /var/lib/apt /var/lib/cache /var/lib/log

# Setup paths for Qt binaries
ENV LD_LIBRARY_PATH=/opt/qt${QTVERMIN}/lib:${LD_LIBRARY_PATH}
ENV PATH=/opt/qt${QTVERMIN}/bin:${PATH}

# Need to patch Qt for compatibility with GCC 11
# See https://codereview.qt-project.org/c/qt/qtbase/+/339417
RUN \
	sed -i 's/ThreadEngineStarter<void>(ThreadEngine<void> \*_threadEngine)/ThreadEngineStarter(ThreadEngine<void> \*_threadEngine)/' \
	/opt/qt515/include/QtConcurrent/qtconcurrentthreadengine.h
