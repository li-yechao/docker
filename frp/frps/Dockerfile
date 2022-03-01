FROM alpine:3 as builder

ARG TARGETARCH

ARG VERSION

ADD https://github.com/fatedier/frp/releases/download/v${VERSION}/frp_${VERSION}_linux_${TARGETARCH}.tar.gz /tmp/frp.tar.gz

RUN cd tmp \
	&& tar --strip 1 -xzvf frp.tar.gz

FROM alpine:3

WORKDIR /app

COPY --from=builder /tmp/frps /app
COPY --from=builder /tmp/frps.ini /app
COPY --from=builder /tmp/frps_full.ini /app

ENTRYPOINT ["/app/frps"]
