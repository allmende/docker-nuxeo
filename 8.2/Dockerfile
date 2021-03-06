# vim:set ft=dockerfile:
FROM       java:8-jre
MAINTAINER Nuxeo <packagers@nuxeo.com>

# Create Nuxeo user
ENV NUXEO_USER nuxeo
RUN useradd -m -d /home/$NUXEO_USER -s /bin/bash $NUXEO_USER

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true 

# Add needed convert tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    perl \
    locales \
    pwgen \
    imagemagick \
    ffmpeg2theora \
    ufraw \
    poppler-utils \
    libreoffice \
    libwpd-tools \
    exiftool \
    ghostscript \
 && rm -rf /var/lib/apt/lists/*

# Mitigate ImageTragick vulnerability
ADD imagemagick-policy.xml /etc/ImageMagick/policy.xml

ENV NUXEO_VERSION 8.2
ENV NUXEO_MD5 13cbc55be36823bec99482c651c1cbc0
ENV NUXEO_HOME /opt/nuxeo/server

# Add distribution
RUN curl -fsSL "http://cdn.nuxeo.com/nuxeo-${NUXEO_VERSION}/nuxeo-cap-${NUXEO_VERSION}-tomcat.zip" -o /tmp/nuxeo-distribution-tomcat.zip \
    && echo "$NUXEO_MD5 /tmp/nuxeo-distribution-tomcat.zip" | md5sum -c - \
    && mkdir -p /tmp/nuxeo-distribution $(dirname $NUXEO_HOME) \
    && unzip -q -d /tmp/nuxeo-distribution /tmp/nuxeo-distribution-tomcat.zip \
    && DISTDIR=$(/bin/ls /tmp/nuxeo-distribution | head -n 1) \
    && mv /tmp/nuxeo-distribution/$DISTDIR $NUXEO_HOME \
    && sed -i -e "s/^org.nuxeo.distribution.package.*/org.nuxeo.distribution.package=docker/" $NUXEO_HOME/templates/common/config/distribution.properties \
    && rm -rf /tmp/nuxeo-distribution* \
    && sed -i "s/LAUNCHER_DEBUG /LAUNCHER_DEBUG -Djvmcheck=nofail /" $NUXEO_HOME/bin/nuxeoctl \
    && chmod +x $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh

RUN mkdir /docker-entrypoint-initnuxeo.d
ENV PATH $NUXEO_HOME/bin:$PATH

WORKDIR $NUXEO_HOME
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 8080
EXPOSE 8787
CMD ["nuxeoctl","console"]
