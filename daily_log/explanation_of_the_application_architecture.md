# Explanation of the Application Architecture

## The basic architecture of my application I would like to build is as follows:

```mermaid
flowchart LR
  FE@{ shape: rect, label: "Frontend" }
  BFF@{ shape: rect, label: "BFF(Backend For Frontend)" }
  BE@{ shape: rect, label: "Backend" }
  BAT@{ shape: rect, label: "Batch" }
  DB@{ shape: cyl, label: "Database" }

  FE -->|graphql|BFF
  BFF -->|gRPC|BE
  BE --> DB
  BAT --> DB
```

## Technologies to be used (still under review)

| Application Type | Framework | 
| --- | --- | 
| Frontend | Next.js(Version: 15.x, Using App Router) | 
| BFF(Frontend For Backend) | Nest.js(10 or 11), graphql |
| Backend | Nest.js(10 or 11), gRPC |
| Batch | Spring boot(3.4.1, Quartz or another) |
| PostgreSQL | 16 |
| MongoDB | 8.0 |
| Kafka | for kafka CDC(debezium/kafka:3.0.0.Final) |
| connect | for kafka CDC(debezium/connect:3.0.0.Final) |
| zookeeper | for kafka CDC(debezium/zookeeper:3.0.0.Final) |

- The reason I chose Nest.js over Spring Boot for the backend is as follows
    - (While I am more familiar with Spring Boot)
    1. learn new technologies
    1. to use gRPC(faster then REST API)

- Also considering whether to use Airflow for Batch Application.

## MSA(Microservice Architecture) is applied, the application architecture will look as follows:

```mermaid
flowchart LR
    subgraph fb[Frontend & BFF]
        direction LR
        FE1@{ shape: rect, label: "Frontend" }
        BFF1@{ shape: rect, label: "BFF(Backend For Frontend)" }
        FE2@{ shape: rect, label: "Frontend" }
        BFF2@{ shape: rect, label: "BFF(Backend For Frontend)" }
        FE1 -->|graphql|BFF1
        FE2 -->|graphql|BFF2
    end
    subgraph be[Backend]
        direction LR
        BE1@{ shape: rect, label: "Backend" }
        BE2@{ shape: rect, label: "Backend" }
    end
    subgraph bat[Batch]
        direction LR
        BAT1@{ shape: rect, label: "Batch" }
        BAT2@{ shape: rect, label: "Batch" }
    end
    subgraph db[Database]
        direction LR
        DB1@{ shape: cyl, label: "Database" }
        DB2@{ shape: cyl, label: "Database" }
    end

    BFF1 -->|gRPC| BE1
    BFF1 -->|gRPC| BE2
    BFF2 -->|gRPC| BE1
    BFF2 -->|gRPC| BE2
    BE1 --> DB1
    BE2 --> DB2
    BAT1 --> DB1
    BAT2 --> DB2
```

## In the Kubernetes environment, it will be executed as shown below:

- For clarity, only the application itself is represented, excluding Service, Ingress, egress, etc.

```mermaid
flowchart LR
    subgraph k8s[Cluster]
        direction LR
        subgraph app1[Application]
            direction LR
            FE1@{ shape: processes, label: "Frontend Deployment" }
            BFF1@{ shape: processes, label: "BFF Deployment" }
            BE1@{ shape: processes, label: "Backend Deployment" }
        end
        subgraph app2[Application]
            direction LR
            FE2@{ shape: processes, label: "Frontend Deployment" }
            BFF2@{ shape: processes, label: "BFF Deployment" }
            BE2@{ shape: processes, label: "Backend Deployment" }
        end
        subgraph bat[Batch]
            direction LR
            BAT1@{ shape: processes, label: "Batch Deployment" }
            BAT2@{ shape: processes, label: "Batch Deployment" }
        end
    end
    subgraph db[Database]
        direction LR
        DB1@{ shape: cyl, label: "Database" }
        DB2@{ shape: cyl, label: "Database" }
    end

    FE1 -->|graphql|BFF1
    FE2 -->|graphql|BFF2
    BFF1 -->|gRPC| BE1
    BFF1 -->|gRPC| BE2
    BFF2 -->|gRPC| BE1
    BFF2 -->|gRPC| BE2
    BE1 --> DB1
    BE2 --> DB2
    BAT1 --> DB1
    BAT2 --> DB2
  
```

- I will make every effort to cover everything from database design to application development. 

- Furthermore, I am diligently studying English. 
