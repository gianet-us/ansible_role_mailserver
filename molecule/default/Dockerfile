FROM docker.io/library/debian:stable

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      python3 \
      systemd \
      systemd-sysv \
      dbus \
      ca-certificates && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

CMD ["/usr/sbin/init"]
