# Diagram 1 — Product Ecosystem Flow

> **Excalidraw version:** [01-product-ecosystem-flow.excalidraw](01-product-ecosystem-flow.excalidraw) · Open at [excalidraw.com](https://excalidraw.com) for interactive editing.

```mermaid
flowchart TD
    subgraph OUTLET["🏪 Restaurant Outlet"]
        direction LR
        POS["Android POS App\n(Samsung Tab / Posiflex)"]
        KOT["KOT Printer\n(Bluetooth / USB)"]
        PINE["Pine Labs Terminal\n(Debit / Credit)"]
        OFFLINE["Offline Mode\n(SQLite Queue)"]
    end

    subgraph CHAIN_HQ["🖥 Restaurant Chain HQ"]
        WEB["Web Portal\n(Vue.js + Laravel)"]
    end

    APIGW["☁️ AWS API Gateway\nAuth · Rate-limit · Request Routing"]

    subgraph BACKEND["⚙️ Backend — Python · Django REST Framework · AWS EC2"]
        direction LR
        ORD["Order\nManagement"]
        INV["Inventory\nService"]
        PAY["Payments\n& Billing"]
        CRM_SVC["CRM\nService"]
        ANA["Analytics\nEngine"]
        MENU["Menu\nManagement"]
        RECEIPT["Receipt\nService"]
        FORECAST["Demand\nForecast"]
        APRIORI["Apriori\nEngine"]
    end

    subgraph DATA["🗄 Data Layer"]
        direction LR
        DB["MySQL\n(Multi-tenant DB)"]
        S3["AWS S3\n(Receipts · Media · Exports)"]
    end

    subgraph INTEGRATIONS["🔗 External Integrations"]
        direction LR
        ZD["Zomato Delivery\nPlatform"]
        NOTIF["Email / SMS\nGateway"]
    end

    subgraph CLOUD["☁️ Cloud Infrastructure"]
        direction LR
        EC2["AWS EC2\n(Backend Servers)"]
        CW["AWS CloudWatch\n(Monitoring)"]
    end

    POS -->|REST / HTTPS| APIGW
    WEB -->|REST / HTTPS| APIGW
    POS -.->|print ticket| KOT
    POS -.->|card tap| PINE
    OFFLINE -.->|sync on reconnect| POS

    APIGW --> BACKEND

    ORD & INV & PAY & CRM_SVC & ANA & MENU & RECEIPT & FORECAST & APRIORI --> DB
    RECEIPT & ANA & FORECAST --> S3

    ORD -->|delivery orders| ZD
    PAY & RECEIPT --> NOTIF

    BACKEND -.->|hosted on| EC2
    EC2 -.->|metrics| CW
```

---

### Layer Reference

| Layer | Components | Technology |
|---|---|---|
| **Outlet Hardware** | Android POS App, KOT Printer, Pine Labs Terminal | Android Java, Bluetooth/USB, Pine Labs SDK |
| **Chain HQ Web** | Web Portal | Vue.js, Laravel (PHP) |
| **API Gateway** | AWS API Gateway | AWS — auth, rate-limiting, routing |
| **Backend Services** | 9 Django service modules | Python, Django REST Framework, AWS EC2 |
| **Data Layer** | MySQL (multi-tenant), AWS S3 | MySQL 5.6/5.7, AWS S3 |
| **External Integrations** | Zomato Delivery, Email/SMS | REST integrations |
| **Cloud Infra** | AWS EC2, CloudWatch | AWS |
