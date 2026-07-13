# API Gateway

Central entry point for the E-Commerce Microservices system — routes all incoming client requests to the appropriate downstream service using Spring Cloud Gateway and Eureka-based service discovery. Built as part of an industrial-level Spring Boot microservices portfolio project.

![Java](https://img.shields.io/badge/Java-17-orange) ![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.15-brightgreen) ![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2024.0.1-blue) ![License](https://img.shields.io/badge/License-MIT-lightgrey)

## 📖 Overview

`api-gateway` is one of six microservices in a larger e-commerce system. It is the **sole entry point** for all client traffic — no service is directly exposed to the outside. It receives every incoming request, matches it against a route definition, and forwards it to the correct downstream service. Load balancing is handled automatically via Eureka service discovery — no hardcoded URLs.

> This service is part of a larger system. See the [main project README](#) for the full architecture and links to all repositories.

### Part of the E-Commerce Microservices Ecosystem

| Service | Responsibility | Port |
|---|---|---|
| eureka-server | Service discovery/registry | 8761 |
| **api-gateway** | **Single entry point, routing, load balancing** | **8080** |
| user-service | Authentication, JWT issuance, user management | 8081 |
| product-service | Product catalog, CRUD, filtering | 8082 |
| order-service | Orders, cart, calls product-service, user-service & payment-service via Feign | 8083 |
| payment-service | Simulated payment gateway flow | 8084 |

## 🏗️ Architecture & Design Decisions

Unlike every other service in the system, the API Gateway has **no controller, no service layer, and no repository**. All routing logic lives entirely in `application.yml`.

```
Client Request
      ↓
[ API Gateway :8080 ]
      ↓  (route match + StripPrefix filter)
      ├──▶ user-service    :8081
      ├──▶ product-service :8082
      ├──▶ order-service   :8083
      └──▶ payment-service :8084
```

Key design decisions and the reasoning behind them:

| Decision | Reasoning |
|---|---|
| **Spring Cloud Gateway (WebFlux/reactive)** | Non-blocking, reactive stack — handles high concurrency with minimal thread overhead. Classic Spring MVC (servlet-based) cannot be used alongside Gateway; they conflict at the runtime level. |
| **`lb://` URI scheme for all routes** | Instead of hardcoding `http://localhost:8082`, `lb://product-service` tells the load balancer to resolve the service name via Eureka at request time. If a service restarts or scales, the gateway adapts automatically. |
| **`StripPrefix=2` on all routes** | Client sends `/api/products/v1/...`. The gateway strips the first two segments (`/api/products`), forwarding `/v1/...` to `product-service` — which is exactly the path the service's controllers expect. The prefix count must match the number of segments being stripped exactly; an off-by-one causes silent routing failures. |
| **No JWT validation at the gateway level** | Each downstream service authenticates its own requests independently. This means services remain secure even if accessed directly, bypassing the gateway — consistent with the security model used across the whole system. |
| **Eureka discovery locator enabled** | In addition to the explicit route definitions, the gateway can also automatically route to any registered Eureka service by name. Explicit routes take precedence and give full control over path transformation. |

## 🛠️ Tech Stack

- Java 17
- Spring Boot 3.5.15
- Spring Cloud 2024.0.1 (Spring Cloud Gateway — WebFlux, Eureka Client)
- Maven

## 📁 Project Structure

```
api-gateway/
├── src/main/java/com/ecommerce/apigateway/
│   └── ApiGatewayApplication.java       # Entry point — no additional classes needed
└── src/main/resources/
    └── application.yml                  # All routing logic lives here
```

> No controllers, services, or repositories — the gateway is pure configuration.

## 🔌 Route Table

All client requests go to `http://localhost:8080`. The gateway matches the path and forwards to the correct service.

| Client Sends | Forwards To | Downstream Service |
|---|---|---|
| `/api/users/v1/**` | `/v1/**` | user-service `:8081` |
| `/api/products/v1/**` | `/v1/**` | product-service `:8082` |
| `/api/orders/v1/**` | `/v1/**` | order-service `:8083` |
| `/api/payments/v1/**` | `/v1/**` | payment-service `:8084` |

### How `StripPrefix=2` Works

```
Client:   GET /api/products/v1/all
Gateway strips:  /api/products          ← 2 segments removed
Forwards: GET /v1/all  →  product-service
```

## ⚙️ Setup & Installation

### Prerequisites

- Java 17+
- Maven 3.8+
- `eureka-server` must be running on port `8761` before starting this service

### 1. Clone the repository

```bash
git clone https://github.com/zainmustafa205/api-gateway.git
cd api-gateway
```

### 2. Run the application

No database, no environment variables required — this service has no persistence layer.

```bash
mvn spring-boot:run
```

The gateway starts on port `8080` and registers itself with Eureka at `http://localhost:8761`.

### 3. Verify registration

Visit `http://localhost:8761` and confirm `API-GATEWAY` appears in the list of registered instances with status `UP`.

### 4. Health check

```
GET http://localhost:8080/actuator/health
```

## 🔒 Configuration Notes

- `eureka-server` must be running **before** the gateway starts — otherwise Eureka registration fails and `lb://` route resolution will not work.
- `StripPrefix` value must **exactly match** the number of path segments being stripped. A mismatch causes the downstream service to receive an incorrect path and return `404` with no obvious error at the gateway level.
- JWT validation is intentionally **not** handled here — each service in the system validates its own tokens independently.
- The gateway uses the **reactive WebFlux stack** — do not add `spring-boot-starter-web` to the dependencies; it will cause a startup conflict.

## 🗺️ Roadmap

- [ ] Add JWT validation filter at gateway level (centralized auth)
- [ ] Add rate limiting per IP / per user
- [ ] Add request/response logging filter
- [ ] Add Caffeine cache for load balancer (production recommendation)

## 📄 License

This project is part of a personal portfolio and is available under the MIT License.

## 🔗 Related Repositories

- [eureka-server](https://github.com/zainmustafa205/eureka-server)
- [user-service](https://github.com/zainmustafa205/user-service)
- [product-service](https://github.com/zainmustafa205/product-service)
- [order-service](https://github.com/zainmustafa205/order-service)
- [payment-service](https://github.com/zainmustafa205/payment-service)
