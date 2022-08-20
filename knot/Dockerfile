ARG BUILD_FROM

# Mainflux
FROM mainflux/auth:0.12.1 as mainflux-auth
FROM mainflux/things:0.12.1 as mainflux-things
FROM mainflux/users:0.12.1 as mainflux-users

FROM $BUILD_FROM

RUN apk add --no-cache \
    postgresql12 \
    postgresql12-contrib


RUN mkdir /run/postgresql && chown postgres:postgres /run/postgresql/ \
    && mkdir -p /run/postgresql/extensions \
	&& chown -R postgres:postgres /run/postgresql/extensions
RUN su postgres -c 'mkdir -p /var/lib/postgresql/data'
RUN su postgres -c 'chmod 0700 /var/lib/postgresql/data'
RUN su postgres -c 'initdb -D /var/lib/postgresql/data'

COPY --from=mainflux-auth /exe /usr/bin/auth
COPY --from=mainflux-things /exe /usr/bin/things
COPY --from=mainflux-users /exe /usr/bin/users

# Copy root filesystem
COPY rootfs /
RUN chmod +x /etc/services.d/mainflux/*
RUN chmod +x /etc/services.d/postgres/*
