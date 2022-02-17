
FROM ruby:2.5-slim-buster

RUN echo "deb http://ftp.us.debian.org/debian testing main contrib non-free" >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y git \
    && apt-get clean all

# install apt dependencies
RUN apt-get update && apt-get install -y \
    wget \
    rsync \
    build-essential \
    libsystemd-dev \
    python3 \
    python3-pyinotify \
    python3-lxml \
    python3-icu \
    ffmpeg \
    poppler-utils \
    imagemagick \
    supervisor \
    locales \
    locales-all

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# compile and install mkclean
RUN cd /tmp \
    && wget https://netcologne.dl.sourceforge.net/project/matroska/mkclean/mkclean-0.8.10.tar.bz2 \
    && tar -xf /tmp/mkclean-0.8.10.tar.bz2 \
    && cd /tmp/mkclean-0.8.10 \
    && sed -i 's/\r//g' ./mkclean/configure.compiled \
    && ./mkclean/configure.compiled \
    && make -C mkclean \
    && cp ./release/gcc_linux_x64/mkclean /usr/bin/mkclean \
    && rm -r /tmp/mkclean-*

# add dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget -q https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

RUN mkdir -p \
    /usr/local/bigbluebutton \
    /etc/bigbluebutton

ENV TAG v2.4.0

# clone only bbb-record-core (lib, scripts and Gemfile) and bbb-playback-presentation scripts
RUN git clone --filter=blob:none --no-checkout https://github.com/bigbluebutton/bigbluebutton.git \
    && cd bigbluebutton \
    && git sparse-checkout set bigbluebutton-config/bin/bbb-record record-and-playback/core/Gemfile record-and-playback/core/Gemfile.lock record-and-playback/core/Rakefile record-and-playback/core/lib record-and-playback/core/scripts record-and-playback/presentation/scripts \
    && git checkout tags/$TAG -b docker-build \
    && ls record-and-playback/core/* \
    && cd record-and-playback \
    && mv core /usr/local/bigbluebutton/ \
    && rsync -av presentation/scripts/ /usr/local/bigbluebutton/core/scripts/

# install ruby dependencies
RUN cd /usr/local/bigbluebutton/core \
    && gem install builder \
    && gem install bundler --no-document \
    && /usr/local/bin/bundle

# log to file instead of journald
RUN sed -i 's|Journald::Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/lib/recordandplayback.rb && \
    sed -i 's|Journald::Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/scripts/rap-caption-inbox.rb && \
    sed -i 's|Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/scripts/rap-process-worker.rb && \
    sed -i 's|Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/scripts/archive/archive.rb && \
    sed -i 's|Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/scripts/publish/presentation.rb && \
    sed -i 's|Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/scripts/utils/captions.rb && \
    sed -i 's|Logger\.new.*|Logger.new("/var/log/bigbluebutton/recording.log")|g' /usr/local/bigbluebutton/core/scripts/process/presentation.rb

# add bbb-record with some adjustments so bbb-record works in this environment
RUN mv bigbluebutton/bigbluebutton-config/bin/bbb-record /usr/bin \
    && cd /usr/bin \
    && chmod +x /usr/bin/bbb-record \
    && sed -i 's/^BBB_WEB.*/BBB_WEB=""/' /usr/bin/bbb-record \
    && sed -i 's/systemctl.*//' /usr/bin/bbb-record \
    && echo "BIGBLUEBUTTON_RELEASE=$TAG" > /etc/bigbluebutton/bigbluebutton-release

# create user
# the ID should match the one creating the files in `core`
RUN groupadd -g 998 bigbluebutton && useradd -m -u 998 -g bigbluebutton bigbluebutton

# change owner
# https://github.com/alangecker/bigbluebutton-docker/issues/63
RUN chown -R 998:998 /usr/local/bigbluebutton

COPY bbb-web.properties /etc/bigbluebutton/bbb-web.properties.tmpl
COPY bigbluebutton.yml /usr/local/bigbluebutton/core/scripts/bigbluebutton.yml.tmpl
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY entrypoint.sh /entrypoint.sh

ENTRYPOINT /entrypoint.sh
