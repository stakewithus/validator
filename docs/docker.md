# Dockerfile to AWS ECR

## Docker
 Most of our projects are complied using docker using either alpine/debian. This helps to better debug and seperate common services such as different golang versions that is running under the same machine.


An example of a dockerfile that we used for our project:

```
FROM golang:1.18-alpine AS build-env

# Set up dependencies
ENV PACKAGES curl make git libc-dev bash gcc linux-headers eudev-dev

RUN apk add --no-cache $PACKAGES

WORKDIR /root

RUN git clone https://github.com/CosmosContracts/Juno.git

WORKDIR /root/Juno

RUN git checkout v9.0.0

ADD https://github.com/CosmWasm/wasmvm/releases/download/v1.0.0/libwasmvm_muslc.x86_64.a /lib/libwasmvm_muslc.x86_64.a
RUN sha256sum /lib/libwasmvm_muslc.x86_64.a | grep f6282df732a13dec836cda1f399dd874b1e3163504dbd9607c6af915b2740479
RUN cp /lib/libwasmvm_muslc.x86_64.a /lib/libwasmvm_muslc.a

RUN LEDGER_ENABLED=false BUILD_TAGS=muslc LINK_STATICALLY=true make build

FROM alpine:edge
# Install ca-certificates
RUN apk add --update ca-certificates

WORKDIR /root

# Copy over binaries from the build-env
COPY --from=build-env /root/Juno/bin/junod /usr/bin/junod
ENTRYPOINT ["junod"]
```

## AWS ECR

Storage of the newly complied docker image for each projects will be pushed and stored in AWS ECR(Amazon Elastic Container Registry) which enables our team to distribute it effectively to the relevant servers