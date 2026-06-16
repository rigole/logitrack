# 🚚 LogiTrack — Real-Time Fleet & Supply Chain Intelligence Platform

> *Tracking 5,000 trucks. Every ping. Every mile. Every second.*

[![Java](https://img.shields.io/badge/Java-17+-orange?style=flat-square&logo=openjdk)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen?style=flat-square&logo=springboot)](https://spring.io/projects/spring-boot)
[![Angular](https://img.shields.io/badge/Angular-17+-red?style=flat-square&logo=angular)](https://angular.io/)
[![Apache Kafka](https://img.shields.io/badge/Kafka-3.x-black?style=flat-square&logo=apachekafka)](https://kafka.apache.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-blue?style=flat-square&logo=postgresql)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)](https://www.docker.com/)

---

## 🌍 What Is This?

**LogiTrack** is a production-grade, event-driven fleet management system inspired by the real-time tracking engines behind DHL, Amazon Logistics, and UPS.

It simulates a fleet of **5,000 delivery trucks** — each broadcasting their **GPS coordinates**, **speed**, and **engine temperature** every **3 seconds** — and processes that firehose of data in real time. Think smart geofencing, instant anomaly detection, and a live dashboard that never sleeps.

Whether you're a backend engineer exploring microservices at scale, a data engineer curious about Kafka pipelines, or a team building your own logistics product — this repo is your blueprint.

---

## ⚡ Key Features

- 📡 **High-frequency telemetry ingestion** — handles 5,000 concurrent truck pings every 3 seconds (~1.6M events/min)
- 🗺️ **Real-time geofencing** — detect when vehicles enter or exit defined zones (warehouses, cities, restricted areas)
- 🌡️ **Engine anomaly alerts** — flag overheating or overspeed events as they happen
- 🧭 **Live fleet dashboard** — Angular UI with real-time map updates
- 🔀 **Event-driven architecture** — fully decoupled services communicating via Kafka
- 🔍 **Service discovery & config management** — Spring Cloud Eureka + Config Server
- 🐳 **One-command local setup** — Docker Compose spins up the entire stack

---

## 🏗️ Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
│                    Angular Tracker UI                            │
│              (Live Map · Alerts · Fleet Stats)                   │
└───────────────────────────┬──────────────────────────────────────┘
                            │ HTTP / WebSocket
┌───────────────────────────▼──────────────────────────────────────┐
│                       API GATEWAY                                │
│            Spring Cloud Gateway (Port 8080)                      │
│         Route · Auth Filter · Rate Limit · Load Balance          │
└────────┬──────────────────────────────────────┬──────────────────┘
         │                                      │
┌────────▼────────┐                   ┌─────────▼────────┐
│ Ingestion       │                   │  Geofence        │
│ Service         │──── Kafka ───────▶│  Service         │
│ (GPS / Temp     │   (truck-pings)   │  (Zone tracking  │
│  / Speed pings) │                   │   & alerts)      │
└─────────────────┘                   └──────────────────┘
         │                                      │
         └──────────────┬───────────────────────┘
                        │
              ┌─────────▼──────────┐
              │    PostgreSQL DB    │
              │  (Truck state,     │
              │   Geofence zones,  │
              │   Event history)   │
              └────────────────────┘

         ┌────────────────────────────────┐
         │   Spring Cloud Infrastructure  │
         │  Config Server · Eureka Server │
         └────────────────────────────────┘
```

---

## 📁 Project Structure

```
logistics-asset-tracker/
├── .github/
│   └── workflows/          # CI/CD pipelines (GitHub Actions)
│
├── backend/
│   ├── config-server/      # Centralized configuration (Spring Cloud Config)
│   ├── discovery-server/   # Service registry (Spring Cloud Eureka)
│   ├── api-gateway/        # Single entry point, routing & auth (Spring Cloud Gateway)
│   ├── ingestion-service/  # Receives GPS pings → publishes to Kafka
│   └── geofence-service/   # Consumes Kafka events → evaluates zones & triggers alerts
│
├── frontend/
│   └── tracker-ui/         # Angular live dashboard (map, stats, alert feed)
│
├── docker-compose.yml      # Kafka + Zookeeper + PostgreSQL stack
└── README.md
```

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Backend Framework | Spring Boot 3, Spring Cloud |
| Service Mesh | Eureka (Discovery) + Spring Cloud Gateway |
| Config Management | Spring Cloud Config Server |
| Messaging | Apache Kafka + Zookeeper |
| Database | PostgreSQL |
| Frontend | Angular 17+ |
| Containerization | Docker + Docker Compose |
| CI/CD | GitHub Actions |

---

## 🚀 Getting Started

### Prerequisites

Make sure you have the following installed:
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (v24+)
- [Java 17+](https://openjdk.org/)
- [Node.js 18+](https://nodejs.org/) & npm
- [Maven 3.9+](https://maven.apache.org/)

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/logistics-asset-tracker.git
cd logistics-asset-tracker
```

### 2. Start the Infrastructure (Kafka, Zookeeper, PostgreSQL)

```bash
docker-compose up -d
```

> This spins up Kafka, Zookeeper, and PostgreSQL in the background. Wait ~30 seconds for services to be healthy.

### 3. Start Backend Services (in order)

```bash

cd backend/config-server && mvn spring-boot:run


cd backend/discovery-server && mvn spring-boot:run


cd backend/api-gateway && mvn spring-boot:run


cd backend/ingestion-service && mvn spring-boot:run


cd backend/geofence-service && mvn spring-boot:run
```

### 4. Start the Frontend

```bash
cd frontend/tracker-ui
npm install
ng serve
```

Navigate to **http://localhost:4200** to view the live fleet dashboard.

---

## 📊 How It Works

```
Simulated Truck  →  POST /api/ping  →  Ingestion Service
                                              │
                                        Kafka Topic
                                       (truck-pings)
                                              │
                                       Geofence Service
                                              │
                              ┌───────────────┴──────────────────┐
                              │                                  │
                       In Zone? → Alert           Engine Temp > 100°C? → Alert
                       Speed > 120? → Flag        Zone exit? → Notify
```

Each truck payload looks like this:

```json
{
  "truckId": "TRK-4821",
  "latitude": 48.8566,
  "longitude": 2.3522,
  "speedKmh": 87.4,
  "engineTempCelsius": 94.2,
  "timestamp": "2025-06-15T14:32:01Z"
}
```

---

## 🔭 Roadmap

- [ ] WebSocket push notifications to Angular dashboard
- [ ] Redis caching for truck state (reduce DB reads)
- [ ] ML-based ETA prediction per route
- [ ] Driver behavior scoring (harsh braking, speeding patterns)
- [ ] Multi-tenant support (manage multiple client fleets)
- [ ] Kubernetes deployment manifests (Helm charts)
- [ ] Prometheus + Grafana observability dashboard

---

## 🤝 Contributing

Contributions are welcome! Please open an issue first to discuss what you'd like to change. Pull requests should target the `develop` branch.

```bash

git checkout -b feature/your-feature-name


git commit -m "feat: add websocket support for live alerts"


git push origin feature/your-feature-name
```

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">
  <sub>Built By Placide  with  Java, Angular, and a passion for distributed systems.</sub>
</div>