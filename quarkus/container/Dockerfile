FROM registry.access.redhat.com/ubi9 AS ubi-micro-build

ENV KEYCLOAK_VERSION 999.0.0-SNAPSHOT
ARG KEYCLOAK_DIST=/libs/keycloak-999.0.0-SNAPSHOT.tar.gz

RUN dnf install -y tar gzip

ADD $KEYCLOAK_DIST /tmp/keycloak/

# The next step makes it uniform for local development and upstream built.
# If it is a local tar archive then it is unpacked, if from remote is just downloaded.
RUN (cd /tmp/keycloak && \
    tar -xvf /tmp/keycloak/keycloak-*.tar.gz && \
    rm /tmp/keycloak/keycloak-*.tar.gz) || true

RUN mv /tmp/keycloak/keycloak-* /opt/keycloak && mkdir -p /opt/keycloak/data
RUN chmod -R g+rwX /opt/keycloak

ADD ubi-null.sh /tmp/
RUN bash /tmp/ubi-null.sh java-17-openjdk-headless glibc-langpack-en findutils

FROM registry.access.redhat.com/ubi9-micro
ENV LANG en_US.UTF-8

COPY libs/keycloak-metrics-spi-5.0.0.jar /opt/keycloak/providers/keycloak-metrics-spi-5.0.0.jar
ADD libs/openremote-theme.jar /opt/keycloak/providers

# Flag for determining app is running in container
ENV KC_RUN_IN_CONTAINER true

COPY --from=ubi-micro-build /tmp/null/rootfs/ /
COPY --from=ubi-micro-build --chown=1000:0 /opt/keycloak /opt/keycloak

RUN echo "keycloak:x:0:root" >> /etc/group && \
    echo "keycloak:x:1000:0:keycloak user:/opt/keycloak:/sbin/nologin" >> /etc/passwd

USER 0
RUN rm -r /opt/keycloak/themes
RUN mkdir -p /deployment/keycloak/themes
RUN ln -s /deployment/keycloak/themes /opt/keycloak
USER 1000

WORKDIR /opt/keycloak

# Configure runtime options
ENV TZ=Europe/Amsterdam
ENV KC_DB=postgres
ENV KC_DB_URL_HOST=postgresql
ENV KC_DB_URL_PORT=5432
ENV KC_DB_URL_DATABASE=openremote
ENV KC_DB_SCHEMA=public
ENV KC_DB_USERNAME=postgres
ENV KC_DB_PASSWORD=postgres
ENV KC_HOSTNAME=localhost
ENV KC_HTTP_RELATIVE_PATH=/auth
ENV KC_PROXY=edge
ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=secret
ENV KC_LOG_LEVEL=info
ENV KEYCLOAK_DEFAULT_THEME=keycloak
ENV KEYCLOAK_ACCOUNT_THEME=keycloak
ENV KEYCLOAK_WELCOME_THEME=keycloak
ENV KEYCLOAK_START_COMMAND=start
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true
ENV KC_FEATURES=token-exchange

HEALTHCHECK --interval=3s --timeout=3s --start-period=30s --retries=120 CMD curl --fail --silent http://localhost:8080/auth || exit 1

EXPOSE 8080

#ENTRYPOINT /opt/keycloak/bin/kc.sh ${KEYCLOAK_START_COMMAND:-start} \
#            --db=${KC_DB:-postgres} --http-relative-path=${KC_HTTP_RELATIVE_PATH:-/auth} --features=${KC_FEATURES:-token-exchange} \
#            --health-enabled=${KC_HEALTH_ENABLED:-true} --metrics-enabled=${KC_METRICS_ENABLED:-true} \
#            --spi-theme-default=${KEYCLOAK_DEFAULT_THEME:-openremote} --spi-theme-account-theme=${KEYCLOAK_ACCOUNT_THEME:-openremote} \
#            --spi-theme-welcome-theme=${KEYCLOAK_WELCOME_THEME:-keycloak} ${KEYCLOAK_START_OPTS:-}

ENTRYPOINT /opt/keycloak/bin/kc.sh ${KEYCLOAK_START_COMMAND:-start} \
            --db=${KC_DB:-postgres} --http-relative-path=${KC_HTTP_RELATIVE_PATH:-/auth} --features=${KC_FEATURES:-token-exchange} \
            --health-enabled=${KC_HEALTH_ENABLED:-true} --metrics-enabled=${KC_METRICS_ENABLED:-true} ${KEYCLOAK_START_OPTS:-}