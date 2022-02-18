FROM debian:bullseye-slim

RUN echo "deb http://ftp.us.debian.org/debian testing main contrib non-free" >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y --allow-unauthenticated git \
    && apt-get clean all

# add freeswitch repo
RUN apt-get update --allow-unauthenticated && \
apt-get install -y --allow-unauthenticated --no-install-recommends gpgv2 gpgv1 gpgv curl wget ca-certificates gnupg gnupg2 lsb-release unzip && \
wget -O /usr/share/keyrings/freeswitch-archive-keyring.gpg https://files.freeswitch.org/repo/deb/debian-release/freeswitch-archive-keyring.gpg && \
echo 'deb [signed-by=/usr/share/keyrings/freeswitch-archive-keyring.gpg] http://files.freeswitch.org/repo/deb/debian-release/ bullseye main' > /etc/apt/sources.list.d/freeswitch.list && \
echo "deb http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list && \ 
echo "deb-src http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list 

# install dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget -q https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# install freeswitch
RUN apt-get update --allow-insecure-repositories && apt-get install -y --allow-unauthenticated \
        freeswitch \
        freeswitch-mod-commands \
        freeswitch-mod-conference \
        freeswitch-mod-console \
        freeswitch-mod-dialplan-xml \
        freeswitch-mod-dptools \
        freeswitch-mod-event-socket \
        freeswitch-mod-native-file \
        freeswitch-mod-opusfile \
        freeswitch-mod-opus \
        freeswitch-mod-sndfile \
        freeswitch-mod-sofia \
        freeswitch-sounds-en-us-callie \
        iptables


# -- get official bbb freeswitch config
# we use svn for retrieving the files since the repo is quite large,
# git sparse-checkout is not yet available with buster and there
# is no other sane way of downloading a single directory via git

ENV TAG v2.3.4
RUN git clone --filter=blob:none --no-checkout https://github.com/bigbluebutton/bigbluebutton.git \
    && cd bigbluebutton \
    && git sparse-checkout set bbb-voice-conference/config/freeswitch/conf \
    && git checkout tags/$TAG -b docker-build

# the current available freeswitch-mod-opusfile is broken,
# it can't write any .opus files. The fix provided in
# https://github.com/signalwire/freeswitch/pull/719/files
# is not sufficient as the module still comes without opus
# write support, so we rather switch to the binary built
# by bigbluebutton and add its dependencies
RUN wget -O /usr/lib/freeswitch/mod/mod_opusfile.so https://github.com/bbb-pkg/bbb-freeswitch-core/raw/43f3a47af1fcf5ea559e16bb28b900c925a7f2c3/opt/freeswitch/lib/freeswitch/mod/mod_opusfile.so \
    && wget -O /tmp/libopusenc0_0.2.1-1bbb1_amd64.deb https://launchpad.net/~bigbluebutton/+archive/ubuntu/support/+files/libopusenc0_0.2.1-1bbb1_amd64.deb \
    && dpkg -i /tmp/libopusenc0_0.2.1-1bbb1_amd64.deb \
    && rm /tmp/libopusenc0_0.2.1-1bbb1_amd64.deb
    
RUN ls bigbluebutton/bbb-voice-conference/config/freeswitch/conf/
RUN cp -R bigbluebutton/bbb-voice-conference/config/freeswitch/conf/* /etc/freeswitch
RUN ls /etc/freeswitch

# add modifications
COPY ./conf /etc/freeswitch/
RUN ls /etc/freeswitch

COPY ./entrypoint.sh /entrypoint.sh
ENTRYPOINT /entrypoint.sh
