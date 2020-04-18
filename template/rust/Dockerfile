FROM openfaas/of-watchdog:0.7.7 as watchdog

FROM rust:1.42-alpine as builder

WORKDIR /home/rust

# Copy all the sources
COPY function ./function
COPY main ./main

RUN cd main && export RUSTFLAGS="-C target-feature=-crt-static" && cargo build --target x86_64-unknown-linux-musl --release

FROM alpine:3 as runner

# Install packages and add non-root user
RUN apk --no-cache add curl ca-certificates \
    && addgroup -S app && adduser -S -g app app

ENV USER=app

# Copy of-watchdog binary
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

# Copy main binary
COPY --from=watchdog /home/rust/target/x86_64-unknown-linux-musl/release/main /usr/bin/main
RUN chmod +x /usr/bin/main

# Set up watchdog for HTTP mode
ENV fprocess="main"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
