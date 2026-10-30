# Dockerfile initally created from
# https://ktor.io/docs/docker.html#-sbmfeb_140

# Stage 1: Cache Gradle dependencies
FROM docker.io/library/gradle:9-jdk21-alpine AS cache
RUN mkdir -p /home/gradle/cache_home
ENV GRADLE_USER_HOME=/home/gradle/cache_home
COPY ./src/build.gradle.* ./src/gradle.properties /home/gradle/app/
COPY ./src/gradle /home/gradle/app/gradle
WORKDIR /home/gradle/app
RUN gradle dependencies --no-daemon

# Stage 2: Build Application
FROM docker.io/library/gradle:9-jdk21-alpine AS build
COPY --from=cache /home/gradle/cache_home /home/gradle/.gradle
COPY --chown=gradle:gradle ./src /home/gradle/src
WORKDIR /home/gradle/src
# Build the fat JAR, Gradle also supports shadow
# and boot JAR by default.
RUN gradle build -x test --no-daemon

# Stage 3: Create the Runtime Image
FROM docker.io/bellsoft/liberica-runtime-container:jre-25-slim-musl AS runtime
EXPOSE 8080
RUN mkdir /app
COPY --from=build /home/gradle/src/build/libs/*-all.jar /app/askkt-all.jar
ENTRYPOINT ["java","-jar","/app/askkt-all.jar", "migration:executeAndStart"]