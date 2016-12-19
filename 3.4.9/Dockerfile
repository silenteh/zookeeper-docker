FROM openjdk:8-jre-alpine
MAINTAINER Elisey Zanko <elisey.zanko@gmail.com>

# Install required packages
RUN apk add --no-cache \
    bash \
    su-exec

ENV ZOO_USER=zookeeper \
    ZOO_CONF_DIR=/opt/zookeeper/conf \
    ZOO_DATA_DIR=/opt/zookeeper/data \
    ZOO_DATA_LOG_DIR=/opt/zookeeper/datalog \
    ZOO_PORT=2181 \
    ZOO_TICK_TIME=2000 \
    ZOO_INIT_LIMIT=5 \
    ZOO_SYNC_LIMIT=2 \
    ZOO_UID=1234 

USER root    

# Add group
RUN set -x \
    && addgroup -g "$ZOO_UID" "$ZOO_USER" 

# Add a user and make dirs
RUN set -x \    
    && adduser -u "$ZOO_UID" -G "$ZOO_UID" -D "$ZOO_USER" \
    && mkdir -p "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR" \
    && chgrp -R 0 "$ZOO_USER:$ZOO_USER" "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR" \
    && chmod -R g+rw "$ZOO_USER:$ZOO_USER" "$ZOO_DATA_LOG_DIR" "$ZOO_DATA_DIR" "$ZOO_CONF_DIR"

USER $ZOO_UID    

ARG GPG_KEY=C823E3E5B12AF29C67F81976F5CECB3CB5E9BD2D

ARG DISTRO_NAME=zookeeper-3.4.9

# Download Apache Zookeeper, verify its PGP signature, untar and clean up
RUN set -x \
    && apk add --no-cache --virtual .build-deps \
        gnupg \
    && wget -q "http://www.apache.org/dist/zookeeper/$DISTRO_NAME/$DISTRO_NAME.tar.gz" \
    && wget -q "http://www.apache.org/dist/zookeeper/$DISTRO_NAME/$DISTRO_NAME.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" \
    && gpg --batch --verify "$DISTRO_NAME.tar.gz.asc" "$DISTRO_NAME.tar.gz" \
    && tar -xzf "$DISTRO_NAME.tar.gz" \
    && mv "$DISTRO_NAME/conf/"* "$ZOO_CONF_DIR" \
    && rm -r "$GNUPGHOME" "$DISTRO_NAME.tar.gz" "$DISTRO_NAME.tar.gz.asc" \
    && apk del .build-deps

WORKDIR $DISTRO_NAME

VOLUME ["$ZOO_DATA_DIR", "$ZOO_DATA_LOG_DIR"]

EXPOSE $ZOO_PORT 2888 3888

ENV PATH=$PATH:/$DISTRO_NAME/bin \
    ZOOCFGDIR=$ZOO_CONF_DIR

COPY docker-entrypoint.sh /opt/zookeeper/
COPY log4j.properties /opt/zookeeper/

# RUN set -x \
# && chown -R zookeeper:zookeeper /opt/zookeeper \
# && chown -R zookeeper:zookeeper /opt/zookeeper/log4j.properties \
# && chown -R zookeeper:zookeeper /$DISTRO_NAME \
# && chmod +x /opt/zookeeper/docker-entrypoint.sh \
# && chmod -R 777 /opt/zookeeper

ENTRYPOINT ["/opt/zookeeper/docker-entrypoint.sh"]

CMD ["zkServer.sh", "start-foreground"]
