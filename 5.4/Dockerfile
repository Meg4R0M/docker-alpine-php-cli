FROM meg4r0m/docker-alpine-php:5.4

LABEL php.version=5.4
LABEL maintainer="Florian Durano <psykoterro@gmail.com>" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.vendor="meg4r0m" \
    org.label-schema.name="php-cli-alpine" \
    org.label-schema.description="small php-cli image based on alpine" \
    org.label-schema.vcs-url="https://github.com/psykoterro/docker-php-cli-alpine"

ARG DOCKER_TIMEZONE=Europe/Paris
ARG DOCKER_LOCALE=fr_FR.UTF-8

# Configure timezone
# -----------------------------------------------------------------------------
RUN echo $DOCKER_TIMEZONE > /etc/timezone; dpkg-reconfigure -f noninteractive tzdata

# Base packages
# Java (used for eg. eZTika indexing)
# poppler-utils used for pdftotext binary (note that eztika is a much better option!)
# Mysql client, useful for db dumps and such
# -----------------------------------------------------------------------------
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    wget \
    bash-completion \
    netcat \
    git \
    pwgen \
    locales \
    sudo \
    cron \
    unzip \
    vim \
    nano \
    mysql-client \
    imagemagick \
    jpegoptim \
    poppler-utils \
    openjdk-7-jre-headless

# Configure locale for date, time & currency php functions
# -----------------------------------------------------------------------------
RUN dpkg-reconfigure locales && \
    echo $DOCKER_LOCALE' UTF-8'>> /etc/locale.gen && \
    locale-gen $DOCKER_LOCALE && \
    /usr/sbin/update-locale LANG=$DOCKER_LOCALE

ENV LC_ALL $DOCKER_LOCALE
ENV LANG $DOCKER_LOCALE
ENV LANGUAGE $DOCKER_LOCALE

# Nodejs global install (https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)
# -----------------------------------------------------------------------------
RUN curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash - && \
    apt-get install -y nodejs

# Composer global install
# -----------------------------------------------------------------------------
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin && \
    mv /usr/local/bin/composer.phar /usr/local/bin/composer && \
    chmod 755 /usr/local/bin/composer

# Local user
# -----------------------------------------------------------------------------
RUN mkdir /tmp/profile
COPY conf/.bashrc_append /tmp/profile/.bashrc_append
COPY conf/.gitconfig /tmp/profile/.gitconfig
COPY conf/.vimrc_append /tmp/profile/.vimrc_append

# nb: the 1000 used here for user id and group id is later on replaced by the code in bootstrap.sh...
RUN addgroup --gid 1000 site && \
    adduser --uid 1000 --ingroup site --home /home/site --shell /bin/bash --disabled-password --gecos "" site && \
    sed -i '$ a site   ALL=\(ALL:ALL\) NOPASSWD: ALL' /etc/sudoers

COPY conf/git-completion.bash /home/site/.git-completion.bash
COPY conf/git-prompt.sh /home/site/.git-prompt.sh

RUN cp /etc/skel/.* /home/site/; \
    cat /tmp/profile/.bashrc_append >> /home/site/.bashrc && \
    cat /tmp/profile/.vimrc_append >> /home/site/.vimrc && \
    cp /tmp/profile/.gitconfig /home/site/ && \
    mkdir -p /home/site/.composer && \
    mkdir -p /home/site/.ssh && \
    chown -R site:site /home/site/ && \
    find /tmp/ -type f -name '.*' -maxdepth 1  -delete

# Install fixuid (https://github.com/boxboat/fixuid) to map host user id with container 'site' user id (used in bootstrap.sh)
RUN USER=site && \
    GROUP=site && \
    curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.1/fixuid-0.1-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - && \
    chown root:root /usr/local/bin/fixuid && \
    chmod 4755 /usr/local/bin/fixuid && \
    mkdir -p /etc/fixuid && \
    printf "user: $USER\ngroup: $GROUP\n" > /etc/fixuid/config.yml

# -----------------------------------------------------------------------------

# Clear archives in apt cache folder
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

COPY conf/bootstrap.sh /run/bootstrap.sh

RUN chmod 755 /run/bootstrap.sh

USER site:site
WORKDIR /var/www/

CMD ["/run/bootstrap.sh"]