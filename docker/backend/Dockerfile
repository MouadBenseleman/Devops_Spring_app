# build stage
FROM maven:3.8.1-openjdk-17 AS build
WORKDIR /build
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

#run stage
FROM amazoncorretto:17

ARG PROFILE=dev
ARG APP_VERSION=1.0.3

WORKDIR /app

COPY --from=build /build/target/*.jar app.jar

EXPOSE 8088

ENV DB_URL=jdbc:postgresql://postgres-sql-bsn:5432/book_social_network
ENV JAR_VERSION=${APP_VERSION}
ENV ACTIVE_PROFILE=${PROFILE}

# Healthcheck via Spring Actuator (bonne pratique observabilité)
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://localhost:8088/actuator/health | grep -q 'UP' || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
CMD ["--spring.profiles.active=${ACTIVE_PROFILE}", "--spring.datasource.url=${DB_URL}"]
