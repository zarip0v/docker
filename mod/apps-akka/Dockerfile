FROM mozilla/sbt:8u212_1.2.8 AS builder

RUN apt-get update && apt-get install -y subversion

RUN echo "deb http://ftp.us.debian.org/debian testing main contrib non-free" >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y git \
    && apt-get clean all

# download bbb-common-message
ENV TAG v2.4.0
RUN git clone --filter=blob:none --no-checkout https://github.com/bigbluebutton/bigbluebutton.git \
    && cd bigbluebutton \
    && git sparse-checkout set bbb-common-message akka-bbb-apps \
    && git checkout tags/$TAG -b docker-build

RUN cd bigbluebutton/bbb-common-message \
    && ./deploy.sh

RUN cd bigbluebutton/akka-bbb-apps \
    && sbt universal:packageBin \
    && unzip target/universal/bbb-apps-akka-0.0.4.zip -d /

# ===================================================

FROM openjdk:8-jre-slim-bullseye

RUN apt update && apt-get install -y wget gosu

# install dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget -q https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

RUN groupadd -g 998 bigbluebutton \
    && useradd -m -u 998 -g bigbluebutton bigbluebutton

COPY --from=builder /bbb-apps-akka-0.0.4 /bbb-apps-akka
COPY bbb-apps-akka.conf /etc/bigbluebutton/bbb-apps-akka.conf.tmpl
COPY logback.xml /bbb-apps-akka/conf/logback.xml

WORKDIR /bbb-apps-akka
CMD dockerize \
    -template /etc/bigbluebutton/bbb-apps-akka.conf.tmpl:/etc/bigbluebutton/bbb-apps-akka.conf \
    gosu bigbluebutton /bbb-apps-akka/bin/bbb-apps-akka
