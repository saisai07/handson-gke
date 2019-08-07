FROM golang:1.12.7-alpine3.10

WORKDIR $GOPATH/src/gcpug
COPY main.go main.go
RUN go build . && \
    cp gcpug /usr/local/bin/gcpug

CMD ["gcpug"]