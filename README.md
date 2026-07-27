# Eureka Service Registry

![Java](https://img.shields.io/badge/Java-17-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.4.2-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2024.0.1-6DB33F?style=for-the-badge&logo=springcloud&logoColor=white)
![Netflix Eureka](https://img.shields.io/badge/Netflix%20Eureka-Server-0193E5?style=for-the-badge&logo=netflix&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-3.x-C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)
![Build](https://img.shields.io/badge/Build-Passing-brightgreen?style=for-the-badge)
![PRs](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=for-the-badge)

---

## What is this?

**Eureka Service Registry** is the **service discovery backbone** of the [Payment Integration System](https://github.com/chandan-howale/payment-integration-system). It allows microservices to register themselves, discover each other, and communicate dynamically without hardcoded URLs.

Built with **Spring Boot 3.4.2** and **Spring Cloud Netflix Eureka**, this service runs on port `8761` and provides a dashboard to monitor all registered microservices in real time.

---

## Why Eureka?

In a microservices architecture, services are deployed across multiple instances and their locations can change. **Eureka solves this** by acting as a central phone book:

```
Without Eureka (Hardcoded):                            With Eureka (Dynamic):

Service A ──▶ Service B (192.168.1.5:8081)             Service A ──▶ Eureka: "Where is B?"
             ^ Hard to maintain                                       Eureka: "B is at 192.168.1.5:8081"
             ^ Breaks on redeploy                                     Service A ──▶ Service B
```

---

## System Architecture

```
+-------------------------------------------------------------------+
|                  PAYMENT INTEGRATION SYSTEM                       |
|                                                                   |
|               +-------------------------+                         |
|               |     EUREKA SERVER       |◀────── YOU ARE HERE    |
|               |   (Service Registry)    |                         |
|               |     Port: 8761          |                         |
|               +------------+------------+                         |
|                            |                                      |
|              +-------------+-------------+                        |
|              |                           |                        |
|   +----------v----------+   +------------v---------+              |
|   |  Payment Processing |   |  PayPal Provider     |              |
|   |  Service            |   |  Service             |              |
|   |  Port: 8081         |   |  Port: 8082          |              |
|   +---------------------+   +----------------------+              |
|                                                                   |
+-------------------------------------------------------------------+
```

### How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                     EUREKA WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. REGISTRATION                                                │
│     Payment Service ──────▶ Eureka Server                      │
│     "Hi, I'm Payment Service at :8081"                          │
│     Eureka: "Registered! I'll remember you"                     │
│                                                                 │
│  2. HEARTBEAT (Keep Alive)                                      │
│     Payment Service ~~~~~▶ Eureka Server                       │
│     "Still here!"                                               │
│     Eureka: "Got it!"                                           │
│                                                                 │
│  3. DISCOVERY                                                   │
│     PayPal Service ──▶ Eureka ──▶ "Where is Payment Service?"  │
│     Eureka: "At 192.168.1.5:8081"                               │
│     PayPal Service ────────────▶ Payment Service               │
│                                                                 │
│  4. FAILOVER                                                    │
│     If a service stops heartbeating → Eureka removes it         │
│     If network issue → Self-Preservation mode protects registry │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Java | 17 | Runtime environment |
| Spring Boot | 3.4.2 | Application framework |
| Spring Cloud | 2024.0.1 | Microservices infrastructure |
| Netflix Eureka Server | - | Service discovery & registry |
| Maven | 3.x | Build & dependency management |
| Lombok | - | Boilerplate code reduction |
| Gson | - | JSON serialization |
| ModelMapper | 3.2.1 | Object mapping |
| JUnit 5 | 5.8.1 | Unit testing |

---

## Prerequisites

Before running this project, make sure you have:

- **Java 17** or higher installed
  ```bash
  java -version
  ```
- **Maven 3.x** installed (or use the included Maven Wrapper)
  ```bash
  mvn -version
  ```
- **Git** for cloning the repository

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/chandan-howale/eureka-service-registry.git
cd eureka-service-registry
```

### 2. Build the Project

```bash
# Build with Maven Wrapper (recommended)
./mvnw clean install

# Or use system Maven
mvn clean install
```

### 3. Run the Service

1. Import the project as a Maven project in your IDE (IntelliJ / Eclipse / VS Code)
2. Run `EurekaServiceRegistryApplication.java` as a Spring Boot application
3. The default profile (`local`) will be active automatically

### 4. Verify It's Running

- **Eureka Dashboard**: Open [http://localhost:8761](http://localhost:8761) in your browser
- **Health Check**: `curl http://localhost:8761/actuator/health` (if actuator is enabled)

You should see the Eureka Dashboard showing registered instances:

```
System Status
─────────────────────────────────
Environment:  local
Current time: 2024-07-24T12:00:00+05:30
Uptime:       1 minute
```

---

## API Endpoints

### Test Endpoint

| Method | Endpoint | Description | Parameters |
|---|---|---|---|
| POST | /add | Adds two numbers | num1 (int), num2 (int) |

#### Example Request

```bash
# Using cURL
curl -X POST "http://localhost:8761/add?num1=5&num2=3"
```

#### Example Response

```
8
```

> **Note:** This is a sample endpoint for testing. In production, the Eureka Server primarily serves the dashboard and service registry APIs.

---

## Configuration Guide

### Application Profiles

### What is Self-Preservation Mode?

Eureka's **Self-Preservation** mode protects the service registry during network partitions. When enabled, if Eureka stops receiving heartbeats from services (due to a network issue, not actual failure), it **does NOT remove** those services from the registry. This prevents mass deregistration of healthy services just because of a temporary network glitch.

| Profile | Self-Preservation | Why |
|---|---|---|
| local | Disabled | Services restart often during development; keeping stale entries causes confusion |
| dev | Disabled | Same as local — fast iteration needs fresh registry |
| qa | Enabled | Closer to production; test with real behavior |
| uat | Enabled | Pre-production environment; must match production behavior |
| prod | Enabled | Critical — prevents mass deregistration during network issues |

> **Rule of thumb**: Disabled in development for convenience, enabled in production for safety.

### Core Configuration (`application.properties`)

```properties
# Server port (standard Eureka port)
server.port=8761

# Application name (displayed in Eureka Dashboard)
spring.application.name=eureka-service-registry

# Don't register THIS server with itself
eureka.client.register-with-eureka=false

# Don't fetch registry from itself
eureka.client.fetch-registry=false
```

### Local Profile (`application-local.properties`)

```properties
# Start immediately with empty registry
eureka.server.wait-time-in-ms-when-sync-empty=0

# Instance evicted if no heartbeat in 10 seconds
eureka.instance.lease-expiration-duration-in-seconds=10

# Check for dead instances every 5 seconds
eureka.server.eviction-interval-timer-in-ms=5000

# Disable self-preservation (safe for development)
eureka.server.enable-self-preservation=false
```

### Eureka Configuration Explained

| Property | Value | Description |
|----------|-------|-------------|
| `server.port` | `8761` | Standard Eureka Server port |
| `eureka.client.register-with-eureka` | `false` | Prevents server from registering with itself |
| `eureka.client.fetch-registry` | `false` | Prevents server from fetching its own registry |
| `eureka.server.wait-time-in-ms-when-sync-empty` | `0` | Start immediately without waiting for peers |
| `eureka.instance.lease-expiration-duration-in-seconds` | `10` | Time before considering a service dead |
| `eureka.server.eviction-interval-timer-in-ms` | `5000` | How often to check for expired instances |
| `eureka.server.enable-self-preservation` | `false` | Disable in dev, enable in production |

---

## Project Structure

```
eureka-service-registry/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/chandan/payments/
│   │   │       ├── EurekaServiceRegistryApplication.java  # Main application
│   │   │       └── controller/
│   │   │           └── AdditionController.java            # Sample endpoint
│   │   └── resources/
│   │       ├── application.properties                     # Main config
│   │       ├── application-local.properties               # Local dev config
│   │       ├── application-dev.properties                 # Dev environment
│   │       ├── application-qa.properties                  # QA environment
│   │       ├── application-uat.properties                 # UAT environment
│   │       └── application-prod.properties                # Production config
│   └── test/
│       └── java/
│           └── com/chandan/payments/
│               └── EurekaServiceRegistryApplicationTests.java
|
├── pom.xml                  # Maven configuration
├── mvnw                     # Maven Wrapper (Unix)
├── mvnw.cmd                 # Maven Wrapper (Windows)
├── .gitignore               # Git ignore rules
└── README.md                # This file
```

---

## Related Repositories

This Eureka Service Registry is part of the **Payment Integration System**:

Main project repo : [payment-integration-system](https://github.com/chandan-howale/payment-integration-system)

| Repository | Description | Status |
|------------|-------------|--------|
| [eureka-service-registry](https://github.com/chandan-howale/eureka-service-registry) | Service discovery & registry | Active |
| [paypal-provider-service](https://github.com/chandan-howale/paypal-provider-service) | PayPal payment integration provider | Active |
| [payment-processing-service](https://github.com/chandan-howale/payment-processing-service) | Core payment processing logic | Active |

### Service Communication Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                  PAYMENT PROCESSING FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Client Request                                                 │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────────┐     ┌──────────────────┐                   │
│  │    Payment      │     │     Eureka       │                   │
│  │    Processing   │────▶│     Server      │                   │
│  │    Service      │     │     :8761        │                   │
│  └────────┬────────┘     └──────────────────┘                   │
│           │                                                     │
│           │  "Where is PayPal Provider?"                        │
│           │──────────────────────────────▶                     │
│           │                                                     │
│           │  "PayPal Provider is at :8082"                      │
│           │◀──────────────────────────────                     │
│           │                                                     │
│           ▼                                                     │
│  ┌─────────────────┐                                            │
│  │    PayPal       │                                            │
│  │    Provider     │                                            │
│  │    Service      │                                            │
│  │    :8082        │                                            │
│  └─────────────────┘                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Deployment

### Production Recommendations

1. **Enable Self-Preservation**: Set `eureka.server.enable-self-preservation=true` in production
2. **Run Multiple Instances**: Deploy 2+ Eureka servers for high availability
3. **Configure Peers**: Enable peer replication for production clusters
4. **Use HTTPS**: Configure SSL for secure communication
5. **Monitor Dashboard**: Keep the Eureka Dashboard accessible for ops team

### Sample Production Configuration

```properties
# Production settings (application-prod.properties)
eureka.server.enable-self-preservation=true
eureka.instance.lease-expiration-duration-in-seconds=30
eureka.server.eviction-interval-timer-in-ms=60000
```

---

## Common Issues & Troubleshooting

### Issue: Port Already in Use

```bash
# Find and kill the process using port 8761
netstat -ano | findstr :8761
taskkill /PID <process-id> /F
```

### Issue: Services Not Registering

1. Ensure Eureka Server is running on port 8761
2. Check `eureka.client.service-url.defaultZone=http://localhost:8761/eureka/` in the client service
3. Verify `register-with-eureka=true` in the client service (not this server)

### Issue: Eureka Dashboard Shows No Services

1. Check if the client service has started successfully
2. Verify the client is pointing to the correct Eureka URL
3. Wait 30 seconds for registration to complete

---

## Support

If you find this project helpful, please give it a star on GitHub!

---

**Built with using Spring Boot & Spring Cloud Netflix Eureka**
