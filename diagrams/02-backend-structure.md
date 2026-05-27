# Diagram 2 — Backend Architecture (L2)

> **Excalidraw version:** [02-backend-structure.excalidraw](02-backend-structure.excalidraw) · Open at [excalidraw.com](https://excalidraw.com) for interactive editing.

```mermaid
flowchart LR
    subgraph CLIENTS["📱 Clients"]
        ANDROID["Android POS App\n(Java · Native)"]
        BROWSER["Web Portal Browser\n(Vue.js SPA)"]
    end

    APIGW["☁️ AWS API Gateway\nAuth · Rate-limit · Route"]

    subgraph DJANGO["🐍 Django REST Framework  ·  Python  ·  AWS EC2"]
        AUTH["Auth Middleware\n(Token / Session)"]

        subgraph CORE_SERVICES["Core Business Services"]
            ORDER_SVC["Order Management\nService"]
            MENU_SVC["Menu Management\nService"]
            INVENTORY_SVC["Inventory\nService"]
            PAYMENT_SVC["Payments & Billing\nService"]
            CRM_SVC["CRM\nService"]
            ANALYTICS_SVC["Analytics &\nReporting Engine"]
        end

        subgraph DATA_SERVICES["Data & Intelligence Services"]
            RECEIPT_SVC["Receipt\nService"]
            FORECAST_SVC["Demand Forecast\nService (Regression)"]
            APRIORI_SVC["Apriori Analytics\nEngine"]
        end
    end

    subgraph STORAGE["🗄 Data Storage"]
        MYSQL["MySQL\n(Multi-tenant DB)"]
        S3["AWS S3\n(Files · Exports · Receipts)"]
    end

    subgraph EXTERNAL["🔗 External"]
        PINELABS["Pine Labs\nPayment SDK"]
        EMAIL["Email / SMS\nGateway"]
        ZOMATO_DELIVERY["Zomato Delivery\nPlatform"]
    end

    ANDROID & BROWSER -->|HTTPS| APIGW
    APIGW --> AUTH
    AUTH --> CORE_SERVICES
    AUTH --> DATA_SERVICES

    ORDER_SVC & MENU_SVC & INVENTORY_SVC & CRM_SVC --> MYSQL
    PAYMENT_SVC & ANALYTICS_SVC --> MYSQL
    RECEIPT_SVC & FORECAST_SVC & APRIORI_SVC --> MYSQL

    RECEIPT_SVC & ANALYTICS_SVC & FORECAST_SVC --> S3

    PAYMENT_SVC -->|SDK| PINELABS
    RECEIPT_SVC -->|dispatch| EMAIL
    ORDER_SVC -->|delivery sync| ZOMATO_DELIVERY
```

---

### Service Module Reference

| Service | Responsibility | Storage | I/O |
|---|---|---|---|
| **Order Management** | Create, update, track, void orders; table management | MySQL | Android App ↔ API |
| **Menu Management** | Item listings, pricing, availability, remote push | MySQL | Android + Web Portal |
| **Inventory Service** | Raw material stock, wastage log, reorder alerts, recipe BOM deduction | MySQL | Android + Web Portal |
| **Payments & Billing** | Pine Labs SDK, tax computation, invoice generation, reconciliation | MySQL | Android + Pine Labs SDK |
| **CRM Service** | Customer profiles, visit history, preferences, loyalty | MySQL | Android + Web Portal |
| **Analytics Engine** | Real-time dashboards, sales trends, top-sellers, chain roll-up | MySQL + S3 | Web Portal |
| **Receipt Service** | Email/SMS receipt generation and dispatch | MySQL + S3 | Triggered by Payments |
| **Demand Forecast** | Regression-based 7–14 day item sales prediction | MySQL + S3 | Web Portal / Scheduler |
| **Apriori Engine** | Association rule mining on historical transactions | MySQL | Batch / Web Portal |
