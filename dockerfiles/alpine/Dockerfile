FROM alpine:3.12.0

ENV APK_PACKAGES \
  ca-certificates \
  openssl \
  easy-rsa

COPY entrypoint.sh /usr/local/bin
RUN ln -s /usr/local/bin/entrypoint.sh / # backwards compat

COPY scripts/* /usr/share/easy-rsa-helpers/

RUN set -x \
    && \
    env \
    && \
    echo "==> Upgrading apk and system..." \
    && apk update && apk upgrade \
    && \
    echo "==> Installing required APKs..." \
    && apk add --no-cache ${APK_PACKAGES} \
    && \
    echo "===> Configuring..." \
    && chmod +x /usr/local/bin/entrypoint.sh \
    && mkdir /easy-rsa \
    && ln -s /usr/share/easy-rsa/* /easy-rsa/ \
    && chmod 700 /easy-rsa \
    && mkdir /pki \
    && chmod 700 /pki \
    && chmod +x /usr/share/easy-rsa-helpers/* \
    && mkdir /easy-rsa-helpers \
    && ln -s /usr/share/easy-rsa-helpers/* /easy-rsa-helpers/ \
    && chmod 700 /easy-rsa-helpers \
    && \
    echo "===> Cleaning up..." \
    && unset http_proxy https_proxy

ENV PATH "/easy-rsa:/easy-rsa-helpers:${PATH}"

WORKDIR /easy-rsa

# EasyRSA data volume
# Bind mount for host access to generated PKI assets
VOLUME [ "/.easyrsa" ]

ENTRYPOINT ["entrypoint.sh"]
CMD [ "./easyrsa", "--help" ]

# Labels
# http://label-schema.org/rc1/ namespace labels
LABEL maintainer="Ryan Craig"

LABEL org.label-schema.schema-version="1.0"
#LABEL org.label-schema.build-date=$BUILD_DATE
LABEL org.label-schema.name="devtestlabs/easyrsa"
LABEL org.label-schema.description="EasyRSA container"
#LABEL org.label-schema.url=$ORG_WEB_URL
#LABEL org.label-schema.vcs-url=$VCS_URL
#LABEL org.label-schema.vcs-ref=$VCS_REF
LABEL org.label-schema.vendor="devtestlabs.xyz"
#LABEL org.label-schema.version=$IMAGE_LONG_VERSION