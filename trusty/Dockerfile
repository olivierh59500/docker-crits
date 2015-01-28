
FROM ubuntu:14.04

MAINTAINER pete.nelson@unitedlex.com

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get update \
 && apt-get install -y \

    apache2 \
    # required for core functionality

    build-essential \
    # required for building certain dependencies

    curl \
    # required for ?

    debianutils \
    # required for which command in install_dependencies.sh (doesn't work anyway)

    git \
    # required for next commands in dockerfile

    libapache2-mod-wsgi \
    # required for core functionality

    libfuzzy-dev \
    # required for pyssdeep

    libpcap-dev \
    libpcre3-dev \
    numactl \
    # required for core functionality

    p7zip-full \
    # required for ?

    python2.7 \
    python-simplejson \
    python-pycurl \
    python-dev \
    python-pydot \
    python-pyparsing \
    python-yaml \
    # required for core functionality

    python-setuptools \
    # required for:

    python-numpy \
    python-matplotlib \
    python-lxml \
    # required for core functionality

    sudo \
    # required for install_dependencies script

    unar \
    # may be a future substitute for unrar*

    unrar-free \
    # required for ? (may need unrar-nonfree from multiverse)

    zip \
    # required for ?

    zsh \
    # required for one of the build scripts?

 && apt-get clean && rm -rf /var/lib/apt/lists/*

# save the repositories from github
RUN mkdir /setup \
 && cd /setup \
 # TODO: download tarballs from github instead of clone (can also remove git install above then)
 && git clone https://github.com/crits/crits \
 && git clone https://github.com/crits/crits_services \
 #TODO: figure out how to leave blank git structure in crits so webapp doesn't complain
 && rm -r crits_services/.git

ADD ushlex-0.99.tar.gz install_dependencies.sh /setup/

# download, install, and cleanup the things in crits_dependencies repo
RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get update \
 && cd /setup \
 && git clone https://github.com/crits/crits_dependencies \
 && cd /setup/crits_dependencies \
 && ../install_dependencies.sh \
 && cd / \
 && rm -r /setup/crits_dependencies \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# this seems to be an oddity missing in Ubuntu 14.04
RUN cd /setup \
 && cd /setup/ushlex-0.99 \
 && python ./setup.py install

# the hard work of actually configuring everything and initializing the DB
RUN mkdir -p /data/db \
 && mv /setup/crits /data/ \
 && cd /data/crits/crits/config \
 && cp database_example.py database.py \
 && sed -i -e "s/SECRET_KEY = ''/SECRET_KEY = 'KEYGOHERE!KEYGOHERE!KEYGOHERE!KEYGOHERE!KEYGOHERE!'/" database.py

# users and permissions
RUN useradd crits && chgrp -R crits /data/crits \
 && touch /data/crits/logs/crits.log \
 && chmod 664 /data/crits/logs/crits.log \
 && usermod -aG www-data crits \
 && chown -R www-data /data/crits

# configure mongodb
# TODO: DB and user init should probably be done on first run by entrypoint script
RUN cd /data/crits/contrib/mongo/NUMA \
 && mongod --fork --logpath /var/log/mongodb.log --logappend --nohttpinterface \
 && sleep 5 \
 && /data/crits/manage.py setconfig allowed_hosts '*' \
 && /data/crits/manage.py create_default_collections \
 && /data/crits/manage.py users -a -A -u admin -e crits@invalid.com -o Admins -f Default -l User \
 && sleep 5

#configure apache
ADD crits.conf crits-ssl.conf /etc/apache2/sites-available/
RUN a2dissite 000-default \
 && a2ensite crits \
 && a2ensite crits-ssl \
 && a2enmod ssl

# TODO: figure out the proper way to fix this
RUN sed -i -e "1iimport sys; sys.path.append('/data/crits')" /data/crits/django.wsgi

VOLUME /data/db

EXPOSE 443
EXPOSE 80

# hack to run the services in the background and show error.log
CMD ["/bin/bash", "-c", "/data/crits/contrib/mongo/UMA/mongod_start.sh && service apache2 start && tail -f /var/log/apache2/error.log"]
