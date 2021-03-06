ARG JENKINS_VER=lts
ARG JENKINS_REGISTRY=jenkins/jenkins
FROM ${JENKINS_REGISTRY}:${JENKINS_VER}

# switch to root, let the entrypoint drop back to jenkins
USER root

# install prerequisite debian packages
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common \
     vim \
     wget \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# install gosu for a better su+exec command
ARG GOSU_VERSION=1.10
RUN dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
 && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
 && chmod +x /usr/local/bin/gosu \
 && gosu nobody true 

# install docker
ARG DOCKER_CLI_VERSION==5:18.09.0~3-0~debian-stretch
# ARG DOCKER_CLI_VERSION=
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - \
 && add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/debian \
     $(lsb_release -cs) \
     stable" \
 && apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    docker-ce-cli${DOCKER_CLI_VERSION} \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* \
 && groupadd -r docker \
 && usermod -aG docker jenkins

# install arduino
RUN apt-get update && \
    apt-get upgrade -y && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
    # Generic
    build-essential \
    curl \
    wget \
    git \
    # Python
    python \
    python-pip \
    picocom \
    gcc-avr \
    avr-libc \
    avrdude \
    arduino \
    arduino-mk \
    arduino-core && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN  wget "https://downloads.arduino.cc/arduino-cli/arduino-cli-latest-linux64.tar.bz2" \
 && tar -xf arduino-cli-latest-linux64.tar.bz2 \
 &&  mv arduino-cli-0.3.4-alpha.preview-linux64 /usr/local/bin/arduino-cli

RUN wget "https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz" \
  && tar -xf openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz  \
  && mv openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit/* /bin


RUN pip install ino

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

# entrypoint is used to update docker gid and revert back to jenkins user
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
HEALTHCHECK CMD curl -sSLf http://localhost:8080/login >/dev/null || exit 1

ARG BUILD_DATE
ARG VCS_REF
ARG IMAGE_PATCH_VER=0
LABEL \
    org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.docker.cmd="docker run -d -p 8080:8080 -v \"$$(pwd)/jenkins-home:/var/jenkins_home\" -v /var/run/docker.sock:/var/run/docker.sock docker-image" \
    org.label-schema.description="Jenkins with docker support, Jenkins ${JENKINS_VER}, Docker ${DOCKER_VER}" \
    org.label-schema.name="bmitch3020/jenkins-docker" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.version="${JENKINS_VER}-${IMAGE_PATCH_VER}"

