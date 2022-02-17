FROM node:14-alpine

# we need newer git for sparse checkout feature
RUN echo 'http://dl-cdn.alpinelinux.org/alpine/v3.13/main' >> /etc/apk/repositories
RUN echo 'http://dl-cdn.alpinelinux.org/alpine/v3.13/community' >> /etc/apk/repositories
RUN apk update && apk upgrade
RUN apk add git==2.30.2-r0

# download dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget -q https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && mkdir /app \
    && adduser -D -u 2002 -g webhooks webhooks \
    && chown webhooks:webhooks /app

USER webhooks


ENV TAG v2.4.0

RUN cd ~ \
    && git clone --filter=blob:none --no-checkout https://github.com/bigbluebutton/bigbluebutton.git \
    && cd bigbluebutton \
    && git sparse-checkout set bbb-webhooks \
    && git checkout tags/$TAG -b docker-build

RUN cd ~/bigbluebutton/bbb-webhooks \
    && mv * /app/ \
    && cd /app && npm install --production

COPY entrypoint.sh /entrypoint.sh
COPY config.yml /app/config/default.yml.tmpl

ENTRYPOINT /entrypoint.sh


