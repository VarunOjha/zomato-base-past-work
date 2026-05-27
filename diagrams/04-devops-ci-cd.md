# Diagram 4 — DevOps & CI/CD Pipeline

> **Excalidraw version:** [04-devops-ci-cd.excalidraw](04-devops-ci-cd.excalidraw) · Open at [excalidraw.com](https://excalidraw.com) for interactive editing.

```mermaid
flowchart LR
    subgraph DEV["👨‍💻 Developer"]
        FEAT["Feature Branch\ngit commit + push"]
    end

    subgraph GITHUB["🐙 GitHub"]
        PR["Pull Request\nCode Review + Approval"]
        MAIN["main branch\n(protected)"]
    end

    subgraph JENKINS["⚙️ Jenkins CI/CD"]
        direction TB
        J1["1. Checkout\ngit clone main"]
        J2["2. Unit Tests\npytest / Django test runner"]
        J3["3. Build Backend\nPython package / Docker image"]
        J4["4. Build Android APK\nGradle assembleRelease"]
        J5["5. Deploy → Staging EC2\nssh + restart Gunicorn"]
        J6["6. QA Gate\n(manual approval)"]
        J7["7. Deploy → Production EC2\nssh + rolling restart"]

        J1 --> J2
        J2 -->|tests pass| J3
        J2 -->|parallel| J4
        J3 --> J5
        J5 --> J6
        J6 -->|approved| J7
    end

    subgraph AWS["☁️ AWS Infrastructure"]
        direction TB
        EC2_STG["EC2 Staging\n(t2.medium est.)"]
        EC2_PROD["EC2 Production\n(t2.medium / t2.large est.)"]
        APIGW["AWS API Gateway\n(prod endpoint)"]
        S3_ART["S3 Build Artifacts\n(APK + build logs)"]
        CW["CloudWatch\nLogs + Alarms"]
    end

    subgraph MOBILE["📱 Android Distribution"]
        APK_DIST["APK Build\nInternal Distribution"]
        POS_DEVICE["Posiflex / Samsung\nPOS Devices at Outlets"]
    end

    FEAT -->|push| PR
    PR -->|code approved + merged| MAIN
    MAIN -->|webhook trigger| J1

    J3 --> S3_ART
    J4 --> APK_DIST
    J5 --> EC2_STG
    J7 --> EC2_PROD
    EC2_PROD --> APIGW
    EC2_PROD -.->|logs| CW
    EC2_STG -.->|logs| CW
    APK_DIST -->|sideload / OTA| POS_DEVICE
```

---

### Pipeline Stage Reference

| Stage | Tool | Description | Gate |
|---|---|---|---|
| **Feature branch** | Git / GitHub | Developer commits on a named feature branch | — |
| **Pull Request** | GitHub PRs | Code review; at least 1 approval required | Manual review |
| **Merge to main** | GitHub | Merge into protected main branch; triggers Jenkins via webhook | PR approval |
| **Checkout** | Jenkins | Jenkins clones latest main at job start | Automatic |
| **Unit Tests** | pytest + Django test runner | Runs all unit and integration tests | Auto-fail on any test failure |
| **Build Backend** | Python / pip | Packages the Django application for deployment | Passes if tests pass |
| **Build Android APK** | Gradle (`assembleRelease`) | Builds signed APK for POS devices | Runs in parallel with backend build |
| **Deploy → Staging** | SSH + shell script | Copies build to staging EC2; restarts Gunicorn | Automatic post-build |
| **QA Gate** | Manual | QA engineer or senior engineer manually approves staging | Manual approval in Jenkins UI |
| **Deploy → Production** | SSH + rolling restart | Deploys to production EC2 with zero-downtime restart | Triggered by QA approval |
| **APK Distribution** | Internal / sideload | APK pushed to build store; installed on outlet POS devices | Manual for initial rollout |
| **Monitoring** | AWS CloudWatch | Both staging and prod EC2 instances push logs and metrics | Continuous |

### Branch Strategy

```
main (protected)
  └── feature/order-api
  └── feature/payment-service
  └── feature/analytics-portal
  └── fix/offline-sync-bug
  └── chore/db-migration-v2
```

> All feature branches are short-lived. Merges to `main` trigger the full Jenkins pipeline automatically via GitHub webhook. No direct pushes to `main`.
