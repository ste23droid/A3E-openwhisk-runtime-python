#FROM openwhisk/dockerskeleton:latest
FROM ste23/whisk-python3action-alpine-tensorflow-numpy-pillow:prod

RUN echo -e '@edgunity http://nl.alpinelinux.org/alpine/edge/community\n\
@edge http://nl.alpinelinux.org/alpine/edge/main\n\
@testing http://nl.alpinelinux.org/alpine/edge/testing\n\
@community http://dl-cdn.alpinelinux.org/alpine/edge/community'\
  >> /etc/apk/repositories

ENV CC /usr/bin/clang
ENV CXX /usr/bin/clang++
ENV OPENCV_VERSION 3.4.2

RUN apk update \
    && apk upgrade \
    && apk add --update --no-cache \
      build-base \
      openblas-dev \
      unzip \
      wget \
      cmake \

      #Intel® TBB, a widely used C++ template library for task parallelism'
      libtbb@testing  \
      libtbb-dev@testing   \

      # Wrapper for libjpeg-turbo
      libjpeg  \

      # accelerated baseline JPEG compression and decompression library
      libjpeg-turbo-dev \

      # Portable Network Graphics library
      libpng-dev \

      # A software-based implementation of the codec specified in the emerging JPEG-2000 Part-1 standard (development files)
      jasper-dev \

      # Provides support for the Tag Image File Format or TIFF (development files)
      tiff-dev \

      # Libraries for working with WebP images (development files)
      libwebp-dev \

      # A C language family front-end for LLVM (development files)
      clang-dev \

      linux-headers \

      && pip install numpy \

      && mkdir /opt \
      && cd /opt \
      && wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip \
      && unzip ${OPENCV_VERSION}.zip \
      && rm -rf ${OPENCV_VERSION}.zip \
      && cd /opt/opencv-${OPENCV_VERSION} \
      && mkdir build \
      && cd build \
      && cmake -D CMAKE_BUILD_TYPE=RELEASE \
                -D BUILD_PYTHON_SUPPORT=ON \
                -D BUILD_EXAMPLES=OFF \
                -D CMAKE_INSTALL_PREFIX=/usr/local \
                -D WITH_FFMPEG=OFF \
                -D WITH_TBB=ON \
                -D WITH_IPP=OFF	\
                -D BUILD_TESTS=OFF \
                -D BUILD_PERF_TESTS=OFF \
                -D INSTALL_PYTHON_EXAMPLES=OFF \
                -D INSTALL_C_EXAMPLES=OFF \
                -D WITH_OPENEXR=OFF .. \
      && make VERBOSE=1  \
      && make -j$(nproc) \
      && make install  \
      && rm -rf /opt/opencv-${OPENCV_VERSION} \
      && rm -rf /var/cache/apk/* \
      && rm -rf /tmp/*
