# API Gateway - Buses RED 🚌

Punto de entrada único para el sistema de microservicios de Buses RED. Implementa enrutamiento, autenticación Basic Auth y CORS mediante Spring Cloud Gateway.

## Tecnologías

- Java 21 (Eclipse Temurin)
- Spring Boot 3.5.7
- Spring Cloud Gateway 2024.0.0
- Spring Security (WebFlux)
- Maven 3.9.9
- Docker (multi-stage build)

## Arquitectura

```
Cliente (Postman)
       │
       ▼
┌──────────────────────────────────┐
│     API Gateway (:8080)          │
│  ┌────────────────────────────┐  │
│  │  SecurityConfig            │  │
│  │  (Basic Auth externalizado)│  │
│  ├────────────────────────────┤  │
│  │  CorsConfig                │  │
│  │  (Todos los orígenes)      │  │
│  ├────────────────────────────┤  │
│  │  Rutas Gateway             │  │
│  │  /api/location/**    →8081 │  │
│  │  /api/schedules/**   →8082 │  │
│  │  /api/consumer-location/** │  │
│  │                      →8083 │  │
│  │  /api/consumer-schedules/**│  │
│  │                      →8084 │  │
│  └────────────────────────────┘  │
└──────────────────────────────────┘
```

## Estructura del Proyecto

```
api-gateway-queue-buses-red/
├── src/main/java/com/busesred/gateway/
│   ├── ApiGatewayApplication.java
│   └── config/
│       ├── CorsConfig.java
│       └── SecurityConfig.java
├── src/main/resources/
│   └── application.yml
├── Dockerfile
└── pom.xml
```

## Tabla de Rutas

| Ruta Externa | Servicio Destino | Puerto |
|---|---|---|
| `/api/location/**` | producer-location-buses-red | 8081 |
| `/api/schedules/**` | producer-schedules-buses-red | 8082 |
| `/api/consumer-location/**` | consumer-location-buses-red | 8083 |
| `/api/consumer-schedules/**` | consumer-schedules-buses-red | 8084 |

> Todas las rutas aplican el filtro `StripPrefix=1` para remover `/api` antes de reenviar al servicio.

## Variables de Entorno

| Variable | Descripción | Ejemplo |
|---|---|---|
| `GATEWAY_USERNAME` | Usuario para Basic Auth | `admin` |
| `GATEWAY_PASSWORD` | Contraseña para Basic Auth | `admin123` |

## Seguridad

- **Basic Auth**: Credenciales externalizadas via variables de entorno (`@Value`)
- **Actuator**: Endpoints `/actuator/**` permitidos sin autenticación
- **CORS**: Todos los orígenes, métodos GET/POST/PUT/DELETE/OPTIONS
- **Cifrado**: BCryptPasswordEncoder para contraseñas

## Endpoints Expuestos

```http
# Health check (sin autenticación)
GET /actuator/health

# Rutas del gateway (con autenticación)
GET /actuator/gateway/routes
```

## Ejecución Local

```bash
# Compilar
mvn clean package -DskipTests

# Ejecutar
GATEWAY_USERNAME=admin GATEWAY_PASSWORD=admin123 \
java -jar target/api-gateway-queue-buses-red-1.0.0.jar
```

## Docker

```bash
# Construir imagen
docker build --no-cache --platform linux/amd64 -t api-gateway-queue-buses-red:latest .

# Ejecutar
docker run -p 8080:8080 \
  -e GATEWAY_USERNAME=admin \
  -e GATEWAY_PASSWORD=admin123 \
  api-gateway-queue-buses-red:latest
```

## Pruebas con Postman

```http
# Health check
GET http://<host>:8080/actuator/health

# Enviar ubicación (se reenvía al producer-location)
POST http://<host>:8080/api/location/send
Authorization: Basic admin:admin123
Content-Type: application/json
```
