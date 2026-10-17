ARG PG_MAJOR=18

ARG FEDORA_VER=44
ARG ARCH=x86_64
ARG YUM_BASE=https://download.postgresql.org/pub/repos/yum/${PG_MAJOR}/fedora/fedora-${FEDORA_VER}-${ARCH}

ARG TEXTSEARCH_VER=1.4.0
ARG VECTOR_VER=0.8.6
ARG PGMQ_VER=1.10.0

ARG BASE_BUILDER=quay.io/hummingbird/curl:latest-builder
ARG BASE_PG=quay.io/hummingbird/postgresql:${PG_MAJOR}

FROM ${BASE_BUILDER} AS fetcher

USER root

RUN dnf install -y gzip tar cpio && dnf clean all

WORKDIR /fetcher


FROM fetcher AS fetcher-textsearch
ARG PG_MAJOR
ARG FEDORA_VER
ARG ARCH
ARG TEXTSEARCH_VER
ARG YUM_BASE

RUN curl -LO ${YUM_BASE}/pg_textsearch_${PG_MAJOR}-${TEXTSEARCH_VER}-1PGDG.f${FEDORA_VER}.${ARCH}.rpm
RUN rpm2cpio pg_textsearch_${PG_MAJOR}-${TEXTSEARCH_VER}-1PGDG.f${FEDORA_VER}.${ARCH}.rpm | cpio -idmv

FROM fetcher AS fetcher-textsearch-llvmjit
ARG PG_MAJOR
ARG FEDORA_VER
ARG ARCH
ARG TEXTSEARCH_VER
ARG YUM_BASE

RUN curl -LO ${YUM_BASE}/pg_textsearch_${PG_MAJOR}-llvmjit-${TEXTSEARCH_VER}-1PGDG.f${FEDORA_VER}.${ARCH}.rpm
RUN rpm2cpio pg_textsearch_${PG_MAJOR}-llvmjit-${TEXTSEARCH_VER}-1PGDG.f${FEDORA_VER}.${ARCH}.rpm | cpio -idmv

FROM fetcher AS fetcher-vector
ARG PG_MAJOR
ARG FEDORA_VER
ARG ARCH
ARG VECTOR_VER
ARG YUM_BASE

RUN curl -LO ${YUM_BASE}/pgvector_${PG_MAJOR}-${VECTOR_VER}-1PGDG.f${FEDORA_VER}.${ARCH}.rpm
RUN rpm2cpio pgvector_${PG_MAJOR}-${VECTOR_VER}-1PGDG.f${FEDORA_VER}.${ARCH}.rpm | cpio -idmv

FROM ghcr.io/pgmq/pg${PG_MAJOR}-pgmq:v${PGMQ_VER} AS pgmq


FROM ${BASE_PG}
ARG PG_MAJOR

COPY --from=fetcher-textsearch /fetcher/usr/pgsql-${PG_MAJOR}/lib/pg_textsearch.so /usr/lib64/pgsql/
COPY --from=fetcher-textsearch /fetcher/usr/pgsql-${PG_MAJOR}/share/extension/* /usr/share/pgsql/extension/

COPY --from=fetcher-textsearch-llvmjit /fetcher/usr/pgsql-${PG_MAJOR}/lib/bitcode/* /usr/lib64/pgsql/bitcode

COPY --from=fetcher-vector /fetcher/usr/pgsql-${PG_MAJOR}/lib/vector.so /usr/lib64/pgsql/
COPY --from=fetcher-vector /fetcher/usr/pgsql-${PG_MAJOR}/share/extension/* /usr/share/pgsql/extension/

COPY --from=pgmq /usr/lib/postgresql/${PG_MAJOR}/lib/pg_partman_bgw.so /usr/lib64/pgsql/
COPY --from=pgmq /usr/lib/postgresql/${PG_MAJOR}/lib/bitcode/src/* /usr/lib64/pgsql/
COPY --from=pgmq /usr/share/postgresql/${PG_MAJOR}/extension/pg_partman* /usr/share/pgsql/extension/
COPY --from=pgmq /usr/share/postgresql/${PG_MAJOR}/extension/pgmq* /usr/share/pgsql/extension/

COPY <<EOF /usr/share/pgsql/postgresql.conf.sample
shared_preload_libraries = 'pg_stat_statements,pg_partman_bgw,pg_textsearch,vector'
pg_partman_bgw.dbname = 'postgres'
pg_partman_bgw.interval = 60
pg_partman_bgw.role = 'postgres'
EOF

COPY <<EOF /docker-entrypoint-initdb.d/01-init-pg_textsearch.sql
CREATE EXTENSION pg_textsearch;
EOF

COPY <<EOF /docker-entrypoint-initdb.d/02-init-pgvector.sql
CREATE EXTENSION vector;
EOF

COPY <<EOF /docker-entrypoint-initdb.d/03-init-pgmq.sql
CREATE EXTENSION pgmq;
CREATE EXTENSION pg_partman;
EOF
