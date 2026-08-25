FROM quay.io/hummingbird/curl:latest-builder AS fetcher

USER root

RUN dnf install -y gzip tar && dnf clean all

WORKDIR /fetcher

RUN curl -LO https://github.com/timescale/pg_textsearch/releases/download/v1.4.0/pg_textsearch-pg18-v1.4.0.tar.gz
RUN tar xvf pg_textsearch-pg18-v1.4.0.tar.gz

FROM quay.io/hummingbird/postgresql:18

COPY --from=fetcher /fetcher/pg18/pg_textsearch.so /usr/lib64/pgsql/

COPY --from=fetcher /fetcher/pg18/pg_textsearch.control /usr/share/pgsql/extension/
COPY --from=fetcher /fetcher/pg18/pg_textsearch--*.sql /usr/share/pgsql/extension/

USER root

RUN echo "shared_preload_libraries = 'pg_textsearch'" >> /usr/share/pgsql/postgresql.conf.sample
RUN echo "CREATE EXTENSION pg_textsearch;" >> /docker-entrypoint-initdb.d/01-init-pg_textsearch.sql

USER postgres
