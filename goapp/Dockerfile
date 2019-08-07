FROM golang:1.12.7-alpine3.10 as build

WORKDIR /go/src/app
COPY main.go main.go

RUN go build .

CMD ["app"]