FROM ubuntu:wily

MAINTAINER Sebastian Gutsche <sebastian.gutsche@gmail.com>

RUN    apt-get update -qq \
    && apt-get install -y wget build-essential m4 dpkg-dev sudo python libssl-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN    adduser --quiet --shell /bin/bash --gecos "Sage user,101,," --disabled-password sage \
    && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers \
    && adduser sage sudo \
    && chown -R sage:sage /home/sage/

USER sage

# see https://github.com/docker/docker/issues/9547 for the sync

ADD install_sage.sh /tmp/install_sage.sh
RUN    sudo chown sage:sage /tmp/install_sage.sh \
    && chmod +x /tmp/install_sage.sh \
    && sync \
    && /tmp/install_sage.sh

ADD postinstall_sage.sh /tmp/postinstall_sage.sh
RUN    sudo chown sage:sage /tmp/postinstall_sage.sh \
    && chmod +x /tmp/postinstall_sage.sh \
    && sync \
    && /tmp/postinstall_sage.sh

ENV HOME /home/sage
WORKDIR /home/sage

EXPOSE 8080

CMD [ "sage" ]
