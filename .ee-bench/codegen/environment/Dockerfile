FROM mcr.microsoft.com/dotnet/sdk:8.0-noble
# Ubuntu 24.04

ENV DEBIAN_FRONTEND=noninteractive

# Ubuntu server blocks a lot of requests, so we use a mirror (version for Ubuntu 23+)
RUN sed -i 's|http://archive.ubuntu.com/ubuntu/|http://eu-west-1.ec2.archive.ubuntu.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources && \
    sed -i 's|http://security.ubuntu.com/ubuntu/|http://eu-west-1.ec2.archive.ubuntu.com/ubuntu/|g' /etc/apt/sources.list.d/ubuntu.sources && \
    apt-get update

# Install common packages
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    git \
    curl \
    wget \
    sudo \
    tar \
    zip \
    unzip \
    autoconf \
    # Required for installing Python
    libcurl4-gnutls-dev \
    # Required for Junie execution
    libexpat1-dev \
    libsecret-1-0 \
    libsecret-1-dev \
    rsync \
    python3 \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

RUN wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh && \
    chmod +x ./dotnet-install.sh && \
    ./dotnet-install.sh --version 9.0.300 --install-dir /usr/share/dotnet

RUN git clone https://github.com/{{ instance.owner }}/{{ instance.repo_name }}.git /app
WORKDIR /app
RUN git checkout {{ instance.base_commit }}

RUN cd MediaBrowser.Model && dotnet add package System.Text.Json --version 8.0.5 && cd .. && \
    cd Emby.Server.Implementations && dotnet add package Microsoft.Extensions.Caching.Memory --version 9.0.0 && cd .. && \
    cd tests/Jellyfin.Server.Tests && dotnet add package Microsoft.Extensions.Options --version 9.0.0 && cd ../.. && \
    dotnet add "./tests/Jellyfin.Model.Tests/Jellyfin.Model.Tests.csproj" package JUnitTestLogger && \
    dotnet add "./tests/Jellyfin.Naming.Tests/Jellyfin.Naming.Tests.csproj" package JUnitTestLogger && \
    dotnet add "./tests/Jellyfin.Server.Implementations.Tests/Jellyfin.Server.Implementations.Tests.csproj" package JUnitTestLogger && \
    dotnet build

LABEL ee-bench.type="codegen"
LABEL ee-bench.version="1.0"
RUN rm -rf /app/.ee-bench/ 2>/dev/null || true
