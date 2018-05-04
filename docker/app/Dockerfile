# Grab the 16.04 image of the ubuntu operating system
FROM ubuntu:16.04

MAINTAINER Sam Broughton

# Set the language and locale because this is not the default, and we'll need UTF-8 for the repository we use to install php because that repo requires UTF-8
RUN apt-get update \
    && apt-get install -y locales \
    && locale-gen en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN apt-get update \
# nginx for our web server
# curl in case we need to make curl requests
# zip, unzip & git are required by composer for certain operations
# software-properties-common to get the add-apt-repository command
# supervisor so we can run both nginx and php-fpm at the same time - it gets run when we start up the container and will be responsible for starting up php and nginx
# sqlite3 so we can use sqlite for the in-memory database when running phpunit tests
    && apt-get install -y nginx curl zip unzip git software-properties-common supervisor sqlite3 \
# adds a repository to get the latest php
    && add-apt-repository -y ppa:ondrej/php \
    && apt-get update \
# installs all our php dependencies for this specific project
    && apt-get install -y php7.0-fpm php7.0-cli php7.0-mcrypt php7.0-gd php7.0-mysql \
        php7.0-imap php-memcached php7.0-mbstring php7.0-xml php7.0-curl \
        php7.0-sqlite3 \
# installs composer - uses php to get the installer, runs it through php with flags, which install it to /usr/bin and names it composer rather than composer.phar
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin --filename=composer \
# makes sure /run/php exists just in case it wasn't created when everything else was installed - it's the location php-fpm is going to use to put some files when it runs
    && mkdir /run/php \
# we're no longer going to use the add-apt-repository so we can remove software-properties-common
    && apt-get remove -y  --purge software-properties-common \
# removes unused dependencies to free up space
    && apt-get -y autoremove \
# cleans up
    && apt-get clean \
# removes things we won't use, so stuff from the temp directories and repository package knowledge
    && rm -rf /tmp/* /var/tmp/ /var/lib/apit/lists/* \
# tells nginx to not run as a daemon, i.e. not run in the background, which is necessary in order for supervisor to run it properly
    && echo "daemon off;" >> /etc/nginx/nginx.conf \
# make 2 symlinks to spit out access and error logs into /dev/stdout and /dev/stderr because that's where docker searches for log outputs, letting us run 'docker log' to see the output of nginx stderr and stdout
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

# overwrites the default nginx configuration with our own so we can make sure it's set up for php
COPY default /etc/nginx/sites-available/default
# overwrites the default php-fpm.conf file so we can make a few minor tweaks - we'll stop it running as a daemon and reroute the stdout and stderr logs
COPY php-fpm.conf /etc/php/7.0/fpm/php-fpm.conf
# exposes port 80 for now as this container isn't going to worry about SSL for now (?)
EXPOSE 80
# copies our supervisord.conf file into the right place - this tells supervisor how to run nginx and php
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
# sets default command - uses the supervisord.conf file we just copied
CMD ["/usr/bin/supervisord"]
