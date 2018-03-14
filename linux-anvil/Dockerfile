FROM ubuntu:16.04

MAINTAINER conda-forge <conda-forge@googlegroups.com>


# Set an encoding to make things work smoothly.
ENV LANG en_US.UTF-8

# Add a timestamp for the build. Also, bust the cache.
ADD http://tycho.usno.navy.mil/timer.html /opt/docker/etc/timestamp

# Resolves a nasty NOKEY warning that appears when using yum.
# RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

# Install basic requirements.
RUN apt update -y && \
    apt install -y \
                   bzip2 \
                   make \
                   patch \
                   grep \
                   sudo \
                   tar \
                   openjdk-8-jre \
                   curl \
                   openssl \
                   nano \
                   vim \
                   unzip && \
    apt clean all

# Install devtoolset 2.
# RUN apt update -y && \
#     apt install -y \
#                    centos-release-scl \
#                    apt-utils && \
#     # apt-config-manager --add-repo http://people.centos.org/tru/devtools/devtools.repo && \
#     # apt update -y && \
#     # apt install -y \
#     #                devtoolset-5-binutils \
#     #                devtoolset-5-gcc \
#     #                devtoolset-5-gcc-c++ && \
#     apt install -y devtoolset-6 && \
#     apt clean all

# Run everything from here in bash, instead of sh
# https://stackoverflow.com/questions/20635472/using-the-run-instruction-in-a-dockerfile-with-source-does-not-work/45087082#45087082
SHELL ["/bin/sh", "-c"]
# give sudo permission for conda user to run yum (user creation is postponed
# to the entrypoint, so we can create a user with the same id as the host)
RUN echo 'conda ALL=NOPASSWD: /usr/bin/apt' >> /etc/sudoers

# Install the latest Miniconda with Python 3 and update everything.
RUN curl -s -L https://repo.continuum.io/miniconda/Miniconda3-4.3.21-Linux-x86_64.sh > miniconda.sh && \
    openssl md5 miniconda.sh | grep c1c15d3baba15bf50293ae963abef853 && \
    bash miniconda.sh -b -p /opt/conda && \
    rm miniconda.sh && \
    touch /opt/conda/conda-meta/pinned && \
    export PATH=/opt/conda/bin:$PATH && \
    conda config --set show_channel_urls True && \
    conda config --add channels conda-forge && \
    conda update --all --yes && \
    conda clean -tipy

# Install conda build and deployment tools.
RUN export PATH="/opt/conda/bin:${PATH}" && \
    conda install --yes --quiet conda-build=3 anaconda-client jinja2 setuptools cmake && \
    conda install --yes git gosu tini && \
    conda clean -tipsy

# Install docker tools.
RUN export PATH="/opt/conda/bin:${PATH}" && \
    export CONDA_GOSU_INFO=`conda list gosu | grep gosu` && \
    echo "gosu ${CONDA_GOSU_INFO[1]}" >> /opt/conda/conda-meta/pinned && \
    export CONDA_TINI_INFO=`conda list tini | grep tini` && \
    echo "tini ${CONDA_TINI_INFO[1]}" >> /opt/conda/conda-meta/pinned && \
    conda clean -tipsy

# Add a file for users to source to activate the `conda`
# environment `root` and the devtoolset compiler. Also
# add a file that wraps that for use with the `ENTRYPOINT`.
COPY entrypoint_source /opt/docker/bin/entrypoint_source
COPY entrypoint /opt/docker/bin/entrypoint

# Ensure that all containers start with tini and the user selected process.
# Activate the `conda` environment `root` and the devtoolset compiler.
# Provide a default command (`bash`), which will start if the user doesn't specify one.
ENTRYPOINT [ "/opt/conda/bin/tini", "--", "/opt/docker/bin/entrypoint" ]
CMD [ "/bin/bash" ]
