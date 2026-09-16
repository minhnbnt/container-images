FROM public.ecr.aws/docker/library/alpine:latest AS upx

RUN --mount=type=cache,id=apk-cache,target=/var/cache/apk apk add upx


FROM quay.io/hummingbird/curl:latest-builder AS fetcher

WORKDIR /fetcher

RUN curl -fLO "https://garagehq.deuxfleurs.fr/_releases/v2.4.1/x86_64-unknown-linux-musl/garage"


FROM upx AS builder

WORKDIR /builder

COPY --from=fetcher /fetcher/garage ./garage

RUN chmod +x ./garage
RUN upx --best --ultra-brute ./garage

FROM quay.io/hummingbird/static

COPY --from=builder /builder/garage /usr/bin/garage

ENTRYPOINT ["/usr/bin/garage"]
