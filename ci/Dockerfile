FROM alpine

RUN apk --update upgrade && \
    apk add --update ca-certificates && \
    update-ca-certificates && \
    rm -rf /var/cache/apk/*

RUN apk add --no-cache zip &&\
    apk add --no-cache python && \
    apk add --no-cache --virtual=build-dependencies wget ca-certificates && \
    wget "https://bootstrap.pypa.io/get-pip.py" -O /dev/stdout | python && \
    apk del build-dependencies

RUN pip install mkdocs
RUN apk update && apk upgrade && \
    apk add --no-cache git && \
    rm -rf /var/cache/apk/*
