# API Gateway

Central entry point for the E-Commerce Microservices system.
Routes all incoming requests to the appropriate downstream service
using Spring Cloud Gateway and Eureka service discovery.

## Port
`8080`

## Routes

| Path Pattern        | Forwards To      |
|---------------------|------------------|
| `/api/users/**`     | user-service     |
| `/api/products/**`  | product-service  |
| `/api/orders/**`    | order-service    |

## Prerequisites
- `eureka-server` must be running on port `8761`

## Run
```bash
./mvnw spring-boot:run
```

## Health Check
```
GET http://localhost:8080/actuator/health
```