# https://hub.docker.com/r/phusion/baseimage/tags?name=bionic&page=1&ordering=last_updated
# 18.04 = bionic beaver
# 20.04 = focal fossa (not yet released by phusion
FROM phusion/baseimage:bionic-1.0.0

# imported! because the terminal does not support odd characters coming from the repository name ppa:ondrej/php
ENV LC_ALL C.UTF-8

ARG WORKSPACE_LOCAL_USER_PUID=1000
ARG WORKSPACE_LOCAL_USER_PGUID=1000
ARG WORKSPACE_LOCAL_USER=admin
ARG WORKSPACE_HOSTNAME=${WORKSPACE_HOSTNAME}
ARG GIT_NAME="Firstname Lastname"
ARG GIT_EMAIL=firstname.lastname@company.org
ARG BIN_PATH=/usr/local/bin/
ARG PHP_VERSIONS
# https://pecl.php.net/package/mcrypt
ARG PHP72_MCRYPT_VERSION=1.0.2
ARG PHP73_MCRYPT_VERSION=1.0.3
ARG PHP74_MCRYPT_VERSION="6cf8c800f2926b65867b10dfe582bbb4596d1b7e"
ARG PHP_MCRYPT_VERSION="6cf8c800f2926b65867b10dfe582bbb4596d1b7e"
# https://golang.org/dl/
ARG GOLANG_VERSION=1.15.8
# https://github.com/nvm-sh/nvm/tags
ARG NVM_VERSION=v0.37.2
# https://nodejs.org/en/download/current/
ARG NVM_LATEST_LTS_VERSION=14.15.5
ARG NVM_CURRENT_VERSION=15.8.0
ENV NVM_DIR=/usr/local/nvm

# prevent error like 'debconf: unable to initialize frontend: Dialog' because not all packages support 'interactive' mode
# change the way debconf (Debian Package Configuration System) configures packages
# 'noninteractive' is needed for the build steps as all packages must work in noninteractive mode as well
ENV DEBIAN_FRONTEND noninteractive

# https://blog.nutsfactory.net/2008/03/06/noninteractive-dpkg-installation-on-debian-system/
# set correct values for localepurge installation (save some disk space - yeah!)
RUN ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/quickndirtycalc boolean true\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/remove_no note\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/mandelete boolean true\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/showfreedspace boolean true\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/verbose boolean true\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/nopurge multiselect en_US.UTF-8\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/dontbothernew boolean false\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"localepurge localepurge/none_selected boolean false\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"postfix postfix/mailname string ${WORKSPACE_HOSTNAME}\""] \
    && ["/bin/bash", "-c", "debconf-set-selections <<< \"postfix postfix/main_mailer_type string 'Internet Site'\""]

# add a non-root user
RUN groupadd -g ${WORKSPACE_LOCAL_USER_PGUID} ${WORKSPACE_LOCAL_USER} \
    && useradd -m -u ${WORKSPACE_LOCAL_USER_PUID} -g ${WORKSPACE_LOCAL_USER_PGUID} ${WORKSPACE_LOCAL_USER}

# add PHP 7.x/8.x & Yarn repositories
RUN add-apt-repository -y ppa:ondrej/php \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# install basic software
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        localepurge \
    # https://github.com/tianon/docker-brew-ubuntu-core/issues/122#issuecomment-380517413
    # reinstate php man pages otherwise errors occur during 'switchphp' command
    && echo "" >> /etc/dpkg/dpkg.cfg.d/excludes \
    && echo "# ... except php man pages ..." >> /etc/dpkg/dpkg.cfg.d/excludes \
    && echo "path-include=/usr/share/man/man1/php*" >> /etc/dpkg/dpkg.cfg.d/excludes \
    && echo "path-include=/usr/share/man/man1/phar*" >> /etc/dpkg/dpkg.cfg.d/excludes \
    # delete all locales except en_US.UTF-8
    && locale-gen --purge en_US.UTF-8 \
    && localepurge

RUN apt-get install -y --no-install-recommends \
        apt-utils \
        bash \
        mlocate \
        net-tools \
        vim \
        curl \
        sudo \
        wget \
        tcpdump \
        telnet \
        traceroute \
        dnsutils \
        procps \
        inetutils-ping \
        yarn \
        mysql-client \
        zip \
        unzip \
        git \
        # mcrypt dependency
        libmcrypt-dev \
        postfix \
    # set vim as default editor
    && update-alternatives --set editor /usr/bin/vim.basic

# install two versions of node using NVM (Node Version Manager)
RUN mkdir -p ${NVM_DIR} \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/${NVM_VERSION}/install.sh | bash \
    && chmod +x ${NVM_DIR}/nvm.sh \
    # source bashrc for NVM (node/npm) to be loaded
    && . ~/.bashrc \
    # install current version (latest features)
    && nvm install ${NVM_CURRENT_VERSION} \
    # install latest LTS version (recommended for most users)
    && nvm install ${NVM_LATEST_LTS_VERSION} \
    # set the latest LTS version as default
    && nvm alias default ${NVM_LATEST_LTS_VERSION} \
    && nvm use default

# install PHP versions
# https://www.php.net/manual/en/extensions.alphabetical.php
RUN for i in ${PHP_VERSIONS}; do \
    \
    apt-get install -y --no-install-recommends \
        php${i}-common \
        php${i}-dev \
        php${i}-bcmath \
        php${i}-cli \
        php${i}-curl \
        php${i}-common \
        php${i}-gd \
        php${i}-intl \
        php${i}-mbstring \
        php${i}-memcached \
        php${i}-mysql \
        php${i}-mysqli \
        php${i}-mysqlnd \
        php${i}-pgsql \
        php${i}-readline \
        php${i}-soap \
        php${i}-sqlite \
        php${i}-sqlite3 \
        php${i}-xdebug \
        php${i}-xml \
        php${i}-zip \
    \
    # mark it as 'manual install', which means it won't get 'apt-get autoremove' at the end
    && apt-mark manual php${i}-dev \
    # install libmcrypt dependency for < PHP7.2
    # this extension has been moved to the PECL repository and is no longer bundled with PHP as of PHP 7.2.0
    # (per PHP 7.2 mcrypt extension is no longer supported in favor of the libsodium cryptography library)
    && if [ "${i}" = "5.6" ] || [ "${i}" = "7.0" ] || [ "${i}" = "7.1" ]; then \
        # the old school way of installing mcrypt
        apt-get install -y --no-install-recommends \
            php${i}-mcrypt \
    ;else \
        if [ ! -d "/tmp/mcrypt" ]; then \
            cd /tmp \
            && git clone https://git.php.net/repository/pecl/encryption/mcrypt.git \
        ;fi \
        &&  cd /tmp/mcrypt \
        && if [ "${i}" = "7.2" ]; then \
            git checkout ${PHP72_MCRYPT_VERSION} \
        ;elif [ "${i}" = "7.3" ]; then \
            git checkout ${PHP73_MCRYPT_VERSION} \
        ;elif [ "${i}" = "7.4" ]; then \
            git checkout ${PHP74_MCRYPT_VERSION} \
        # only PHP 8.0 remains
        ;else \
            git checkout ${PHP_MCRYPT_VERSION} \
        ;fi \
        # https://pear.php.net/manual/en/pyrus.commands.build.php
        && phpize${i} --clean \
        && phpize${i} \
        && ./configure --with-php-config=/usr/bin/php-config${i} \
        && make \
        && make install \
        # enable the mcrypt extension
        && echo -e '; configuration for php mcrypt module\n; priority=20\nextension=mcrypt.so' > /etc/php/${i}/mods-available/mcrypt.ini \
        && phpenmod -v ${i} mcrypt \
    ;fi \
    \
    #  ext-json is not applicable in PHP8
    && if [ "${i}" != "8.0" ]; then \
        apt-get install -y --no-install-recommends \
            php${i}-json \
    ;fi \
;done

# remove pecl mcrypt source
RUN rm -rf /tmp/mcrypt

# install globally popular PHP dev tools
RUN \
    # install latest Composer globally
    curl -LsS https://getcomposer.org/composer.phar -o ${BIN_PATH}composer \
    && chmod u+x ${BIN_PATH}composer \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}composer \
    \
    # install Composer V1 globally
    && curl -LsS https://getcomposer.org/composer-1.phar -o ${BIN_PATH}composerv1 \
    && chmod u+x ${BIN_PATH}composerv1 \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}composerv1 \
    \
    # install CodeCeption globally
    && curl -LsS http://codeception.com/codecept.phar -o ${BIN_PATH}codeception \
    && chmod u+x ${BIN_PATH}codeception \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}codeception \
    \
    # install Deployer globally
    && curl -LsS http://deployer.org/deployer.phar -o ${BIN_PATH}deployer \
    && chmod u+x ${BIN_PATH}deployer \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}deployer \
    \
    # install PHPmd globally
    && curl -LsS https://phpmd.org/static/latest/phpmd.phar -o ${BIN_PATH}phpmd \
    && chmod u+x ${BIN_PATH}phpmd \
    \
    # install PHPcpd globally
    && curl -LsS https://phar.phpunit.de/phpcpd.phar -o ${BIN_PATH}phpcpd \
    && chmod u+x ${BIN_PATH}phpcpd \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpcpd \
    \
    # install PHPcs globally
    && curl -LsS https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar -o ${BIN_PATH}phpcs \
    && chmod u+x ${BIN_PATH}phpcs \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpcs \
    \
    # install PHPcbf globally
    && curl -LsS https://squizlabs.github.io/PHP_CodeSniffer/phpcbf.phar -o ${BIN_PATH}phpcbf \
    && chmod u+x ${BIN_PATH}phpcbf \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpcbf \
    \
    # install PHPUnit globally (only works for PHP > 7.3) and make it the default
    && curl -LsS https://phar.phpunit.de/phpunit-9.5.1.phar -o ${BIN_PATH}phpunit9 \
    && chmod u+x ${BIN_PATH}phpunit9 \
    && ln -sf ${BIN_PATH}phpunit9 /usr/local/bin/phpunit \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpunit9 \
    \
    # install PHPUnit globally (only works for PHP 7.1, PHP 7.2)
    && curl -LsS https://phar.phpunit.de/phpunit-7.5.20.phar -o ${BIN_PATH}phpunit7 \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpunit7 \
    && chmod u+x ${BIN_PATH}phpunit7 \
    \
    # install PHPUnit globally (only works for PHP 7.0)
    && curl -LsS https://phar.phpunit.de/phpunit-6.5.14.phar -o ${BIN_PATH}phpunit6 \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpunit6 \
    && chmod u+x ${BIN_PATH}phpunit6 \
    \
    # install PHPUnit globally (works for < PHP 7.0)
    && curl -LsS https://phar.phpunit.de/phpunit-5.7.27.phar -o ${BIN_PATH}phpunit5 \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}phpunit5 \
    && chmod u+x ${BIN_PATH}phpunit5 \
    \
    # install Golang globally
    && curl -LsS https://golang.org/dl/go${GOLANG_VERSION}.linux-amd64.tar.gz -o /tmp/golang.tgz \
    && tar -C /tmp -xzf /tmp/golang.tgz \
    && mv /tmp/go /usr/local \
    && ln -s /usr/local/go/bin/go ${BIN_PATH}go \
    && ln -s /usr/local/go/bin/gofmt ${BIN_PATH}gofmt \
    && chown ${WORKSPACE_LOCAL_USER}:  ${BIN_PATH}go \
    && chmod u+x ${BIN_PATH}go \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}gofmt \
    && chmod u+x ${BIN_PATH}gofmt \
    \
    # install splitsh-lite
    && curl -LsS https://github.com/splitsh/lite/releases/download/v1.0.1/lite_linux_amd64.tar.gz -o ${BIN_PATH}splitsh-lite.tgz \
    && tar -zxf ${BIN_PATH}splitsh-lite.tgz -C ${BIN_PATH} \
    && rm ${BIN_PATH}splitsh-lite.tgz \
    && chown ${WORKSPACE_LOCAL_USER}: ${BIN_PATH}splitsh-lite \
    && chmod u+x ${BIN_PATH}splitsh-lite

# install npm packages, bower, gulp, node-sass (Fast SASS compiler written in C)
# somehow NVM installs using user 'nobody' (id: 1001)
# during package installation we'll continue down this path
# source bashrc for NVM (node/npm) to be loaded
RUN chown -R nobody:nogroup ${NVM_DIR} \
    && . ~/.bashrc \
    && npm install -g \
        bower \
        gulp \
        node-sass \
        webpack \
        @angular/cli \
    # chown NVM for the workspace user
    && chown -R ${WORKSPACE_LOCAL_USER}: ${NVM_DIR}

# allow the local user to execute sudo without requiring a password
RUN sed -i "s|${WORKSPACE_LOCAL_USER} ALL=(ALL) ALL|${WORKSPACE_LOCAL_USER} ALL=(ALL) NOPASSWD: ALL|" "/etc/sudoers" \
    # set inet_interfaces to lookback-only
    && sed -i -e 's|inet_interfaces = all|inet_interfaces = loopback-only|g' /etc/postfix/main.cf

# clean up
RUN chown -R ${WORKSPACE_LOCAL_USER} /home/${WORKSPACE_LOCAL_USER} \
    && apt-get clean \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc /usr/share/locale /var/log/apt/* /var/log/*.log \
    && rm -rf /etc/php/*/cli/php.ini

# !imporant!
# fall back to the 'teletype' for  debconf
# this way both interactive and non-interactive modes get set
# 'teletype' is needed for interactive configuration steps the user might want to do inside a running container
ENV DEBIAN_FRONTEND teletype

# NPM
EXPOSE 4200
