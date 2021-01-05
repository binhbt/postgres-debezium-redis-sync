FROM maven:3.5.2-jdk-8-alpine AS maven_build

COPY pom.xml /tmp/

COPY src /tmp/src/

WORKDIR /tmp/

RUN mvn package

#pull base image

FROM openjdk:8-jdk-alpine

#maintainer 
MAINTAINER leo@gmail.com
#expose port 8082
EXPOSE 8082

#default command
CMD java -jar /data/debezium-redis-demo-1.0-SNAPSHOT.jar

#copy hello world to docker image from builder image

COPY --from=maven_build /tmp/target/debezium-redis-demo-1.0-SNAPSHOT.jar /data/debezium-redis-demo-1.0-SNAPSHOT.jar
