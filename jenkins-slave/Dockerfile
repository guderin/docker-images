FROM ubuntu:bionic

LABEL maintainer="RB-Devops <RB-devops@harman.com>"
LABEL description="Jenkins ssh slave"

ENV MAVEN_VERSION=3.6.0 \
    MAVEN_HOME=/usr/share/maven

# In case you need proxy
#RUN echo 'Acquire::http::Proxy "http://127.0.0.1:8080";' >> /etc/apt/apt.conf

# Set timezone
ENV TZ 'Asia/Jerusalem'
RUN echo $TZ > /etc/timezone \ 
    && apt-get update && apt-get install -y tzdata \
    && rm /etc/localtime \ 
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \ 
    && dpkg-reconfigure -f noninteractive tzdata \ 
    && apt-get clean

# Add locales after locale-gen as needed
# Upgrade packages on image
# Preparations for sshd

RUN apt-get -q update && apt-get install -y locales curl python-pip build-essential libssl-dev doxygen git vim libc6-dev-i386 ntpdate bash-completion \
    && locale-gen en_US.UTF-8 \
    && DEBIAN_FRONTEND="noninteractive" apt-get -q upgrade -y -o Dpkg::Options::="--force-confnew" --no-install-recommends \
    && DEBIAN_FRONTEND="noninteractive" apt-get -q install -y -o Dpkg::Options::="--force-confnew" --no-install-recommends openssh-server \
    && apt-get -q autoremove \
    && apt-get -q clean -y && rm -rf /var/lib/apt/lists/* && rm -f /var/cache/apt/*.bin \
    && sed -i 's|session    required     pam_loginuid.so|session    optional     pam_loginuid.so|g' /etc/pam.d/sshd \
    && mkdir -p /var/run/sshd

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV container docker

# Install JDK 8 (latest edition)
RUN apt-get -q update \
    && DEBIAN_FRONTEND="noninteractive" apt-get -q install -y -o Dpkg::Options::="--force-confnew" --no-install-recommends software-properties-common \
    && add-apt-repository -y ppa:openjdk-r/ppa \
    && apt-get -q update \
    && DEBIAN_FRONTEND="noninteractive" apt-get -q install -y -o Dpkg::Options::="--force-confnew" --no-install-recommends openjdk-8-jre-headless \
    && pip install Gitpython \
    #
    # clean all cache to clean space
    #
    && apt-get purge -y unzip \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /var/cache/apt/*.bin \
    && apt-get -q clean -y \
    && apt-get -y autoremove \
    #
    # Set user jenkins to the image
    #
    && useradd -m -d /home/jenkins -s /bin/bash jenkins \
    && echo "jenkins:jenkins" | chpasswd \
    #
    # install maven
    #
    && curl -fsSL https://www-us.apache.org/dist/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz | tar xzf - -C /usr/share \
    && mv /usr/share/apache-maven-$MAVEN_VERSION /usr/share/maven \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

# Standard SSH port
EXPOSE 22/tcp

# Default command
CMD ["/usr/sbin/sshd", "-D"]
