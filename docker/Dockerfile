FROM opensuse/tumbleweed

ARG ZYPP="zypper --non-interactive"

ENV RUNNER_NAME=""
ENV RUNNER_WORK_DIRECTORY="_work"
ENV RUNNER_TOKEN=""
ENV RUNNER_REPOSITORY_URL=""
ENV RUNNER_LABELS=""
ENV RUNNER_ALLOW_RUNASROOT=true
ENV GITHUB_ACCESS_TOKEN=""
ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache

RUN ${ZYPP} ref

# Required libraries and tools
RUN ${ZYPP} install tar gzip zip unzip autoconf automake dh-autoreconf libtool findutils bison ninja
RUN ${ZYPP} install icu ca-certificates lsb-release
# RUN ${ZYPP} install libstdc++ libstdc++-devel

# Build tools
# RUN ${ZYPP} install clang gcc gcc-c++
RUN ${ZYPP} install git curl cmake jq bash screen vim

# Powershell - https://github.com/PowerShell/PowerShell/releases
ARG PWSH_URL="https://github.com/PowerShell/PowerShell/releases/download/v7.3.1/powershell-7.3.1-1.rh.x86_64.rpm"
RUN ${ZYPP} install libicu libopenssl1_1
RUN rpm -ivh --nodeps ${PWSH_URL}

# Hugo - docutils -> rst2html
RUN ${ZYPP} install hugo docutils

RUN mkdir -p /home/runner ${AGENT_TOOLSDIRECTORY}
WORKDIR /home/runner

# Supervisor
RUN ${ZYPP} install supervisor
COPY supervisord.conf /etc/supervisord.conf
RUN chmod 644 /etc/supervisord.conf

# Netlify cli
RUN ${ZYPP} install npm
RUN npm install -g netlify-cli

RUN GH_RUNNER_VERSION=${GH_RUNNER_VERSION:-$(curl --silent "https://api.github.com/repos/actions/runner/releases/latest" | grep tag_name | sed -E 's/.*"v([^"]+)".*/\1/')} \
    && curl -L -O https://github.com/actions/runner/releases/download/v${GH_RUNNER_VERSION}/actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz \
    && tar -zxf actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz
RUN rm -f actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz
RUN chown -R root: /home/runner

# Entrypoint
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

# Start supervisor
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]