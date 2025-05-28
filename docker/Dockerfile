FROM golang:1.24 AS builder

ARG GITHUB_REPO=github.com/${{ values.owner }}/${{ values.name }}
ARG VERSION="dev"
ARG COMMIT="none"
ARG BUILD_UNIX_TIME=$(date +%s%M)

WORKDIR /app

COPY . .

RUN go mod download

RUN CGO_ENABLED=0 \
    go build \
    -ldflags " -s -w \
        -X ${GITHUB_REPO}/pkg/version.VERSION=${VERSION} \
        -X ${GITHUB_REPO}/pkg/version.COMMIT=${COMMIT} \
        -X ${GITHUB_REPO}/pkg/version.DATE=${BUILD_UNIX_TIME}" \
    -tags netgo,osusergo -a -o bin/${{ values.name }}

FROM scratch

COPY --from=builder /app/bin/${{ values.name }} /${{ values.name }}

EXPOSE 8080

CMD ["/${{ values.name }}", "server" ]
