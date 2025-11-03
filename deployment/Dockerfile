FROM maven:3.9.9-eclipse-temurin-17 AS build
WORKDIR /app

# Copiamos primero el descriptor para aprovechar la cache de Maven
COPY pom.xml ./
RUN mvn -B dependency:go-offline

# Copiamos el código fuente y generamos el artefacto
COPY src ./src
RUN mvn -B -DskipTests clean package

FROM eclipse-temurin:17-jre AS runtime
WORKDIR /app

# Copiamos el artefacto construido al contenedor final
COPY --from=build /app/target/*-SNAPSHOT.jar app.jar

EXPOSE 8080
ENV JAVA_OPTS=""

ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} -jar /app/app.jar"]
