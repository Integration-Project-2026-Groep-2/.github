<div align="center">

<a href="https://github.com/Integration-Project-2026-Groep-2"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/banner.svg" alt="Integration Project 2026 — Groep 2" width="1440"></a>

[![Kubernetes](https://img.shields.io/badge/Infra-Kubernetes_%28kubeadm%29-326CE5?style=flat-square&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Flannel](https://img.shields.io/badge/CNI-Flannel-00b4d8?style=flat-square&logo=kubernetes&logoColor=white)](https://github.com/flannel-io/flannel)
[![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-EF7B4D?style=flat-square&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![RabbitMQ](https://img.shields.io/badge/Broker-RabbitMQ-FF6600?style=flat-square&logo=rabbitmq&logoColor=white)](https://www.rabbitmq.com/)
[![Elasticsearch](https://img.shields.io/badge/Search-Elasticsearch_9-005571?style=flat-square&logo=elasticsearch&logoColor=white)](https://www.elastic.co/)
[![Kibana](https://img.shields.io/badge/Dashboards-Kibana_9.3-E8478B?style=flat-square&logo=kibana&logoColor=white)](https://www.elastic.co/kibana)
[![Cloudflare](https://img.shields.io/badge/DNS%2FTLS-Cloudflare-F38020?style=flat-square&logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![GHCR](https://img.shields.io/badge/Registry-ghcr.io-181717?style=flat-square&logo=github&logoColor=white)](https://ghcr.io)

**A fully integrated event management platform — 9 microservices, one Kubernetes cluster.**  
Services communicate asynchronously via **RabbitMQ** (XML/XSD), are deployed through **ArgoCD** GitOps,  
monitored in real-time by **Elasticsearch + Kibana**, and AI-diagnosed via the **Model Context Protocol**.

🌐 **[www.integration-project-2026-groep-2.my.be](https://www.integration-project-2026-groep-2.my.be)**

</div>

---

## 📑 Table of Contents

- [Project Overview](#-project-overview)
- [System Architecture](#-system-architecture)
- [Services](#-services)
- [Infrastructure & Kubernetes](#️-infrastructure--kubernetes)
- [Messaging — RabbitMQ](#-messaging--rabbitmq)
- [Observability — Controlroom](#-observability--controlroom)
- [AI Integration — MCP](#-ai-integration--mcp)
- [IoT — Badge Scanner](#-iot--badge-scanner)
- [CI/CD Pipeline](#️-cicd-pipeline)
- [Repository Structure](#-repository-structure)
- [Getting Started (Local)](#-getting-started-local)
- [Getting Started (Kubernetes)](#️-getting-started-kubernetes)
- [Branch Strategy](#-branch-strategy)
- [Credits](#-credits)

---

## 🎯 Project Overview

**Integration Project 2026 — Groep 2** is a polyglot microservice platform built for EhB (Erasmushogeschool Brussel). It manages the full lifecycle of an event: visitor registration, session scheduling, on-site payments & consumption tracking, invoicing, transactional email, and real-time platform monitoring — all connected through a shared message broker.

The platform runs on a **single-node Kubernetes cluster** (expandable), deployed via **ArgoCD GitOps** across separate `dev` and `prod` namespaces, with traffic entering through **Cloudflare** onto **NGINX Gateway Fabric**.

| Capability | Service | Technology |
|---|---|---|
| Public website & visitor registration | Frontend | Drupal (PHP) |
| Customer relationship management | CRM | Salesforce (Python) |
| Session & speaker scheduling | Planning | Node.js (TypeScript) |
| On-site payments & consumptions | Kassa | Odoo POS (Python) |
| QR-Code badge identification | IoT Badge Scanner | Raspberry Pi (Python) |
| Invoicing & billing | Facturatie | FOSSBilling (PHP) |
| Transactional email | Mailing | SendGrid (JavaScript) |
| Monitoring, logging & alerting | Controlroom | Go + Elasticsearch |
| AI-assisted diagnostics | mcp-master | Rust (MCP) |
| Message transport | — | RabbitMQ (XML/AMQP) |
| Deployment | — | Kubernetes + ArgoCD |

---

## 🏗 System Architecture

### Service Interaction Map

```mermaid
graph TD
    classDef svc  fill:#1e3a5f,color:#93c5fd,stroke:#3b82f6,stroke-width:1.5px
    classDef obs  fill:#072a3a,color:#7dd3fc,stroke:#0369a1,stroke-width:1.5px
    classDef ai   fill:#1e1040,color:#d8b4fe,stroke:#7c3aed,stroke-width:1.5px
    classDef iot  fill:#1c1205,color:#fde68a,stroke:#d97706,stroke-width:1.5px
    classDef ext  fill:#1a1a1a,color:#94a3b8,stroke:#334155,stroke-width:1px
    classDef rmq  fill:#1a0c00,color:#fb923c,stroke:#FF6600,stroke-width:2px

    Browser(["👤 Visitor / Browser"]):::ext
    Admin(["🛠️ Organiser / Admin"]):::ext

    Browser -->|HTTPS| FE["🌐 Frontend\nDrupal (PHP)"]:::svc
    Admin   -->|HTTPS| FE
    Admin   -->|HTTPS| CRM["📋 CRM\nSalesforce (Python)"]:::svc
    Admin   -->|HTTPS| PLAN["📅 Planning\nNode.js (TS)"]:::svc
    Admin   -->|HTTPS| KSA["🛒 Kassa\nOdoo POS (Python)"]:::svc

    IOT["🔌 IoT Badge Scanner\nRaspberry Pi (Python)"]:::iot -->|HTTP| PlatformAPI["Platform API"]

    subgraph BROKER ["⚡ RabbitMQ Cluster (dev + prod)"]
        direction LR
        E1["heartbeat.direct"]:::rmq
        E2["user.topic"]:::rmq
        E3["invoice.topic"]:::rmq
        E4["contact.topic"]:::rmq
        E5["news.topic"]:::rmq
        E6["logs.direct"]:::rmq
        E7["..."]:::rmq
    end

    FE   <-->|XML/AMQP| BROKER
    CRM  <-->|XML/AMQP| BROKER
    PLAN <-->|XML/AMQP| BROKER
    KSA  <-->|XML/AMQP| BROKER
    FAC["🧾 Facturatie\nFOSSBilling (PHP)"]:::svc <-->|XML/AMQP| BROKER
    MAIL["✉️ Mailing\nSendGrid (JS)"]:::svc      <-->|XML/AMQP| BROKER
    CR["🖥️ Controlroom\nGo"]:::obs               <-->|XML/AMQP| BROKER

    CR -->|index| ES[("📊 Elasticsearch")]:::obs
    ES --> KIB["📈 Kibana 9.3"]:::obs
    CR -->|Adaptive Cards| TEAMS["📢 MS Teams"]:::ext

    MCP["🤖 mcp-master\nRust"]:::ai -->|MCP/HTTP| CR
    MCP -->|MCP/HTTP| CRM
    MCP -->|"/metrics"| PROM["📡 Prometheus"]:::obs
```

### Kubernetes Cluster Architecture

```mermaid
graph TB
    Internet(["🌍 Internet"])
    CF["☁️ Cloudflare\nDNS + TLS"]
    NP["NodePort :30097"]

    Internet --> CF --> NP

    subgraph K8S["☸️ Kubernetes Cluster — kubeadm, single-node (expandable)"]
        direction TB

        subgraph CI["🔧 Cluster Infrastructure"]
            NGWF["NGINX Gateway Fabric\n(Gateway API)"]
            ARGO["🐙 ArgoCD (App-of-Apps)"]
            SS["🔐 Sealed Secrets"]
            HL["🪖 Headlamp UI"]
        end

        NP --> NGWF

        subgraph DEV["📦 integration-project-2026-groep-2-dev"]
            D9["9 Service Pods"]
            DRMQ[("RabbitMQ Cluster\n(Cluster Operator)")]
            DES[("Elasticsearch Cluster\n(ECK Operator)")]
            DKIB["Kibana"]
        end

        subgraph PROD["📦 integration-project-2026-groep-2-prod"]
            P9["9 Service Pods"]
            PRMQ[("RabbitMQ Cluster\n(Cluster Operator)")]
            PES[("Elasticsearch Cluster\n(ECK Operator)")]
            PKIB["Kibana"]
        end

        NGWF --> DEV
        NGWF --> PROD
        ARGO -->|sync| DEV
        ARGO -->|sync| PROD

        FL["🌐 Flannel CNI"]
        NP2["🔒 Network Policies\n(zero-trust)"]
    end

    GH["🐙 k8s-manifests\n(GitHub)"] -->|GitOps| ARGO
    GHCR["📦 ghcr.io\n(container images)"] --> DEV & PROD
```

### CI/CD Flow

```mermaid
flowchart LR
    DEV(["👨‍💻 Developer"])
    DEV -->|"git push / PR"| GH["GitHub"]

    subgraph CI["🔁 CI — GitHub Actions"]
        UT["Unit Tests"]
        IT["Integration Tests\n(Docker Compose)"]
        UT --> IT
    end

    GH -->|"on: push / PR"| CI

    subgraph CD["🚀 CD — GitHub Actions"]
        BI["Build Docker Image"]
        PI["Push → ghcr.io"]
        PA["Patch tag in\nk8s-manifests"]
        BI --> PI --> PA
    end

    CI -->|CI passes + main| CD

    PA -->|"commit"| ARGO["🐙 ArgoCD"]
    ARGO -->|"auto-sync"| K8S["☸️ Kubernetes"]
```

---

## 🧩 Services

### <a href="https://github.com/Integration-Project-2026-Groep-2/Frontend"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/frontend.svg" alt="Frontend" width="320"></a>
[![PHP](https://img.shields.io/badge/PHP-8.x-777BB4?style=flat-square&logo=php&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Frontend)
[![Drupal](https://img.shields.io/badge/CMS-Drupal-0678BE?style=flat-square&logo=drupal&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Frontend)
[![Repo](https://img.shields.io/badge/Repo-Frontend-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/Frontend)

The main public-facing website built with **Drupal**. Visitors register, view the event schedule, and manage their profile. Custom Drupal modules handle RabbitMQ publishing and consuming for user lifecycle events.

| | |
|---|---|
| Language | PHP |
| Framework | Drupal |
| Database | MySQL |
| Publishes to | `contact.topic`, `heartbeat.direct`, `logs.direct` |
| Consumes from | `crm.user.confirmed`, `news.topic` |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/CRM"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/crm.svg" alt="CRM" width="320"></a>
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/CRM)
[![Salesforce](https://img.shields.io/badge/CRM-Salesforce-00A1E0?style=flat-square&logo=salesforce&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/CRM)
[![AsyncAPI](https://img.shields.io/badge/AsyncAPI-v1.5.0-2D9CDB?style=flat-square)](https://github.com/Integration-Project-2026-Groep-2/CRM/blob/main/docs/crm-asyncapi-v1.yaml)
[![Repo](https://img.shields.io/badge/Repo-CRM-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/CRM)

Salesforce integration layer. A single Python container runs **3 asyncio tasks** and manages **23 XML contracts** (AsyncAPI v1.5.0). Acts as the canonical source of truth for users and companies across the platform. Also exposes an MCP server for AI-assisted diagnostics.

| | |
|---|---|
| Language | Python |
| External system | Salesforce |
| Architecture | 1 container → 3 asyncio tasks + sender utility |
| XML contracts | 23 (heartbeat, status, user lifecycle, company events) |
| Publishes to | `contact.topic`, `heartbeat.direct`, `crm.status.checked` |
| Consumes from | 11 queues across other services |
| MCP server | Exposes diagnostic tools to `mcp-master` |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/Facturatie"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/facturatie.svg" alt="Facturatie" width="320"></a>
[![PHP](https://img.shields.io/badge/PHP-8.x-777BB4?style=flat-square&logo=php&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Facturatie)
[![FOSSBilling](https://img.shields.io/badge/Billing-FOSSBilling_0.7.2-E63946?style=flat-square)](https://github.com/Integration-Project-2026-Groep-2/Facturatie)
[![Repo](https://img.shields.io/badge/Repo-Facturatie-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/Facturatie)

Billing and invoicing service based on **FOSSBilling 0.7.2**. Generates and finalises invoices from CRM user events, then emits `invoice.finalized` to trigger Mailing (which dispatches the invoice via SendGrid).

| | |
|---|---|
| Language | PHP + Twig |
| Framework | FOSSBilling 0.7.2 |
| Database | MariaDB |
| Heartbeat interval | 1 second |
| Publishes to | `invoice.topic` (`invoice.finalized`), `heartbeat.direct` |
| Consumes from | `crm.user.*` |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/Planning"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/planning.svg" alt="Planning" width="320"></a>
[![TypeScript](https://img.shields.io/badge/TypeScript-Node.js_24-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Planning)
[![PostgreSQL](https://img.shields.io/badge/DB-PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Planning)
[![Repo](https://img.shields.io/badge/Repo-Planning-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/Planning)

Manages event **sessions, locations, and speakers**. Exposes a REST API consumed by the Frontend and communicates bidirectionally over RabbitMQ for user and session lifecycle events.

| | |
|---|---|
| Language | TypeScript |
| Runtime | Node.js 24 LTS |
| Database | PostgreSQL |
| Health endpoint | `GET /health` → `{"status":"ok","service":"planning"}` |
| Publishes to | `heartbeat.direct`, `logs.direct` |
| Consumes from | `contact.topic`, `crm.user.*` |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/Mailing"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/mailing.svg" alt="Mailing" width="320"></a>
[![JavaScript](https://img.shields.io/badge/JavaScript-Node.js-F7DF1E?style=flat-square&logo=javascript&logoColor=black)](https://github.com/Integration-Project-2026-Groep-2/Mailing)
[![SendGrid](https://img.shields.io/badge/Email-SendGrid-00B0E8?style=flat-square&logo=twilio&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Mailing)
[![Repo](https://img.shields.io/badge/Repo-Mailing-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/Mailing)

Transactional email service powered by **SendGrid**. Listens across three topic exchanges and dispatches templated emails for user confirmations, invoice notifications, and news broadcasts.

| | |
|---|---|
| Language | JavaScript |
| Runtime | Node.js |
| Database | MariaDB |
| Email provider | SendGrid (dynamic templates) |
| Publishes to | `user.topic`, `heartbeat.direct` |
| Consumes from | `contact.topic`, `invoice.topic`, `news.topic` |

**Key flows:**
```
crm.user.confirmed  → validate XSD → upsert user  → welcome email    → mail_logs
invoice.finalized   → validate XSD → fetch invoice → send PDF email   → mail_logs
news.notify.all     → validate XSD → fetch users   → broadcast email  → mail_logs
```

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/Kassa"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/kassa.svg" alt="Kassa" width="320"></a>
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Kassa)
[![Odoo](https://img.shields.io/badge/POS-Odoo-714B67?style=flat-square&logo=odoo&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Kassa)
[![Repo](https://img.shields.io/badge/Repo-Kassa-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/Kassa)

Physical **point-of-sale system** based on **Odoo POS**, used at the event venue. Tracks consumption and processes payments. Integrated with the IoT Badge Scanner so attendees can identify themselves by scanning their QR code.

| | |
|---|---|
| Language | Python + JavaScript |
| Framework | Odoo POS |
| IoT integration | iot-badge-scanner (Raspberry Pi, QR-Code) |
| Publishes to | `heartbeat.direct`, `logs.direct` |
| Consumes from | `crm.user.confirmed`, `contact.topic` |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/Controlroom"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/controlroom.svg" alt="Controlroom" width="320"></a>
[![Go](https://img.shields.io/badge/Go-1.26-00ADD8?style=flat-square&logo=go&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Controlroom)
[![Elasticsearch](https://img.shields.io/badge/Elasticsearch-9-005571?style=flat-square&logo=elasticsearch&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Controlroom)
[![Kibana](https://img.shields.io/badge/Kibana-9.3.1-E8478B?style=flat-square&logo=kibana&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/Controlroom)
[![Repo](https://img.shields.io/badge/Repo-Controlroom-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/Controlroom)

The **central monitoring, ingestion, and diagnostic hub** of the platform. Written in Go, it ingests all service telemetry into Elasticsearch, auto-provisions Kibana dashboards, fires Microsoft Teams alerts, and exposes an MCP server for AI agents.

| | |
|---|---|
| Language | Go 1.26 |
| Storage | Elasticsearch |
| Dashboards | Kibana 9.3.1 (auto-provisioned from `dashboards.ndjson`) |
| Alerting | Microsoft Teams (Adaptive Cards webhooks) |
| Kubernetes scrape | Pod metadata every 5 seconds |
| MCP server | `controlroom-mcp` — exposes tools for AI diagnostics |

**Capabilities at a glance:**

| Component | Description |
|---|---|
| **Watchdog** | Monitors heartbeat counts; alerts Teams when a service goes offline or recovers |
| **Ingestors** | Heartbeats, logs, status checks, user/company events, check-ins — indexed to Elasticsearch |
| **Kibana Sync** | Auto-provisions Data Views and dashboards on startup |
| **Dead-Letter Safety** | `controlroom.dlx` captures failed messages; reconnects with exponential backoff (1s → 60s) |
| **MCP Server** | `error_analysis`, `heartbeat_status`, `fetch_logs`, `fetch_recent_deploys`, `fetch_recent_commits`, `fetch_blob`, `request_changes` (auto-PR to GitHub) |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/mcp-master"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/mcp-master.svg" alt="mcp-master" width="320"></a>
[![Rust](https://img.shields.io/badge/Rust-stable-CE422B?style=flat-square&logo=rust&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/mcp-master)
[![Prometheus](https://img.shields.io/badge/Metrics-Prometheus-E6522C?style=flat-square&logo=prometheus&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/mcp-master)
[![Repo](https://img.shields.io/badge/Repo-mcp--master-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/mcp-master)

A **Rust-based MCP aggregator** that fans out to the individual MCP servers exposed by Controlroom and CRM, presenting a single entry-point for AI assistants (Claude, Gemini, or custom agents) to diagnose and manage the full platform.

| | |
|---|---|
| Language | Rust |
| Connects to | Controlroom MCP, CRM MCP |
| Metrics | Prometheus `/metrics` endpoint (`--server-mode`) |
| Default LLM | Claude Sonnet 4.x (configurable) |

**Prometheus metrics exposed:**

| Metric | Type | Description |
|---|---|---|
| `http_requests_total` | counter | Requests by method, route, status |
| `llm_tokens_total` | counter | Input / output / cache tokens |
| `mcp_tool_calls_total` | counter | Tool calls by name, server, outcome |
| `chat_requests_total` | counter | Chat requests by mode and outcome |
| `incidents_total` | counter | Incident events detected |
| `incident_pipeline_duration_seconds` | summary | End-to-end incident resolution time |

---

### <a href="https://github.com/Integration-Project-2026-Groep-2/iot-badge-scanner"><img src="https://raw.githubusercontent.com/Integration-Project-2026-Groep-2/.github/main/profile/assets/badges/iot-badge-scanner.svg" alt="IoT Badge Scanner" width="320"></a>
[![Python](https://img.shields.io/badge/Python-Raspberry_Pi-3776AB?style=flat-square&logo=python&logoColor=white)](https://github.com/Integration-Project-2026-Groep-2/iot-badge-scanner)
[![Repo](https://img.shields.io/badge/Repo-iot--badge--scanner-181717?style=flat-square&logo=github)](https://github.com/Integration-Project-2026-Groep-2/iot-badge-scanner)

**Raspberry Pi** QR-Code badge scanner deployed physically at the event venue. Attendees present or scan their QR code to identify themselves at the point-of-sale (POS) for consumption tracking and payments. Structured as a clean client/server/shared architecture.

| | |
|---|---|
| Language | Python |
| Hardware | Raspberry Pi + camera / QR-code scanner |
| Architecture | `client/` (Pi scanner) · `server/` (bridge) · `shared/` (contracts) |
| Integration | HTTP to platform services |
| Deployment | **Edge device** — runs on physical hardware, not in-cluster |

---

## ☸️ Infrastructure & Kubernetes

### Cluster Setup

The platform runs on a **single-node Kubernetes cluster** provisioned with `kubeadm`, designed to be horizontally expanded at any time by joining additional worker nodes.

| Component | Detail |
|---|---|
| Provisioner | `kubeadm` |
| CNI | Flannel |
| Node topology | Single node (expandable) |
| Ingress / L7 routing | NGINX Gateway Fabric (Gateway API) |
| External DNS + TLS | Cloudflare → NodePort `:30097` |
| Secret management | Kustomize `secretGenerator` (`.env`-based) + Sealed Secrets |
| Broker operator | RabbitMQ Cluster Operator (shared, manages dev + prod) |
| Search operator | Elastic Cloud on Kubernetes — ECK (shared, manages dev + prod) |
| Optional UI | Headlamp |

### Namespaces

```
integration-project-2026-groep-2-dev    # Development environment
integration-project-2026-groep-2-prod   # Production environment
argocd                                  # ArgoCD control plane
nginx-gateway                           # NGINX Gateway Fabric
kube-system                             # Flannel, Sealed Secrets
```

### k8s-manifests Repository Structure

> All Kubernetes manifests live in [k8s-manifests](https://github.com/Integration-Project-2026-Groep-2/k8s-manifests).

```
k8s-manifests/
├── namespace.yaml                  # Namespace definitions
├── kustomization.yaml              # Root Kustomize — generates secrets from .env
├── argocd/
│   └── app-of-apps.yaml            # App-of-Apps root Application
├── apps/                           # Per-service ArgoCD Applications + K8s manifests
│   ├── frontend/
│   ├── crm/
│   ├── facturatie/
│   ├── planning/
│   ├── mailing/
│   ├── kassa/
│   ├── controlroom/
│   ├── mcp-master/
│   └── iot-badge-scanner/
├── infrastructure/                 # RabbitMQ, Elasticsearch, Kibana operators + CRs
├── gateway/                        # Gateway API config + HTTPRoutes
├── network-policies/               # Zero-trust pod-to-pod segmentation
└── secrets/                        # Secret templates (no real values committed)
```

### ArgoCD — App-of-Apps

```mermaid
graph LR
    ROOT["🐙 App-of-Apps\nargocd"]

    ROOT --> DEV["📦 dev namespace"]
    ROOT --> PROD["📦 prod namespace"]

    DEV --> DI["RabbitMQ Cluster\nElasticsearch + Kibana"]
    DEV --> DA["Frontend · CRM · Facturatie\nPlanning · Mailing · Kassa\nControlroom · mcp-master · IoT"]

    PROD --> PI["RabbitMQ Cluster\nElasticsearch + Kibana"]
    PROD --> PA["Frontend · CRM · Facturatie\nPlanning · Mailing · Kassa\nControlroom · mcp-master · IoT"]
```

### Networking

| Layer | Technology |
|---|---|
| External DNS + TLS termination | Cloudflare |
| Cluster entrypoint | NodePort `:30097` |
| L7 host-based routing | NGINX Gateway Fabric (Gateway API) |
| Inter-pod security | Kubernetes Network Policies (zero-trust) |
| Internal pod networking | Flannel CNI |

---

## 📨 Messaging — RabbitMQ

All 9 services communicate asynchronously through **RabbitMQ**. Each environment (dev/prod) has its own dedicated RabbitMQ cluster, both managed by a single **RabbitMQ Cluster Operator** running in the cluster. Messages are encoded as **XML** and validated against **XSD schemas** at both producer and consumer. The CRM service alone defines **23 AsyncAPI v1.5.0 contracts**.

### Exchange Topology

```mermaid
graph LR
    subgraph PUB["Publishers"]
        FE["Frontend"]
        CRM["CRM"]
        FAC["Facturatie"]
        MAIL["Mailing"]
        PLAN["Planning"]
        KSA["Kassa"]
        CR["Controlroom"]
    end

    subgraph EX["RabbitMQ Exchanges"]
        HB["heartbeat.direct"]
        CT["contact.topic"]
        IT["invoice.topic"]
        NT["news.topic"]
        UT["user.topic"]
        LG["logs.direct"]
        ST["crm.status.checked"]
        DLX["controlroom.dlx\n(Dead-Letter)"]
    end

    subgraph SUB["Subscribers"]
        CR2["Controlroom\n(all heartbeats + logs)"]
        MAIL2["Mailing\n(user confirmed, invoice finalized, news)"]
        FE2["Frontend\n(user confirmed)"]
        FAC2["Facturatie\n(user events)"]
    end

    FE   -->|"routing.heartbeat"| HB
    CRM  -->|"crm.user.*"| CT
    CRM  -->|"crm.status.*"| ST
    FAC  -->|"invoice.finalized"| IT
    MAIL -->|"mailing.user.*"| UT
    PLAN -->|"routing.heartbeat"| HB
    KSA  -->|"routing.heartbeat"| HB
    FAC  -->|"routing.heartbeat"| HB
    CR   -->|"news.notify.all"| NT

    HB --> CR2
    LG --> CR2
    CT --> MAIL2
    IT --> MAIL2
    NT --> MAIL2
    CT --> FE2
    CT --> FAC2

    DLX -->|failed messages| Q1["*.dlq queues"]
```

### Message Format

All messages are XML documents validated against XSD schemas on both sides. Example heartbeat:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Heartbeat>
    <serviceId>mailing</serviceId>
    <timestamp>2026-05-23T12:00:00.000Z</timestamp>
</Heartbeat>
```

Schemas live in each service's `contracts/` or `pkg/xsd/` directory. The Controlroom ships a custom `xmlgen` tool that auto-generates typed Go structs from XSD files.

### Dead-Letter Strategy

Failed messages (parse errors, XSD validation failures, consumer crashes) are automatically routed to `controlroom.dlx` and land in Dead-Letter Queues. The Controlroom reconnects on broker disconnection using **exponential backoff** (1 s → 60 s ceiling).

---

## 📊 Observability — Controlroom

### Data Flow

```mermaid
flowchart TD
    SVC["All 9 Services"]

    SVC -->|"heartbeat XML\nevery 1 second"| HBQ["controlroom.heartbeat.queue"]
    SVC -->|"WARN / ERROR logs"| LQ["controlroom.logs.queue"]
    SVC -->|"status check XML"| SCQ["statuscheck queue"]

    HBQ --> CR["🖥️ Controlroom (Go)"]
    LQ  --> CR
    SCQ --> CR

    CR -->|"index"| E1[("ES: heartbeats")]
    CR -->|"index"| E2[("ES: controlroom-logs")]
    CR -->|"index"| E3[("ES: statuscheck")]
    CR -->|"index"| E4[("ES: users")]
    CR -->|"scrape every 5s"| E5[("ES: k8s-pods")]

    E1 & E2 & E3 & E4 & E5 --> KIB["📈 Kibana\n(auto-provisioned dashboards)"]

    CR --> WD["🐕 Watchdog"]
    WD -->|"heartbeat &lt; 30 in 60s"| TEAMS["📢 Microsoft Teams\nCritical Alarm"]
    WD -->|"service recovers"| TEAMS2["📢 Microsoft Teams\nResolution Alert"]
    WD -->|"HeartbeatStatusEvent"| RMQ["RabbitMQ"]
```

### Kibana Data Views

After deploying, create these Data Views in **Stack Management → Data Views** with `@timestamp` as the time field:

| Index | Content |
|---|---|
| `heartbeats` | Per-service heartbeat stream (1 s cadence) |
| `controlroom-logs` | Aggregated WARN / ERROR logs from all services |
| `statuscheck` | Service health status history |
| `users` | User registration and check-in events |
| `k8s-pods` | Kubernetes pod metadata, scraped every 5 s |

### Alerting Thresholds

| Condition | Action |
|---|---|
| Service heartbeat drops below **30** in last **60 s** | 🚨 Teams Critical Alarm + XML event to RabbitMQ |
| Service heartbeat recovers above threshold | ✅ Teams Resolution alert |
| ERROR / WARN spike detected | Teams notification |
| Weekly digest | XML published to `news.topic` |

---

## 🤖 AI Integration — MCP

The platform has a full **Model Context Protocol** integration layer, enabling AI assistants to diagnose, analyse, and resolve issues autonomously — including opening pull requests with fixes.

```mermaid
graph LR
    AGENT["🤖 AI Agent\n(e.g. Claude)"]

    AGENT -->|MCP / HTTP| MCPM["mcp-master\n(Rust)"]

    MCPM -->|MCP / HTTP| CR_MCP["Controlroom MCP\n(Go)"]
    MCPM -->|MCP / HTTP| CRM_MCP["CRM MCP\n(Python)"]

    CR_MCP --> ES["Elasticsearch\n(logs, heartbeats, pods)"]
    CR_MCP --> GH["GitHub API\n(commits, PRs, workflows)"]
    CRM_MCP --> SF["Salesforce\n(users, companies)"]
```

**Available MCP tools (Controlroom):**

| Tool | Description |
|---|---|
| `error_analysis` | Query Elasticsearch error logs with Lucene strings |
| `heartbeat_status` | Fetch recent heartbeat events per service |
| `statuscheck_summary` | Review recent service health status history |
| `fetch_logs` | Get ERROR/WARN logs around a timestamp window |
| `fetch_recent_deploys` | Query GitHub Actions workflow run history |
| `fetch_recent_commits` | Get recent commits per service repository |
| `fetch_blob` | Read file contents from GitHub by path or SHA |
| `request_changes` | **Auto-open a pull request** with a proposed fix |

---

## 🔌 IoT — Badge Scanner

The **iot-badge-scanner** runs on a **Raspberry Pi** physically deployed at the event venue. It reads attendee QR codes and communicates with platform services to associate visitor identities with POS transactions.

```
iot-badge-scanner/
├── client/    # Runs on the Raspberry Pi — reads QR codes, sends scan events
├── server/    # Bridge process (connects scanner events to platform services)
└── shared/    # Shared contracts and data models
```

> The scanner is an **edge device** and does **not** run inside Kubernetes. It connects outbound to the cluster's exposed platform endpoint (via the domain) or directly over the local event network.

---

## ⚙️ CI/CD Pipeline

Every service repository follows the identical pipeline pattern:

### 1 — Continuous Integration (GitHub Actions)

Triggered on every **PR** and **push to `main`**:

1. **Unit tests** — isolated business logic
2. **Integration tests** — full Docker Compose stack with RabbitMQ + DB, validates XML/XSD contracts end-to-end

### 2 — Continuous Deployment (GitHub Actions → ArgoCD)

Triggered when CI passes for commits on `main`.

- A push (merge) to `main` automatically deploys to the `dev` environment: build → push to **`ghcr.io/...:latest`** → patch the `dev` manifests in `k8s-manifests` → ArgoCD auto-syncs `dev`.
- Production deployments are gated on GitHub Releases: creating a Release (a tagged release marked "ready for production") triggers the CD pipeline to promote the image/tag and patch the `prod` manifests, then ArgoCD auto-syncs `prod`.

```
Developer → PR → CI (tests pass) → merge to main
    → CD → deploy to dev (auto)
Developer → Create GitHub Release (tag + "ready for production")
    → CD → promote to prod (release-triggered)
```

> ⚠️ **Deployment is gated on CI.** No image reaches any environment unless tests pass; promotions to production require a GitHub Release.

---

## 📁 Repository Structure

| Repository | Language | Description |
|---|---|---|
| [Frontend](https://github.com/Integration-Project-2026-Groep-2/Frontend) | ![PHP](https://img.shields.io/badge/-PHP-777BB4?style=flat-square&logo=php&logoColor=white) | Drupal event website & visitor registration portal |
| [CRM](https://github.com/Integration-Project-2026-Groep-2/CRM) | ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) | Salesforce integration, 23 XML contracts (AsyncAPI v1.5) |
| [Facturatie](https://github.com/Integration-Project-2026-Groep-2/Facturatie) | ![PHP](https://img.shields.io/badge/-PHP-777BB4?style=flat-square&logo=php&logoColor=white) | FOSSBilling 0.7.2 — invoicing & billing |
| [Planning](https://github.com/Integration-Project-2026-Groep-2/Planning) | ![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) | Sessions, locations & speakers management |
| [Mailing](https://github.com/Integration-Project-2026-Groep-2/Mailing) | ![JavaScript](https://img.shields.io/badge/-JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) | SendGrid transactional email service |
| [Kassa](https://github.com/Integration-Project-2026-Groep-2/Kassa) | ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) | Odoo POS — on-site cash register & consumption tracker |
| [Controlroom](https://github.com/Integration-Project-2026-Groep-2/Controlroom) | ![Go](https://img.shields.io/badge/-Go-00ADD8?style=flat-square&logo=go&logoColor=white) | Monitoring hub — Elasticsearch ingestion, watchdog, MCP |
| [mcp-master](https://github.com/Integration-Project-2026-Groep-2/mcp-master) | ![Rust](https://img.shields.io/badge/-Rust-CE422B?style=flat-square&logo=rust&logoColor=white) | MCP aggregator, Prometheus metrics, AI diagnostics |
| [iot-badge-scanner](https://github.com/Integration-Project-2026-Groep-2/iot-badge-scanner) | ![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white) | Raspberry Pi QR-Code badge scanner (edge device) |
| [k8s-manifests](https://github.com/Integration-Project-2026-Groep-2/k8s-manifests) | ![Kubernetes](https://img.shields.io/badge/-Kubernetes-326CE5?style=flat-square&logo=kubernetes&logoColor=white) | All Kubernetes manifests — ArgoCD App-of-Apps GitOps |
| [Infra](https://github.com/Integration-Project-2026-Groep-2/Infra) | ![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat-square&logo=docker&logoColor=white) | Local Docker Compose, nginx & RabbitMQ configs, scripts |

---

## 🌿 Branch Strategy

All service repositories follow the same branching model (most repos use `main` + feature branches):

| Branch | Purpose |
|---|---|
| `main` | Primary branch: CI runs on PRs and pushes; a push/merge to `main` triggers CI and auto-deploys to the `dev` environment. Production deployments are performed only via a GitHub Release (tagged "ready for production"). Use PR merges — do not push directly. |
| `feature/xxx` | Feature work — one branch per task, open a PR to `main` when complete. |

CI runs on every PR and every push to `main`. A push (merge) to `main` automatically deploys to the `dev` environment. Production deployments occur only when a GitHub Release (a tagged release marked "ready for production") is created, which triggers promotion to `prod`.

---

## 💜 Credits

<div align="center">

Made with ❤️ by **EhB Integration Project 2026 Groep 2**

*Erasmushogeschool Brussel — Academic Year 2025 / 2026*

[![GitHub Org](https://img.shields.io/badge/GitHub-Integration--Project--2026--Groep--2-181717?style=for-the-badge&logo=github)](https://github.com/Integration-Project-2026-Groep-2)
[![Domain](https://img.shields.io/badge/Live-integration--project--2026--groep--2.my.be-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://www.integration-project-2026-groep-2.my.be)

</div>
