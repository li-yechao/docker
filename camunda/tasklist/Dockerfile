FROM alpine:latest as builder

ARG TARGETARCH

ARG VERSION

ENV TMP_DIR=/tmp/tasklist \
    TINI_VERSION=v0.19.0

ADD https://github.com/camunda-cloud/zeebe/releases/download/${VERSION}/camunda-cloud-tasklist-${VERSION}.tar.gz /tmp/tasklist.tar.gz

RUN mkdir -p ${TMP_DIR} && \
    tar xfvz /tmp/tasklist.tar.gz --strip 1 -C ${TMP_DIR}

ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-${TARGETARCH} ${TMP_DIR}/bin/tini
RUN chmod +x -R ${TMP_DIR}/bin/

FROM eclipse-temurin:17-jre-focal

ENV HOME=/usr/local/tasklist
ENV PATH "${HOME}/bin:${PATH}"

COPY --from=builder /tmp/tasklist /usr/local/tasklist

ENTRYPOINT ["tini", "--", "/usr/local/tasklist/bin/tasklist"]
