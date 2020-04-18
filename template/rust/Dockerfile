FROM openfaas/of-watchdog:0.7.7 as watchdog

FROM rust:1.42 as builder

WORKDIR /home/rust

# Copy all the sources
COPY function ./function
COPY main ./main

RUN cd main && cargo build --release

FROM debian:bullseye-slim as runner

# Install packages and add non-root user
RUN apt update && apt install -y curl ca-certificates && \
    groupadd -r app && useradd -r -g app app

# Copy of-watchdog binary
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

# Copy main binary
COPY --from=builder /home/rust/main/target/release/main /usr/bin/main
RUN chmod +x /usr/bin/main

# Set up watchdog for HTTP mode
ENV fprocess="main"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

# Run as non root user
USER app

CMD ["fwatchdog"]
