ARG VERSITYGW_VER=1.10.0

FROM docker.io/library/alpine:latest AS alpine

RUN --mount=type=cache,id=apk-cache,target=/var/cache/apk apk add upx

FROM ghcr.io/versity/versitygw:v${VERSITYGW_VER} AS versitygw


FROM alpine AS builder

WORKDIR /builder

RUN adduser -D -H versitygw

COPY --from=versitygw /usr/local/bin/versitygw ./versitygw
RUN upx --best --lzma ./versitygw
RUN mkdir -p /data/iam /data/versioning /data/s3 && chown versitygw:versitygw /data/iam /data/versioning /data/s3


FROM docker.io/library/alpine:latest

RUN adduser -D -H versitygw

ENV VGW_IAM_DIR=/data/iam
ENV VGW_VERSIONING_DIR=/data/versioning

COPY --from=builder /builder/versitygw /usr/local/bin/versitygw
COPY --from=builder /data /data
COPY --from=versitygw /usr/local/bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

USER versitygw

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
