ARG BUILD_FROM
ARG BUILD_ARCH
FROM golang:1.16-alpine AS babeltower-builder

# Install build utilities
RUN apk --no-cache add --virtual .build-deps \
    bash \
    make \
    git

WORKDIR /go/src/github.com/

# Clone and build mainflux services
ENV MAINFLUX_VERSION=0.12.1
RUN mkdir -p /go/src/github.com/mainflux
RUN wget -q -O- https://github.com/mainflux/mainflux/archive/$MAINFLUX_VERSION.tar.gz|tar xz -C /go/src/github.com/mainflux --strip-components=1


FROM babeltower-builder AS builder-armhf
ENV GOARCH=arm
ENV GOARM=6
FROM babeltower-builder AS builder-armv7
ENV GOARCH=arm
ENV GOARM=7
FROM babeltower-builder AS builder-aarch64
ENV GOARCH=arm64
FROM babeltower-builder AS builder-amd64
ENV GOARCH=amd64
FROM babeltower-builder AS builder-i386
ENV GOARCH=386

FROM builder-$BUILD_ARCH AS builder
WORKDIR /go/src/github.com/mainflux
RUN make auth && make users && make things

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
RUN make GOARCH=$GOARCH GOARM=$GOARM bin
RUN mv /go/src/github.com/CESARBR/knot-babeltower/app-linux-$GOARCH /go/src/github.com/CESARBR/knot-babeltower/app

# Remove build dependencies
RUN apk del .build-deps

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

COPY --from=builder /go/src/github.com/mainflux/build/mainflux-auth /usr/bin/auth
COPY --from=builder /go/src/github.com/mainflux/build/mainflux-things /usr/bin/things
COPY --from=builder /go/src/github.com/mainflux/build/mainflux-users /usr/bin/users

COPY --from=builder /go/src/github.com/CESARBR/knot-babeltower/app /usr/bin/babeltower
COPY --from=builder /go/src/github.com/CESARBR/knot-babeltower/internal/ /etc/services.d/babeltower/internal
COPY --from=babeltower-builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

ENV MF_AUTH_HTTP_PORT=8183
ENV MF_AUTH_GRPC_PORT=8181
ENV MF_USERS_HTTP_PORT=8180
ENV MF_THINGS_HTTP_PORT=8182

# Copy root filesystem
COPY rootfs /
RUN chmod +x /etc/services.d/mf-*/*
RUN chmod +x /etc/services.d/babeltower/*
RUN chmod +x /etc/cont-init.d/*
