# Mermaid-tester

flowchart LR
  %% ===== Edge and Networking =====
  subgraph EDGE[Edge and Networking]
    DNS[Route53 DNS]
    CDN[CloudFront CDN]
    WAF[WAF and Bot Filter]
    ALB[ALB\nL7 Load Balancer]
  end

  %% ===== Platform and Shared Services =====
  subgraph PLATFORM[Platform and Shared Services]
    GATE[API Gateway\nSpring Cloud Gateway]
    DISC[Eureka Service Discovery]
    CONF[Spring Cloud Config Server]
    AUTH[Auth\nOIDC JWT]
    REDIS[(Redis Cluster)]
    SECRETS[AWS Secrets Manager]
  end

  %% ===== Application Services =====
  subgraph APP[Application Services]
    subgraph EMP[employee-service]
      EAPI[/REST api v1 employees/]
      OUTBOX[(outbox_event)]
      RELAY[OutboxRelay\nScheduled]
    end
    subgraph DEP[department-service]
      DAPI[/REST api v1 departments/]
      EVDEP[/POST events employee/]
      RLOG[(received_event_log)]
    end
    subgraph PROJ[project-service optional]
      PAPI[/REST api v1 projects/]
      EVPROJ[/POST events employee/]
    end
  end

  %% ===== Data and Storage =====
  subgraph DATA[Data and Storage]
    PGE[(Postgres employee_db)]
    PGD[(Postgres department_db)]
    PGP[(Postgres project_db)]
    S3[(S3 buckets)]
  end

  %% ===== Observability =====
  subgraph OBS[Observability]
    OTEL[OpenTelemetry SDK]
    PROM[Prometheus or CloudWatch Metrics]
    LOGS[Loki or ELK JSON Logs]
    TRC[Jaeger or XRay Tracing]
    GRAF[Grafana Dashboards]
  end

  %% ===== CI CD =====
  subgraph CICD[CI CD]
    SCM[Git Repo]
    PIPE[Build and Deploy Pipeline]
    IMG[Container Registry ECR]
  end

  %% ===== Flows =====
  USER((Client)) --> DNS --> CDN --> WAF --> ALB --> GATE

  GATE -- HTTP --> EAPI
  GATE -- HTTP --> DAPI
  GATE -- HTTP --> PAPI
  GATE <--> AUTH
  GATE <--> REDIS

  EAPI <--> DISC
  DAPI <--> DISC
  PAPI <--> DISC
  EAPI <--> CONF
  DAPI <--> CONF
  PAPI <--> CONF
  EAPI <--> AUTH
  DAPI <--> AUTH
  PAPI <--> AUTH
  EAPI <--> REDIS
  DAPI <--> REDIS
  PAPI <--> REDIS
  EAPI <--> SECRETS
  DAPI <--> SECRETS
  PAPI <--> SECRETS

  EAPI --> OUTBOX
  RELAY --> OUTBOX
  RELAY -- POST X-Event-Type --> EVDEP
  RELAY -- POST X-Event-Type --> EVPROJ
  EVDEP --> RLOG

  EAPI --> PGE
  DAPI --> PGD
  PAPI --> PGP

  EAPI --> S3
  DAPI --> S3
  PAPI --> S3

  EAPI -. metrics logs traces .-> OTEL
  DAPI -. metrics logs traces .-> OTEL
  PAPI -. metrics logs traces .-> OTEL
  OTEL --> PROM
  OTEL --> LOGS
  OTEL --> TRC
  PROM --> GRAF
  LOGS --> GRAF
  TRC --> GRAF

  SCM --> PIPE --> IMG
  PIPE --> GATE
  PIPE --> EAPI
  PIPE --> DAPI
  PIPE --> PAPI
