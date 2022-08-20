ARG BUILD_FROM
FROM golang:1.16-alpine AS babeltower-builder

ARG GOARCH
ARG GOARM

# Install build utilities
RUN apk --no-cache add --virtual .build-deps \
    bash \
    make \
    git

# Set the Current Working Directory inside the container
WORKDIR /go/src/github.com/CESARBR/

RUN git clone https://github.com/CESARBR/knot-babeltower.git

WORKDIR /go/src/github.com/CESARBR/knot-babeltower

RUN git checkout devel

# Don't use ca-certificates as .build-deps so that we use import certificates
# from builder, in the final image
RUN apk --no-cache add \
    ca-certificates \
    && update-ca-certificates


# Install project development tools and dependencies
RUN go get github.com/ahmetb/govvv

# Build the application
RUN make bin

# Remove build dependencies
RUN apk del .build-deps


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

COPY --from=babeltower-builder /go/src/github.com/CESARBR/knot-babeltower/app-linux-amd64 /usr/bin/babeltower
COPY --from=babeltower-builder /go/src/github.com/CESARBR/knot-babeltower/internal/ /etc/services.d/babeltower/internal
COPY --from=babeltower-builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy root filesystem
COPY rootfs /
RUN chmod +x /etc/services.d/mainflux/*
RUN chmod +x /etc/services.d/postgres/*
RUN chmod +x /etc/services.d/babeltower/*
