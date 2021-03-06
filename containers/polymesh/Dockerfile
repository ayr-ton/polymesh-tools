ARG golangbase=1.15
FROM golang:${golangbase} as gobuild

ADD src/health-check/ /opt/health-check/
ADD src/rotate-keys/ /opt/rotate-keys/

WORKDIR /opt/health-check
RUN ls -a && \
    go build && \
    chmod 0755 /opt/health-check/polymesh-health-check
WORKDIR /opt/rotate-keys
RUN ls -a && \
    go build && \
    chmod 0755 /opt/rotate-keys/polymesh-rotate-keys


FROM debian:buster as releasefetch

ADD src/fetch.sh /opt/fetch/fetch.sh
WORKDIR /opt/fetch
RUN bash /opt/fetch/fetch.sh


FROM gcr.io/distroless/cc
USER 4001:4001

COPY --chown=4002:4002 --from=gobuild      /opt/health-check/polymesh-health-check /usr/local/bin/check
COPY --chown=4002:4002 --from=gobuild      /opt/rotate-keys/polymesh-rotate-keys   /usr/local/bin/rotate
COPY --chown=4001:4001 --from=releasefetch /opt/files/                    /

ENTRYPOINT ["/usr/local/bin/polymesh"]
CMD [ "-d", "/var/lib/polymesh" ]

HEALTHCHECK \
    --interval=10s \
    --start-period=120s \
    --timeout=5s \
    --retries=6 \
    CMD /usr/local/bin/check liveness

