FROM ubuntu:18.04
MAINTAINER quriouspixel

ENV QTVER=5.14.2
ENV QTVERMIN=514
ENV LLVMVER=10
ENV GCCVER=10

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
    add-apt-repository -y ppa:beineri/opt-qt-${QTVER}-bionic && \
    apt-mark hold libsdl2-2.0-0 && \
    DEBIAN_FRONTEND=noninteractive apt-get update && apt-get -y full-upgrade && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    aria2 \
    build-essential \
    dpkg \
    fuse \
    #glslang-dev \
    #glslang-tools \
    $GCC_BINARY $GXX_BINARY \
    libboost-all-dev \
    liblz4-dev \
    liblzma-dev \
    libssl-dev \
    libopus-dev \
    libpthread-stubs0-dev \
    libzip-dev \
    libzstd-dev \
    zlib1g-dev \
    libasound2-dev \
    libpulse-dev \
    pulseaudio \
    p7zip \
    p7zip-full \
    python \
    python-pip \
    python-setuptools \
    qt514-meta-full \
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
    pip install conan 
    
RUN \
	update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-${GCCVER} 10 && \
	update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-${GCCVER} 10 && \
	ls -al /opt/qt514/ && \
	ls -al /opt/qt514/bin/ && \
	find . -name "qt514-env.sh" -exec sh qt514-env.sh {} \; && \
	. /opt/qt514/bin/qt514-env.sh && \
	rm -rf /opt/qt514/examples /opt/qt514/doc && \
	gcc --version && \
	g++ --version 
    
ENV CMAKEVER=3.19.1
RUN \
	cd /tmp && \
	curl -sLO https://cmake.org/files/v${CMAKEVER%.*}/cmake-${CMAKEVER}-Linux-x86_64.sh && \
	sh cmake-${CMAKEVER}-Linux-x86_64.sh --prefix=/usr --skip-license && \
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

ENV SDL2VER=2.0.12
RUN \
	cd /tmp && \
	curl -sLO https://libsdl.org/release/SDL2-${SDL2VER}.tar.gz && \
	tar -xzf SDL2-${SDL2VER}.tar.gz && \
	cd SDL2-${SDL2VER} && \
	./configure --prefix=/usr && \
	make && make install && \
	rm ../SDL2-${SDL2VER}.tar.gz 

ENV VULKANVER=1.2.162
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
	
RUN apt-get clean autoclean && apt-get autoremove --yes && rm -rf /var/lib/apt /var/lib/cache /var/lib/log

RUN mkdir -p /yuzu
RUN chown yuzu /yuzu	
USER 1027
WORKDIR /yuzu

RUN conan install catch2/2.13.0@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install fmt/7.1.2@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install mbedtls/2.23.0@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install nlohmann_json/3.8.0@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install boost/1.73.0@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install lz4/1.9.2@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install zlib/1.2.11@ -s compiler.libcxx=libstdc++11 --build=missing && \
    conan install zstd/1.4.5@ -s compiler.libcxx=libstdc++11 --build=missing

