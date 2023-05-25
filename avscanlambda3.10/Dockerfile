FROM --platform=linux/x86_64 public.ecr.aws/lambda/python:3.10

# Set up working directories
RUN mkdir -p /opt/app
RUN mkdir -p /opt/app/build
RUN mkdir -p /opt/app/bin/


# Copy in the lambda source
WORKDIR /opt/app
COPY ./*.py /opt/app/
COPY requirements.txt /opt/app/requirements.txt

# Install packages
# update security
RUN : \
    && yum -y update --security \
i    && yum clean all \
    && rm -rf /var/cache/yum \
    && :

# This had --no-cache-dir, tracing through multiple tickets led to a problem in wheel
# Install required packages
RUN : \
    && yum update -y \
    && yum install -y \
        cpio \
        python3 \
        python3-pip \
        yum-utils \
        zip \
        unzip \
        less \
        libtool-ltdl \
        binutils \
    && yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
    && /var/lang/bin/python3.10 -m pip install --upgrade pip \
    && pip install -r /opt/app/requirements.txt \
    && yum clean all \
    && rm -rf /var/cache/yum \
    && :

# Download libraries we need to run in lambda
WORKDIR /tmp
RUN yumdownloader -x \*i686 --archlist=x86_64 \
  clamav \
  clamav-lib \
  clamav-scanner-systemd \
  clamav-update \
  elfutils-libs \
  json-c \
  lz4 \
  pcre2 \
  systemd-libs \
  libprelude \
  gnutls \
  libtasn1 \
  lib64nettle \
  nettle \
  libtool-ltdl \
  binutils

RUN rpm2cpio clamav-0*.rpm | cpio -idmv
RUN rpm2cpio clamav-lib*.rpm | cpio -idmv
RUN rpm2cpio clamav-update*.rpm | cpio -idmv
RUN rpm2cpio clamd-0*.rpm | cpio -idmv
RUN rpm2cpio elfutils-libs*.rpm | cpio -idmv
RUN rpm2cpio json-c*.rpm | cpio -idmv
RUN rpm2cpio lz4*.rpm | cpio -idmv
RUN rpm2cpio pcre*.rpm | cpio -idmv
RUN rpm2cpio systemd-libs*.rpm | cpio -idmv
RUN rpm2cpio gnutls* | cpio -idmv
RUN rpm2cpio nettle* | cpio -idmv
RUN rpm2cpio binutils* | cpio -idmv
RUN rpm2cpio lib* | cpio -idmv
RUN rpm2cpio *.rpm | cpio -idmv
RUN rpm2cpio libtasn1* | cpio -idmv
RUN rpm2cpio libtool-ltdl*.rpm | cpio -idmv
RUN yumdownloader -x \*i686 --archlist=x86_64 libxml2
RUN rpm2cpio libxml2*.rpm | cpio -idmv

RUN yumdownloader -x \*i686 --archlist=x86_64 clamav
RUN rpm2cpio clamav-0*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 clamav-lib
RUN rpm2cpio clamav-lib*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 clamav-update
RUN rpm2cpio clamav-update*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 json-c
RUN rpm2cpio json-c*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 pcre2
RUN rpm2cpio pcre*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libtool-ltdl
RUN rpm2cpio libtool-ltdl*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libxml2
RUN rpm2cpio libxml2*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 bzip2-libs
RUN rpm2cpio bzip2-libs*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 xz-libs
RUN rpm2cpio xz-libs*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libprelude
RUN rpm2cpio libprelude*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 gnutls
RUN rpm2cpio gnutls*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 nettle
RUN rpm2cpio nettle*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libcurl
RUN rpm2cpio libcurl*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libnghttp2
RUN rpm2cpio libnghttp2*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libidn2
RUN rpm2cpio libidn2*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libssh2
RUN rpm2cpio libssh2*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 openldap
RUN rpm2cpio openldap*.rpm | cpio -vimd

RUN yumdownloader -x \*i686 --archlist=x86_64 libunistring
RUN rpm2cpio libunistring*.rpm | cpio -vimd

RUN ls /usr/lib64

RUN cp -f /usr/lib64/libsasl2.so.3 /usr/lib64/libssl3.so /usr/lib64/libn*.so /usr/lib64/libn*.so.* /usr/lib64/libs*.so /usr/lib64/libs*.so.* /usr/lib64/libc*.so /usr/lib64/libc*.so.* /usr/lib64/ld* /tmp/usr/lib64/

# Copy over the binaries and libraries
RUN cp -r /tmp/usr/bin/clamscan /tmp/usr/bin/freshclam /tmp/usr/lib64/* /opt/app/bin/

# Fix the freshclam.conf settings
RUN echo "DatabaseMirror database.clamav.net" > /opt/app/bin/freshclam.conf
RUN echo "CompressLocalDatabase yes" >> /opt/app/bin/freshclam.conf

ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib:/var/lang/lib
# Create the zip file
WORKDIR /opt/app
RUN zip -r9 --exclude="*test*" /opt/app/build/lambda.zip *.py bin

WORKDIR /var/lang/lib/python3.10/site-packages
RUN zip -r9 /opt/app/build/lambda.zip *

WORKDIR /opt/app
