
FROM ubuntu:14.04

MAINTAINER pete.nelson@unitedlex.com

# needed so runit will install correctly
RUN touch /etc/inittab

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

    runit \
    # required for process supervising

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

ADD ushlex-0.99.tar.gz install_dependencies.sh /setup/

# download, install, and cleanup the things in crits_dependencies repo
RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get update \
 && cd /setup \
 && git clone https://github.com/crits/crits_dependencies \
 && cd /setup/crits_dependencies \
 # use OUR install_dependencies.sh, not git's
 && ../install_dependencies.sh \
 && cd / \
 && rm -r /setup/crits_dependencies \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# this seems to be an oddity missing in Ubuntu 14.04
RUN cd /setup \
 && cd /setup/ushlex-0.99 \
 && python ./setup.py install

# download the main and plugin repositories from github
RUN cd /setup \
 # TODO: download tarballs from github instead of clone (can also remove git install above then)
 && git clone https://github.com/crits/crits \
 && git clone https://github.com/crits/crits_services \
 #TODO: figure out how to delete git structure in crits without webapp complaining
 && rm -r crits_services/.git

#TODO: install appropriate services

# configure the webapp
RUN mkdir -p /data/db \
 && mv /setup/crits /data/ \
 && cd /data/crits/crits/config \
 && cp database_example.py database.py \
 && sed -i -e "1iimport sys; sys.path.append('/data/crits')" /data/crits/django.wsgi

# adjust users and permissions
RUN useradd crits && chgrp -R crits /data/crits \
 && touch /data/crits/logs/crits.log \
 && chmod 664 /data/crits/logs/crits.log \
 && usermod -aG www-data crits \
 && chown -R www-data /data/crits

# the apache2 run script takes care of starting mongo and cron first if needed
ADD apache/ /etc/service/apache/
ADD runit_bootstrap /usr/sbin/
RUN chmod +x /etc/service/apache/run /etc/service/apache/mongo/run /etc/service/apache/cron/run /usr/sbin/runit_bootstrap

# only need this if we're using the internal MongoDB install
# recommend using a separate mongo container and linking to it
VOLUME /data/db

EXPOSE 443
EXPOSE 80

ENTRYPOINT ["/usr/sbin/runit_bootstrap"]
CMD ["help"]
