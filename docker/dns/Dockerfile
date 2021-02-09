FROM debian:buster-slim
RUN set -ex ;\
    apt-get update ;\
    apt-get install -y --no-install-recommends \
            iproute2 \
            dnsmasq \
            ca-certificates
COPY assets /etc/dnsmasq/