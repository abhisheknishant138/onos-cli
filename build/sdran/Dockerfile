ARG ONOS_BUILD_VERSION=undefined

FROM onosproject/golang-build:$ONOS_BUILD_VERSION AS build
ENV GO111MODULE=on
RUN go get github.com/opennetworkinglab/gnmi/cmd/gnmi_cli && \
    go get github.com/atomix/cli/cmd/atomix
COPY . /go/src/github.com/onosproject/onos-cli
RUN cd /go/src/github.com/onosproject/onos-cli && GOFLAGS=-mod=vendor make build-sdran

FROM alpine:3.11
RUN apk add bash bash-completion curl libc6-compat

RUN addgroup -S onos && adduser -S -G onos onos --home /home/onos
USER onos
WORKDIR /home/onos

COPY --from=build /go/src/github.com/onosproject/onos-cli/build/_output/sdran /usr/local/bin/sdran
COPY --from=build /go/bin/atomix /usr/local/bin/atomix
COPY --from=build /go/bin/gnmi_cli /usr/local/bin/gnmi_cli
COPY --from=build /go/src/github.com/onosproject/onos-cli/pkg/certs/* /etc/ssl/certs/

RUN mkdir /home/onos/.onos && \
    cp /etc/profile /home/onos/.bashrc && \
    sdran completion bash > /home/onos/.onos/bash_completion.sh && \
    echo "source /home/onos/.onos/bash_completion.sh" >> /home/onos/.bashrc && \
    mkdir /home/onos/.atomix && \
    atomix completion bash > /home/onos/.atomix/bash_completion.sh && \
    echo "source /home/onos/.atomix/bash_completion.sh" >> /home/onos/.bashrc && \
    echo "eval \$(resize)" >> /home/onos/.bashrc

ENTRYPOINT ["/bin/bash"]
