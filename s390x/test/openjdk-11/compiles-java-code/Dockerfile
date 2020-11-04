FROM quay.io/ibmz/maven:3.6.3

RUN apt-get update && apt-get install -y ca-certificates-java && update-ca-certificates -f

COPY . /usr/src

WORKDIR /usr/src/test

CMD [ "mvn", "clean", "compile" ]
