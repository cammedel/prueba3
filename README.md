## Instrucciones para inicializar el proyecto

Este proyecto es un microservicio construido con Spring Boot y Java.

## Requisitos previos

- GitHub
- Docker y Docker Compose
- Maven

## Pasos iniciales

1. Clonar el repositorio con `git clone <url>`.
2. Entrar en la carpeta del proyecto con `cd <nombre-proyecto>`.

## Despliegue local

Se recomienda utilizar Docker para el despliegue, por lo que es necesario tener Docker Desktop instalado.

1. (Opcional) Crear la carpeta `deployment` para los artefactos de Docker.
2. Crear los archivos de despliegue (`Dockerfile`, `docker-compose.yml`).
3. Comandos habituales:
   - `docker compose up --build -d`
   - `docker compose up`
   - `docker compose ps`

## Pruebas automatizadas con pipeline

El pipeline de GitHub Actions automatiza la calidad y la trazabilidad del microservicio.

1. Jacoco ejecuta las pruebas unitarias. En el `pom.xml` esta configurado el plugin necesario. Comandos: `mvn test`, `mvn verify`. El reporte puede abrirse en `target/site/jacoco/index.html`.
2. SonarCloud verifica la calidad del codigo usando los resultados de Jacoco. Se configuro con una cuenta en SonarCloud y un token almacenado como secreto (`SONAR_TOKEN`).
3. Snyk analiza la seguridad de las dependencias. Requiere descargar las dependencias, crear una cuenta y guardar el token (`SNYK_TOKEN`) como secreto en GitHub.

## Integracion en CI/CD y toma de decisiones (IE4)

El workflow `.github/workflows/main.yml` corre en cada push o pull request e integra pruebas, seguridad y despliegue:

- `mvn clean verify` valida la calidad y genera los reportes que usa Jacoco/Sonar. Si las pruebas fallan, el job termina y no se continua.
- SonarCloud aplica las Quality Gates. Cuando se incumplen umbrales de cobertura o se detectan code smells criticos, el paso falla y bloquea el merge, guiando decisiones sobre deuda tecnica.
- Snyk usa `SNYK_TOKEN` para autenticarse, escanea dependencias Maven y adjunta un reporte JSON. Vulnerabilidades de severidad alta devuelven un exit code distinto de cero y detienen el pipeline hasta que se actualicen dependencias o se justifique una excepcion.
- Solo si todo lo anterior pasa se construye la imagen Docker (`docker build -t prueba2-app -f deployment/Dockerfile .`) y se levanta el stack en `deployment/` con `docker compose up -d`. Un error en el arranque corta el pipeline y evita liberar versiones inestables.

### Evidencia de detencion ante fallas criticas

- Si una prueba unitaria falla o hay errores de compilacion, `mvn clean verify` devuelve error y los pasos posteriores no se ejecutan. El pipeline se marca como `failed`, demostrando el control de calidad.
- Si Snyk detecta vulnerabilidades de severidad alta, el comando `snyk test` falla. Al no usar `continue-on-error`, el workflow se detiene antes del despliegue, cumpliendo el requisito de seguridad.
- Si `docker compose up -d` no puede levantar los contenedores (por puertos ocupados o healthchecks fallidos), el paso termina en error y el job se corta, dejando registro para decidir la correccion.

Estas herramientas aportan metricas y bloqueos automaticos que informan decisiones tecnicas sobre la aceptacion de un cambio o su despliegue (criterio IE4).

## Trazabilidad

La trazabilidad se garantiza al vincular el codigo con el despliegue en Docker mediante el pipeline descrito.
