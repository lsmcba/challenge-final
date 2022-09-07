FROM golang:1.16-alpine

WORKDIR /app

ADD . .

RUN go mod init helloworld
RUN go build -o /helloworld

EXPOSE 3002

CMD ["go", "run", "app.go"]
