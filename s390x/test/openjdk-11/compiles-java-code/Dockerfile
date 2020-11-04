FROM quay.io/ibmz/maven:3.6.3

RUN apt-get install -y ca-certificates

COPY . /usr/src

WORKDIR /usr/src/test

CMD [ "mvn", "-X", "clean", "compile" ]
