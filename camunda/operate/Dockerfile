FROM alpine:latest as builder

ARG TARGETARCH

ARG VERSION

ENV TMP_DIR=/tmp/operate \
    TINI_VERSION=v0.19.0

ADD https://github.com/camunda-cloud/zeebe/releases/download/${VERSION}/camunda-cloud-operate-${VERSION}.tar.gz /tmp/operate.tar.gz

RUN mkdir -p ${TMP_DIR} && \
    tar xfvz /tmp/operate.tar.gz --strip 1 -C ${TMP_DIR}

ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-${TARGETARCH} ${TMP_DIR}/bin/tini
RUN chmod +x -R ${TMP_DIR}/bin/

FROM eclipse-temurin:17-jre-focal

ENV HOME=/usr/local/operate
ENV PATH "${HOME}/bin:${PATH}"

COPY --from=builder /tmp/operate /usr/local/operate

ENTRYPOINT ["tini", "--", "/usr/local/operate/bin/operate"]
