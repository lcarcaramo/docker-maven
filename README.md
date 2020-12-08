# Tags
> _Built from [`quay.io/ibm/openjdk:11.0.8`](https://quay.io/repository/ibm/openjdk?tab=info)_
-	`3.6.3` - [![Build Status](https://travis-ci.com/lcarcaramo/docker-maven.svg?branch=master)](https://travis-ci.com/lcarcaramo/docker-maven)

### __[Original Source Code](https://github.com/carlossg/docker-maven)__

# Maven

[Apache Maven](http://maven.apache.org) is a software project management and comprehension tool.
Based on the concept of a project object model (POM),
Maven can manage a project's build,
reporting and documentation from a central piece of information.


# How to use this image

* Create a `Dockerfile` to build an image the builds your Maven project.

```
FROM quay.io/ibm/maven:3.6.3

COPY . /usr/src

WORKDIR /usr/src/<Your maven project folder>

CMD [ "mvn", "valid", "maven" "options" ]
```

* Build the image.

`docker build . --tag <custom maven image>`

* Run a container using the image that you just built to perform the Maven operation specified in the image on your Maven project.

`docker run --name <maven container> <custom maven image>`


# Multi-stage Builds

You can build your application with Maven and package it in an image that does not include Maven using [multi-stage builds](https://docs.docker.com/engine/userguide/eng-image/multistage-build/).

```
# build
FROM quay.io/ibm/maven:3.6.3
WORKDIR /usr/src/app
COPY pom.xml .
RUN mvn -B -e -C -T 1C org.apache.maven.plugins:maven-dependency-plugin:3.1.1:go-offline
COPY . .
RUN mvn -B -e -o -T 1C verify

# package without maven
FROM quay.io/ibm/openjdk:11.0.8
COPY --from=0 /usr/src/app/target/*.jar ./
```

# Reusing the Maven local repository

The local Maven repository can be reused across containers by creating a volume and mounting it in `/root/.m2`.

    docker volume create --name maven-repo
    docker run -it -v maven-repo:/root/.m2 quay.io/ibm/maven:3.6.3 mvn archetype:generate # will download artifacts
    docker run -it -v maven-repo:/root/.m2 quay.io/ibm/maven:3.6.3 mvn archetype:generate # will reuse downloaded artifacts


# Packaging a local repository with the image

The `$MAVEN_CONFIG` dir default to `/root/.m2` could be configured as a volume so anything copied there in a Dockerfile 
at build time is lost. For that reason the dir `/usr/share/maven/ref/` exists, and anything in that directory will be copied 
on container startup to `$MAVEN_CONFIG`.

To create a pre-packaged repository, create a `pom.xml` with the dependencies you need and use this in your `Dockerfile`.
`/usr/share/maven/ref/settings-docker.xml` is a settings file that 
changes the local repository to `/usr/share/maven/ref/repository`,
but you can use your own settings file as long as it uses `/usr/share/maven/ref/repository` 
as local repo.

    COPY pom.xml /tmp/pom.xml
    RUN mvn -B -f /tmp/pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve

To add your custom `settings.xml` file to the image use

    COPY settings.xml /usr/share/maven/ref/

For an example, check the `tests` dir


# Running as non-root

Maven needs the user home to download artifacts to, and if the user does not exist in the image an extra
`user.home` Java property needs to be set.

For example, to run as user `1000` mounting the host' Maven repo

    docker run -v maven-repo:/var/maven/.m2 -ti --rm -u 1000 -e MAVEN_CONFIG=/var/maven/.m2 quay.io/ibm/maven:3.6.3 mvn -Duser.home=/var/maven archetype:generate


# License

View [license information](https://www.apache.org/licenses/) for the software contained in this image.
