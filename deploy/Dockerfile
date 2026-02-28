FROM golang:1-alpine AS builder

COPY . /go/src/github.com/finb/bark-server

WORKDIR /go/src/github.com/finb/bark-server

RUN set -ex \
    && apk add git gcc libc-dev \
    && BUILD_VERSION=$(git describe --tags 2>/dev/null || echo "unknown") \
    && BUILD_DATE=$(date "+%F %T") \
    && COMMIT_SHA1=$(git rev-parse HEAD 2>/dev/null || echo "unknown") \
    && go install -trimpath -ldflags \
        "-X 'main.version=${BUILD_VERSION}' \
        -X 'main.buildDate=${BUILD_DATE}' \
        -X 'main.commitID=${COMMIT_SHA1}' \
        -w -s"

FROM alpine

LABEL maintainer="mritd <mritd@linux.com>"

RUN echo 'hosts: files dns' > /etc/nsswitch.conf

ARG TIMEZONE=Asia/Shanghai

ENV TZ=${TIMEZONE}

RUN set -ex \
    && apk upgrade \
    && apk add --no-cache bash ca-certificates tzdata \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone

COPY --from=builder /go/bin/bark-server /usr/local/bin/bark-server
COPY --from=builder /go/src/github.com/finb/bark-server/deploy/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 8080

RUN mkdir -p /data

CMD ["/usr/local/bin/bark-server", "-addr", "0.0.0.0:8080", "-data", "/data"]
