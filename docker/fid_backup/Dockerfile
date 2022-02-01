ARG POSTGRES_VERSION=alpine
FROM postgres:${POSTGRES_VERSION}

RUN apk update \
    && apk upgrade \
    && apk add --no-cache bash \
    && apk add --no-cache pgbackrest \
    && rm -rf /var/cache/apk/*

RUN mkdir -p -m 770 /var/log/pgbackrest \
    && chown postgres:postgres /var/log/pgbackrest \
    && mkdir -p /etc/pgbackrest \
    && mkdir -p /etc/pgbackrest/conf.d \
    && touch /etc/pgbackrest/pgbackrest.conf \
    && chmod 640 /etc/pgbackrest/pgbackrest.conf \
    && chown postgres:postgres /etc/pgbackrest/pgbackrest.conf \
    && mkdir -p /var/lib/pgbackrest \
    && chmod 750 /var/lib/pgbackrest \
    && chown postgres:postgres /var/lib/pgbackrest

COPY ./postgresql.conf /var/lib/postgresql/data/postgresql.conf

CMD ["postgres"]

EXPOSE 5432