FROM golang:latest
RUN mkdir /app
WORKDIR /app
ADD . /app
RUN go build -o main .
EXPOSE 8888
CMD /app/main
