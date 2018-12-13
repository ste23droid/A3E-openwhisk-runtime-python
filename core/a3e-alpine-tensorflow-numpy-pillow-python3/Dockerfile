FROM openwhisk/dockerskeleton:latest

# RUN which pip
# /usr/local/bin/pip
# RUN which pip3
# /usr/local/bin/pip3

# RUN ls -a
# RUN cd /actionProxy && ls
# RUN cd ..
# RUN cd /action && ls

RUN apk add --no-cache \
        bzip2-dev \
        gcc \
        libc-dev \
        libxslt-dev \
        libxml2-dev \
        libffi-dev \
        linux-headers \
        openssl-dev \
        # for Pillow support
        jpeg-dev

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

CMD ["/bin/bash", "-c", "cd pythonAction && python -u pythonrunner.py"]

# Based on https://github.com/better/alpine-tensorflow/blob/master/Dockerfile

ENV JAVA_HOME /usr/lib/jvm/java-1.8-openjdk
# resources for bazel when building tensorflow: RAM, CPU, IO
ENV LOCAL_RESOURCES 3048,1.0,1.0
ENV TENSORFLOW_VERSION 1.10.1
ENV BAZEL_VERSION 0.17.2
RUN apk add --no-cache py3-numpy py3-numpy-f2py freetype libpng libjpeg-turbo imagemagick graphviz git libstdc++
RUN apk add --no-cache --virtual=.build-deps \
        bash \
        cmake \
        curl \
        freetype-dev \
        gcc \
        g++ \
        libjpeg-turbo-dev \
        libpng-dev \
        linux-headers \
        build-base \
        make \
        musl-dev \
        openblas-dev \
        openjdk8 \
        patch \
        perl \
        python3-dev \
        py-numpy-dev \
        rsync \
        sed \
        swig \
        zip

# install wheel (if needed)
RUN pip3 install --no-cache-dir wheel numpy

RUN which pip
# /usr/local/bin/pip
RUN which pip3
# /usr/local/bin/pip3
RUN pip --version
# pip 18.1 from /usr/local/lib/python3.6/site-packages/pip (python 3.6)
RUN pip3 --version
# pip 18.1 from /usr/local/lib/python3.6/site-packages/pip (python 3.6)
RUN which python
# /usr/local/bin/python
RUN which python3
# /usr/local/bin/python3
RUN python --version
# Python 3.6.7
RUN python3 --version
# Python 3.6.7

# Bazel download
RUN cd /tmp \
    && curl -SLO https://github.com/bazelbuild/bazel/releases/download/${BAZEL_VERSION}/bazel-${BAZEL_VERSION}-dist.zip \
    && mkdir bazel-${BAZEL_VERSION} \
    && unzip -qd bazel-${BAZEL_VERSION} bazel-${BAZEL_VERSION}-dist.zip

# Bazel install
RUN cd /tmp/bazel-${BAZEL_VERSION} \
    && sed -i -e '/"-std=c++0x"/{h;s//"-fpermissive"/;x;G}' tools/cpp/cc_configure.bzl \
    && sed -i -e '/#endif  \/\/ COMPILER_MSVC/{h;s//#else/;G;s//#include <sys\/stat.h>/;G;}' third_party/ijar/common.h \
    && sed -i -e 's/-classpath/-J-Xmx8192m -J-Xms128m -classpath/g' scripts/bootstrap/compile.sh \
    && bash compile.sh \
    && cp -p output/bazel /usr/bin/

# Download Tensorflow
RUN cd /tmp \
    && curl -SL https://github.com/tensorflow/tensorflow/archive/v${TENSORFLOW_VERSION}.tar.gz \
        | tar xzf -

# Build Tensorflow
# See https://github.com/tensorflow/tensorflow/issues/348
RUN cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && : musl-libc does not have "secure_getenv" function, also patch sources \
    && sed -i -e '/JEMALLOC_HAVE_SECURE_GETENV/d' third_party/jemalloc.BUILD \
    && sed -i -e '/define TF_GENERATE_BACKTRACE/d' tensorflow/core/platform/default/stacktrace.h \
    && sed -i -e '/define TF_GENERATE_STACKTRACE/d' tensorflow/core/platform/stacktrace_handler.cc \
    && : the types below are fixed in tensorflow 1.11 \
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
        bash configure
RUN cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && bazel build -c opt --local_resources ${LOCAL_RESOURCES} --jobs 1 //tensorflow/tools/pip_package:build_pip_package
RUN cd /tmp/tensorflow-${TENSORFLOW_VERSION} \
    && ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
RUN cp /tmp/tensorflow_pkg/tensorflow-${TENSORFLOW_VERSION}-cp36-cp36m-linux_x86_64.whl /root

# Make sure it's built properly
RUN pip3 install --no-cache-dir /root/tensorflow-${TENSORFLOW_VERSION}-cp36-cp36m-linux_x86_64.whl \
    && python3 -c 'import tensorflow'

# cleanup temp files
RUN apk del .build-deps \
    && rm -f /usr/bin/bazel \
    && rm -rf /tmp/*
RUN rm -rf /root/.cache
