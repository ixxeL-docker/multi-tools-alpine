FROM gcr.io/kaniko-project/executor:latest as kaniko

FROM alpine:latest AS builder

RUN apk update --no-cache \
    && apk add curl npm tar unzip --no-cache --update \
    && curl -sLO "https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64" \
    && chmod +x ./argocd-linux-amd64 \
    && mv ./argocd-linux-amd64 /usr/local/bin/argocd \
    && curl -sLO "https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64" \
    && chmod +x ./kubectl-argo-rollouts-linux-amd64 \
    && mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts \
    && curl -sLO "https://github.com/argoproj/argo-workflows/releases/latest/download/argo-linux-amd64.gz" \
    && gunzip argo-linux-amd64.gz \
    && chmod +x argo-linux-amd64 \
    && mv ./argo-linux-amd64 /usr/local/bin/argo \
#     && HELM_VERSION=$(curl -sSL "https://github.com/kubernetes/helm/releases/latest" | grep -oE 'v[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq | tail -1) \
#     && curl -sSL "https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz" | tar xzv \
#     && mv linux-amd64/helm /usr/local/bin/helm \
    && curl -sSL "https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz" | tar xzv \
    && mv linux-amd64/helm /usr/local/bin/helm \
    && curl -sSL "https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz" -o /tmp/oc.tar.gz \
    && tar -xzvf /tmp/oc.tar.gz -C /usr/local/bin \
    && rm /tmp/oc.tar.gz \
    && KUBECTL_VERSION=$(curl -s "https://storage.googleapis.com/kubernetes-release/release/stable.txt") \
    && curl -sSLO "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
    && chmod +x ./kubectl \
    && mv ./kubectl /usr/local/bin/kubectl \
    && VAULT_VERSION=$(curl -sSL "https://releases.hashicorp.com/vault" | grep -oE '/[0-9]+\.[0-9]+\.[0-9]+/' | head -1 | sed 's/\///g') \
    && curl -sLO "https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip" \
    && unzip "./vault_${VAULT_VERSION}_linux_amd64.zip" -d /usr/local/bin \
    && ENVCONSUL_VERSION=$(curl -sSL "https://releases.hashicorp.com/envconsul" | grep -oE '/[0-9]+\.[0-9]+\.[0-9]+/' | head -1 | sed 's/\///g') \
    && curl -sLO "https://releases.hashicorp.com/envconsul/${ENVCONSUL_VERSION}/envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" \
    && unzip "./envconsul_${ENVCONSUL_VERSION}_linux_amd64.zip" -d /usr/local/bin \
    && curl -fL "https://install-cli.jfrog.io" | sh \
    && curl -sSfL "https://raw.githubusercontent.com/anchore/grype/main/install.sh" | sh -s -- -b /usr/local/bin \
    && curl -sSfL "https://raw.githubusercontent.com/anchore/syft/main/install.sh" | sh -s -- -b /usr/local/bin \
    && curl -sSL -o /usr/local/bin/semver "https://raw.githubusercontent.com/fsaintjacques/semver-tool/master/src/semver" \
    && chmod +x /usr/local/bin/semver \
    && npm install -g github-files-fetcher \
    && fetcher --url="https://github.com/aquasecurity/trivy/tree/main/contrib" --out=/mnt \
    && curl -sL "https://github.com/vmware-tanzu/carvel-ytt/releases/latest/download/ytt-linux-amd64" -o /usr/local/bin/ytt \
    && chmod +x /usr/local/bin/ytt

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
    && apk add github-cli glab terraform yq --no-cache --update --repository=http://dl-cdn.alpinelinux.org/alpine/edge/community \
    && ln -s /usr/bin/python3 /usr/bin/python \
    && helm plugin install "https://github.com/chartmuseum/helm-push" \
    && helm plugin install "https://github.com/databus23/helm-diff" 
