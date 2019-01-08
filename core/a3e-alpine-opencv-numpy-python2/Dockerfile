FROM python:2-alpine3.6

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
    && make \
    && make install  \
    && rm -rf /opt/opencv-${OPENCV_VERSION} \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*

# here starts the OpenWhiskPython2Action dockerfile
RUN apk add --no-cache \
        bash \
        bzip2-dev \
        gcc \
        libc-dev \
        libxslt-dev \
        libxml2-dev \
        libffi-dev \
        linux-headers \
        openssl-dev \
        python-dev

# Install common modules for python
RUN pip install --no-cache-dir --upgrade pip setuptools six \
 && pip install --no-cache-dir \
        gevent==1.3.6 \
        flask==1.0.2 \
        beautifulsoup4==4.6.3 \
        httplib2==0.11.3 \
        kafka_python==1.4.3 \
        lxml==4.2.5 \
        python-dateutil==2.7.3 \
        requests==2.19.1 \
        scrapy==1.5.1 \
        simplejson==3.16.0 \
        virtualenv==16.0.0 \
        twisted==18.7.0

ENV FLASK_PROXY_PORT 8080

# Add the action proxy
ADD https://raw.githubusercontent.com/apache/incubator-openwhisk-runtime-docker/master/core/actionProxy/actionproxy.py /actionProxy/actionproxy.py

ADD pythonrunner.py /pythonAction/
RUN mkdir -p /action

CMD ["/bin/bash", "-c", "cd pythonAction && python -u pythonrunner.py"]
