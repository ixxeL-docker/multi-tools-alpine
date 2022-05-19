FROM gcr.io/kaniko-project/executor:latest as kaniko

FROM alpine:latest AS builder

SHELL ["/bin/ash", "-eo", "pipefail", "-c"]
RUN apk update --no-cache
RUN apk add curl npm tar --no-cache --update 
RUN curl -sLO "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64" 
RUN chmod +x ./argocd-linux-amd64 
RUN mv ./argocd-linux-amd64 /usr/local/bin/argocd 
RUN curl -sLO "https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64" 
RUN chmod +x ./kubectl-argo-rollouts-linux-amd64 
RUN mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts 
RUN curl -sLO "https://github.com/argoproj/argo-workflows/releases/latest/download/argo-linux-amd64.gz" 
RUN gunzip argo-linux-amd64.gz 
RUN chmod +x argo-linux-amd64 
RUN mv ./argo-linux-amd64 /usr/local/bin/argo 
RUN HELM_VERSION=$(curl -sSL "https://github.com/kubernetes/helm/releases/latest" | grep -oE 'v[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq | tail -1) 
RUN curl -sSL "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz" -o /tmp/helm.tar.gz
RUN tar -xzvf /tmp/helm.tar.gz -C /usr/local/bin
RUN ls -la /usr/local/bin
RUN mv linux-amd64/helm /usr/local/bin/helm 
RUN curl -sSL "https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz" -o /tmp/oc.tar.gz 
RUN tar -xzvf /tmp/oc.tar.gz -C /usr/local/bin 
RUN rm /tmp/oc.tar.gz 
RUN KUBECTL_VERSION=$(curl -s "https://storage.googleapis.com/kubernetes-release/release/stable.txt") 
RUN curl -sSLO "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" 
RUN chmod +x ./kubectl 
RUN mv ./kubectl /usr/local/bin/kubectl 
RUN VAULT_VERSION=$(curl -sSL "https://releases.hashicorp.com/vault" | grep -oE '/[0-9]+\.[0-9]+\.[0-9]+/' | head -1 | sed 's/\///g') 
RUN curl -sLO "https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip" 
RUN unzip "./vault_${VAULT_VERSION}_linux_amd64.zip" -d /usr/local/bin 
RUN ENVCONSUL_VERSION=$(curl -sSL "https://releases.hashicorp.com/envconsul" | grep -oE '/[0-9]+\.[0-9]+\.[0-9]+/' | head -1 | sed 's/\///g') 
RUN curl -sLO "https://releases.hashicorp.com/envconsul/${ENVCONSUL_VERSION}/envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" 
RUN unzip "./envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" -d /usr/local/bin 
RUN curl -fL "https://install-cli.jfrog.io" | sh 
RUN curl -sSfL "https://raw.githubusercontent.com/anchore/grype/main/install.sh" | sh -s -- -b /usr/local/bin 
RUN curl -sSfL "https://raw.githubusercontent.com/anchore/syft/main/install.sh" | sh -s -- -b /usr/local/bin 
RUN curl -sSL -o /usr/local/bin/semver "https://raw.githubusercontent.com/fsaintjacques/semver-tool/master/src/semver" 
RUN chmod +x /usr/local/bin/semver 
RUN npm install -g github-files-fetcher 
RUN fetcher --url="https://github.com/aquasecurity/trivy/tree/main/contrib" --out=/mnt 
RUN curl -sL "https://github.com/vmware-tanzu/carvel-ytt/releases/latest/download/ytt-linux-amd64" -o /usr/local/bin/ytt 
RUN chmod +x /usr/local/bin/ytt

FROM alpine:latest

LABEL maintainer="Frederic Spiers <fredspiers@gmail.com>" \
      component="CI/CD tools"
    
ENV TZ="Europe/Paris" \
    IMG_VERSION="2.0.0" \
    PATH="/kaniko:$PATH"
    
COPY --from=ixxel/musl-dns-hack-alpine /lib/ld-musl-x86_64.so.1 /lib/ld-musl-x86_64.so.1

COPY --from=kaniko /kaniko/executor /usr/local/bin/executor

COPY --from=builder /usr/local/bin/oc \
                    /usr/local/bin/argo \
                    /usr/local/bin/kubectl-argo-rollouts \
                    /usr/local/bin/grype \
                    /usr/local/bin/syft \
                    /usr/local/bin/jf \
                    /usr/local/bin/semver \
                    /usr/local/bin/envconsul \
                    /usr/local/bin/vault \
                    /usr/local/bin/argocd \
                    /usr/local/bin/helm \
                    /usr/local/bin/kubectl \
                    /usr/local/bin/ytt \
                    /usr/local/bin/
                    
COPY --from=builder /mnt/contrib ./contrib

WORKDIR /usr/app

COPY ./get-gitlab-project-id.py .
                    
RUN apk update --no-cache \
    && apk upgrade --no-cache \
    && apk add --no-cache --update \
    libc6-compat \
    podman \
    skopeo \
    buildah \
    img \ 
    git \
    wget \
    curl \
    gettext \
    bash \
    jq \
    yq \
    python3 \
    py3-pip \
    bind-tools \
    util-linux \
    tzdata \
    openssl \
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && rm -f /var/lib/containers/storage/libpod/bolt_state.db \
    && sed -i "s/driver = \"overlay\"/driver = \"vfs\"/" /etc/containers/storage.conf \
    && pip3 install --no-cache-dir awxkit ansible-tower-cli python-gitlab \
    && apk add trivy --no-cache --update --repository=http://dl-cdn.alpinelinux.org/alpine/edge/testing \
    && apk add github-cli glab terraform --no-cache --update --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && helm plugin install "https://github.com/chartmuseum/helm-push" \
    && helm plugin install "https://github.com/databus23/helm-diff" 
