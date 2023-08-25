FROM rknop/devuan-daedalus-rknop AS sndata_installed
MAINTAINER Rob Knop <raknop@lbl.gov>

# First target just gets sndata installed.  It's a potentially slow step,
# so we want to populate the docker cache early
#
# I am not yet happy with this; I'd like to detect if SNDATA_ROOT_FILE doesn't
# exist, and not fail on the copy when it doesn't.

ARG SNDATA_ROOT_URL=https://zenodo.org/record/7416122/files/SNDATA_ROOT_2022-12-08.tar.gz?download=1
ARG SNDATA_ROOT_FILE=SNDATA_ROOT_2022-12-08.tar.gz

SHELL ["/bin/bash", "-c"]
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get -y update && apt-get -y install curl && apt-get clean && rm -rf /var/lib/apt/lists/*

# Place to compile stuff
RUN mkdir /usr/src/snana
WORKDIR /usr/src/snana

COPY ${SNDATA_ROOT_FILE} /usr/src/snana/sndata_root.tar.gz
RUN if [[ -e /usr/src/snana/sndata_root.tar.gz ]] ; \
    then echo "Using locally copied sndata_root" ; \
    else curl -L $SNDATA_ROOT_URL -o sndata_root.tar.gz ; \
    fi

RUN mkdir /sndata_root \
    && cd /sndata_root \
    && tar xpf /usr/src/snana/sndata_root.tar.gz

# ======================================================================
FROM rknop/devuan-daedalus-rknop AS snana
MAINTAINER Rob Knop <raknop@lbl.gov>

ARG SNANA_ARCHIVE=https://github.com/rknop/SNANA.git
ARG SNANA_CHECKOUT=docker

# Do this so that we don't double copy by including the tarfile
COPY --from=sndata_installed /sndata_root /sndata_root

# Update and install distro packages
RUN apt-get -y update \
    && apt-get -y upgrade \
    && TZ='UTC' apt-get -y install tzdata locales curl git python3 python3-pip \
                                   libgsl-dev gsl-bin libcfitsio-dev python3-fitsio \
                                   libncurses-dev gfortran gcc \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Generate the UTF8 locale
RUN cat /etc/locale.gen | perl -npe 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' > /etc/locale.gen.new \
    && mv /etc/locale.gen.new /etc/locale.gen
RUN locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

ENV SNDATA_ROOT /sndata_root
ENV SNANA_DIR /snana/SNANA

# Home directory (with place for astropy defaults during install)
# Mount the curveball working install here
RUN mkdir /snana
RUN mkdir /snana/.astropy
ENV HOME /snana
ENV PS1 "snana$ "

WORKDIR /snana
RUN git clone $SNANA_ARCHIVE \
    && cd /snana/SNANA \
    && git checkout $SNANA_CHECKOUT
