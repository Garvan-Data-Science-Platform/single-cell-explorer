ARG BASE_IMAGE="533267185808.dkr.ecr.us-west-2.amazonaws.com/docker.io/sci/library/ubuntu:22.04"
FROM ${BASE_IMAGE}

ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y gettext nginx moreutils build-essential libxml2-dev python3-dev python3-pip zlib1g-dev python3-requests python3-aiohttp llvm jq git zip curl openssl && \
    curl -sL https://deb.nodesource.com/setup_20.x  | bash - && \
    apt-get -y install nodejs && \
    node -v && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

COPY hosted/nginx.conf /etc/nginx/nginx.conf
COPY hosted/nginx.ssl.conf* /etc/nginx/
ARG LOCALHOST_SSL_CERT=false
# Generate self-signed SSL certificate only when LOCALHOST_SSL_CERT=true
RUN if [ "$LOCALHOST_SSL_CERT" = "true" ]; then \
      mv /etc/nginx/nginx.ssl.conf /etc/nginx/nginx.conf; \
      openssl req -x509 -newkey rsa:4096 -nodes \
        -keyout /etc/ssl/private/nginx-selfsigned.key \
        -out /etc/ssl/certs/nginx-selfsigned.crt \
        -days 3650 \
        -subj "/CN=explorer.corporanet.local"; \
    fi

ADD server/requirements.txt .
RUN python3 -m pip install -r requirements.txt

ADD Makefile Makefile
ADD common.mk common.mk
ADD environment.default.json environment.default.json
ADD client/ client/
ENV DEPLOYMENT_TYPE=hosted
ARG USE_FOR_PORTAL=false
ENV USE_FOR_PORTAL=${USE_FOR_PORTAL}
RUN make build-client

ADD server/ server/
RUN mkdir -p /server/common/web/templates/

WORKDIR /server/ecs
RUN make build-ecs
WORKDIR /

ADD hosted/config.yaml config.yaml
ADD hosted/start.sh start.sh
RUN chmod +x /start.sh
ENV CXG_CONFIG_FILE=/config.yaml

ARG IMAGE_TAG
LABEL commit=${IMAGE_TAG}
ENV IMAGE_TAG=${IMAGE_TAG}

CMD ["/start.sh"]
