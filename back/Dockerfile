# syntax=docker/dockerfile:1.7

FROM gradle:8.7-jdk17-alpine AS build
WORKDIR /src
COPY --chown=gradle:gradle . .
RUN gradle build --no-daemon -x test

FROM eclipse-temurin:17-jre-alpine
RUN addgroup -S app && adduser -S app -G app \
 && apk add --no-cache wget
WORKDIR /app
COPY --from=build /src/build/libs/microcrm-0.0.1-SNAPSHOT.jar app.jar
RUN chown -R app:app /app
USER app
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD wget -qO- http://localhost:8080/persons > /dev/null || exit 1
ENTRYPOINT ["java","-jar","/app/app.jar"]
