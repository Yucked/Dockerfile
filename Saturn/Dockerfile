FROM alpine:edge

LABEL maintainer="yucked@pm.me"
LABEL name="Saturn"

# Install requirements
RUN apk add --no-cache \
    ca-certificates \
    less \
    ncurses-terminfo-base \
    tzdata \
    userspace-rcu \
    curl \
    R \
    procps \
    bash \
    libgdiplus --repository https://dl-3.alpinelinux.org/alpine/edge/testing/ \
    lttng-ust --repository https://dl-cdn.alpinelinux.org/alpine/edge/main

# Setup glibc
RUN curl --remote-name-all https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/{glibc-2.34-r0.apk,file2}
RUN curl https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub -o /etc/apk/keys/sgerrand.rsa.pub && \
    curl -L https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-2.34-r0.apk -o glibc-2.34-r0.apk && \
    apk add --allow-untrusted --force-overwrite glibc-2.34-r0.apk && \
    curl -L https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-bin-2.34-r0.apk -o glibc-bin-2.34-r0.apk && \
    apk add --allow-untrusted --force-overwrite glibc-bin-2.34-r0.apk && \
    curl -L https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-i18n-2.34-r0.apk -o glibc-i18n-2.34-r0.apk && \
    apk add --allow-untrusted --force-overwrite glibc-i18n-2.34-r0.apk

# Install Miniconda
ARG CONDA_ROOT=/home/miniconda
RUN curl -L https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -o miniconda.sh
RUN chmod +x miniconda.sh && ./miniconda.sh -b -f -p ${CONDA_ROOT} && rm miniconda.sh &&  \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh

# Install PowerShell
ARG PS_FILE=powershell-7.2.0-linux-alpine-x64.tar.gz PS_LOC=/opt/microsoft/powershell/7
RUN curl -L https://github.com/PowerShell/PowerShell/releases/download/v7.2.0/${PS_FILE} -o ${PS_FILE} && \
    mkdir -p ${PS_LOC} && tar zxf ${PS_FILE} -C ${PS_LOC} && \
    chmod +x ${PS_LOC}/pwsh && rm ${PS_FILE} && \
    ln -s ${PS_LOC}/pwsh /usr/bin/pwsh

# Install DotNet
RUN curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel STS && \
    ln -s ${DOTNET_ROOT}/dotnet /usr/bin/dotnet

# Setup Environment Variables
ENV DOTNET_CLI_TELEMETRY_OPTOUT=true \
    DOTNET_INTERACTIVE_CLI_TELEMETRY_OPTOUT=true \
    NUGET_XMLDOC_MODE=skip \
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    JUPYTER_PORT=9090 \
    NOTEBOOKS_DIR=/root/notebooks
ENV DOTNET_ROOT=/root/.dotnet
ENV PATH=$PATH:${CONDA_ROOT}/bin:${DOTNET_ROOT}:${DOTNET_ROOT}/tools:${PS_LOC}:${DOTNET_ROOT}/dotnet

# Install C++, Jupyterlab, update packages, .Net and R Kernel
RUN conda install -y sqlite jupyterlab xeus-cling -c conda-forge && conda update --all && \
    dotnet tool install -g Microsoft.dotnet-interactive && \
    dotnet interactive jupyter install && \     
    R | bash /dev/stdin "install.packages(c('repr', 'IRdisplay', 'IRkernel'), type = 'source')" && \
    R | bash /dev/stdin "IRkernel::installspec(user = FALSE)" && \
    mkdir ${NOTEBOOKS_DIR}

EXPOSE ${JUPYTER_PORT}
WORKDIR ${NOTEBOOKS_DIR}
ENTRYPOINT jupyter lab --allow-root --ip=0.0.0.0 --port=${JUPYTER_PORT} --no-browser \
    --NotebookApp.allow_password_change=False \
    --NotebookApp.notebook_dir=${NOTEBOOKS_DIR} \
    --NotebookApp.token=""
