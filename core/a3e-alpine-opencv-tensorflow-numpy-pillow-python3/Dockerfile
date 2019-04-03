#FROM openwhisk/dockerskeleton:latest
FROM ste23/whisk-python3action-alpine-tensorflow-numpy-pillow

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
RUN pip install \
    lxml==4.2.5 \
    beautifulsoup4==4.6.3 \
    httplib2==0.11.3 \
    kafka_python==1.4.3 \
    python-dateutil==2.7.3 \
    requests==2.19.1 \
    scrapy==1.5.1 \
    simplejson==3.16.0 \
    virtualenv==16.0.0 \
    twisted==18.7.0 \
    # adding Pillow support
    Pillow==5.3.0

ENV FLASK_PROXY_PORT 8080

RUN mkdir -p /pythonAction
ADD pythonrunner.py /pythonAction/
RUN rm -rf /action
RUN mkdir /action

# Based on https://github.com/better/alpine-tensorflow/blob/master/Dockerfile

ENV JAVA_HOME /usr/lib/jvm/java-1.8-openjdk
# resources for bazel when building tensorflow: RAM, CPU, IO
ENV LOCAL_RESOURCES 3048,1.0,1.0
ENV TENSORFLOW_VERSION 1.10.1
ENV BAZEL_VERSION 0.17.2

RUN apk add --no-cache py3-numpy py3-numpy-f2py freetype libpng libjpeg-turbo git libstdc++ \
    && apk add --no-cache --virtual=.build-deps bash cmake curl freetype-dev gcc g++ libjpeg-turbo-dev libpng-dev linux-headers build-base make musl-dev openblas-dev openjdk8 patch perl python3-dev py-numpy-dev rsync sed swig zip \
    && pip3 install --no-cache-dir wheel numpy \
    && cd /tmp \
    && curl -SLO https://github.com/bazelbuild/bazel/releases/download/${BAZEL_VERSION}/bazel-${BAZEL_VERSION}-dist.zip \
    && mkdir bazel-${BAZEL_VERSION} \
    && unzip -qd bazel-${BAZEL_VERSION} bazel-${BAZEL_VERSION}-dist.zip \
    && cd bazel-${BAZEL_VERSION} \
    && sed -i -e '/"-std=c++0x"/{h;s//"-fpermissive"/;x;G}' tools/cpp/cc_configure.bzl \
    && sed -i -e '/#endif  \/\/ COMPILER_MSVC/{h;s//#else/;G;s//#include <sys\/stat.h>/;G;}' third_party/ijar/common.h \
    && sed -i -e 's/-classpath/-J-Xmx8192m -J-Xms128m -classpath/g' scripts/bootstrap/compile.sh \
    && bash compile.sh \
    && cp -p output/bazel /usr/bin/ \
    && cd /tmp \
    && wget -q -O - https://github.com/tensorflow/tensorflow/archive/v${TENSORFLOW_VERSION}.tar.gz | tar -xzf - -C /tmp \
    && cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && sed -i -e '/JEMALLOC_HAVE_SECURE_GETENV/d' third_party/jemalloc.BUILD \
    && sed -i -e '/define TF_GENERATE_BACKTRACE/d' tensorflow/core/platform/default/stacktrace.h \
    && sed -i -e '/define TF_GENERATE_STACKTRACE/d' tensorflow/core/platform/stacktrace_handler.cc \
    && sed -i -e 's/uint /uint32_t /g' tensorflow/contrib/lite/kernels/internal/spectrogram.cc \
    && PYTHON_BIN_PATH=/usr/local/bin/python \
        PYTHON_LIB_PATH=/usr/local/lib/python3.6/site-packages \
        CC_OPT_FLAGS="-march=native" \
        TF_NEED_JEMALLOC=1 \
        TF_NEED_GCP=0 \
        TF_NEED_HDFS=0 \
        TF_NEED_S3=0 \
        TF_ENABLE_XLA=0 \
        TF_NEED_GDR=0 \
        TF_NEED_VERBS=0 \
        TF_NEED_OPENCL=0 \
        TF_NEED_CUDA=0 \
        TF_NEED_MPI=0 \
        TF_NEED_KAFKA=0 \
        TF_NEED_AWS=0 \
        TF_NEED_OPENCL_SYCL=0 \
        TF_DOWNLOAD_CLANG=0 \
        TF_SET_ANDROID_WORKSPACE=0 \
        bash configure \
    && cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && bazel build -c opt --local_resources ${LOCAL_RESOURCES} --jobs 1 //tensorflow/tools/pip_package:build_pip_package \
    && cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg \
    && cp /tmp/tensorflow_pkg/tensorflow-${TENSORFLOW_VERSION}-cp36-cp36m-linux_x86_64.whl /root \
    && pip3 install --no-cache-dir /root/tensorflow-${TENSORFLOW_VERSION}-cp36-cp36m-linux_x86_64.whl \
    && apk del .build-deps \
    && rm -f /usr/bin/bazel \
    && rm -rf /tmp/*   \
    && rm -rf /root/.cache

# open whisk python action command
CMD ["/bin/bash", "-c", "cd pythonAction && python -u pythonrunner.py"]
