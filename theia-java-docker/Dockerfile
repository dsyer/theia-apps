ARG NODE_VERSION=10
FROM ibmcom/ibmjava:8-sdk as java-base

FROM node:$NODE_VERSION
COPY --from=java-base /opt/ibm/java /opt/ibm/java

ENV JAVA_HOME=/opt/ibm/java/jre \
    PATH=/opt/ibm/java/jre/bin:/opt/ibm/java/bin/:$PATH

RUN apt-get update && \
    apt-get install -y curl apt-transport-https maven gradle && \
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && apt-get install -y yarn && \
    rm -rf /var/lib/apt/lists/*
# See : https://github.com/theia-ide/theia-apps/issues/34
RUN adduser --disabled-password --gecos '' theia
RUN chmod g+rw /home && \
    mkdir -p /home/project && \
    chown -R theia:theia /home/theia && \
    chown -R theia:theia /home/project;
WORKDIR /home/theia
USER theia

ARG version=latest
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN
RUN yarn --cache-folder ./ycache && rm -rf ./ycache
RUN yarn theia build
RUN mkdir plugins && \
    curl -L https://github.com/redhat-developer/vscode-java/releases/download/v0.54.2/redhat.java-0.54.2.vsix -o plugins/redhat.java-0.54.2.vsix && \
    curl -L https://github.com/microsoft/vscode-java-debug/releases/download/0.24.0/vscjava.vscode-java-debug-0.24.0.vsix -o plugins/vscjava.vscode-java-debug-0.24.0.vsix && \
    curl -L https://github.com/microsoft/vscode-java-test/releases/download/0.22.0/vscjava.vscode-java-test-0.22.0.vsix -o plugins/vscjava.vscode-java-test-0.22.0.vsix && \
curl -L https://marketplace.visualstudio.com/_apis/public/gallery/publishers/vscjava/vsextensions/vscode-maven/0.20.1/vspackage | gunzip -c - > plugins/vscjava.vscode-maven-0.20.1.vsix
EXPOSE 3000
ENV SHELL /bin/bash
ENV JAVA_HOME /opt/ibm/java
ENTRYPOINT [ "yarn", "theia", "start", "/home/project", "--hostname=0.0.0.0", "--plugins=local-dir:./plugins" ]
