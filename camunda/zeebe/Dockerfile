FROM alpine:latest as builder

ARG TARGETARCH

ARG VERSION

ENV TMP_DIR=/tmp/zeebe \
    TINI_VERSION=v0.19.0

ADD https://github.com/camunda-cloud/zeebe/releases/download/${VERSION}/camunda-cloud-zeebe-${VERSION}.tar.gz /tmp/zeebe.tar.gz

RUN mkdir -p ${TMP_DIR} && \
    tar xfvz /tmp/zeebe.tar.gz --strip 1 -C ${TMP_DIR} && \
    mkdir ${TMP_DIR}/data

ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-${TARGETARCH} ${TMP_DIR}/bin/tini
COPY startup.sh ${TMP_DIR}/bin/startup.sh
RUN chmod +x -R ${TMP_DIR}/bin/
RUN chmod 0775 ${TMP_DIR} ${TMP_DIR}/data

FROM eclipse-temurin:17-jre-focal

ENV ZB_HOME=/usr/local/zeebe \
    ZEEBE_BROKER_GATEWAY_NETWORK_HOST=0.0.0.0 \
    ZEEBE_STANDALONE_GATEWAY=false
ENV PATH "${ZB_HOME}/bin:${PATH}"

WORKDIR ${ZB_HOME}
EXPOSE 26500 26501 26502
VOLUME ${ZB_HOME}/data

RUN groupadd -g 1000 zeebe && \
    adduser -u 1000 zeebe --system --ingroup zeebe && \
    chmod g=u /etc/passwd && \
    chown 1000:0 ${ZB_HOME} && \
    chmod 0775 ${ZB_HOME}

COPY --from=builder --chown=1000:0 /tmp/zeebe/bin/startup.sh /usr/local/bin/startup.sh
COPY --from=builder --chown=1000:0 /tmp/zeebe ${ZB_HOME}

ENTRYPOINT ["tini", "--", "/usr/local/bin/startup.sh"]
