FROM openfaas/of-watchdog:0.7.2 as watchdog

FROM rust:1.44 as builder

WORKDIR /home/rust

# Copy all the sources
COPY function ./function
COPY main ./main

RUN cd main && cargo build --release

FROM ubuntu:latest as runner

RUN mkdir /root/.ssh

ARG ssh_priv_key
RUN echo "${ssh_priv_key}" >> /root/.ssh/id_rsa
ARG ssh_pub_key
RUN echo "${ssh_pub_key}" >> /root/.ssh/id_rsa.pub

# Install packages and add non-root user
RUN apt-get update && apt-get -y install curl ca-certificates openssl

# Copy of-watchdog binary
COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog
COPY --from=builder /home/rust/main/target/release/main /usr/bin/main
RUN chmod +x /usr/bin/main

ENV fprocess="main"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"
ENV JAGO_SOURCE="gitlab.com/jago-contributors/jago"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
