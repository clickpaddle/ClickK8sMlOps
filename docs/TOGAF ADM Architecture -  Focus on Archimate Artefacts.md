# From Business Vision to DevOps: Modeling an Iris Classification Use Case Across the TOGAF ADM Cycle

In modern enterprise architecture, **having a structured approach to modeling systems is crucial**. One of the challenges many organizations face is the **lack of clarity in representing Business, Application, and Technology layers**. Without disciplined modeling, these layers often become a **jumbled mix of concepts, components, and technologies**, resulting in diagrams that are difficult to interpret and misalign stakeholders.

The **TOGAF Architecture Development Method (ADM)** provides a **step-by-step framework** to ensure architectural work remains consistent and traceable, from **high-level business objectives** down to **solution deployment and operational readiness**. At each phase of the ADM cycle, specific **artefacts and diagrams** help communicate the design:

- **Phase A & B (Architecture Vision and Business Architecture)**: Focus on the **business service, stakeholders, and objectives**. Artefacts highlight the value delivered to users and the processes that support it.
- **Phase C (Information Systems & Data Architecture)**: Represent **application components, data objects, and technology capabilities**. Diagrams clarify how business services are realized in terms of applications and data.
- **Phase D & E (Technology Architecture and Opportunities / Solutions)**: Detail the **platform, runtime, and infrastructure layers**, providing visibility on **deployment, scaling, and observability**. Artefacts support DevOps teams by linking conceptual models to deployable solutions.

Throughout these phases, **visual clarity is key**. The **relationships between business services, application components, and technology capabilities** must be explicit, showing not just the components but **how they interact, realize, and depend on one another**.

From an **architectural perspective**, it is essential to **avoid overloading diagrams with secondary or repetitive details**. In this use case, we have **explicitly excluded the operational steps of training the Iris model**. While training is a critical ML process, including it in the diagrams would introduce **additional components and interactions** that are **not central to understanding the realization of the business service and its deployment**. This selective abstraction is part of my IT architecture approach: **focus on clarity, traceability, and the evolution of the architecture, rather than modeling every operational detail**.

An additional challenge is **audience comprehension**. Architects (“purists”) may value **semantic accuracy and formal representations**, while developers and DevOps teams often need **practical clarity for implementation and operations**. Therefore, it is important to **seek feedback from both groups** to ensure that diagrams communicate effectively **without losing rigor**, and that subtleties in the representation of the three layers are understood at every phase of the ADM cycle.

To illustrate this approach, we use a **practical example: an Iris Classification ML service**. This use case is simple enough to follow through all ADM phases, yet **rich enough to demonstrate the evolution of the architecture from business vision to deployed, observable service**. We will present diagrams for **each phase**, showing **how the business layer, application layer, and technology layer are progressively elaborated**, and highlighting the **key artefacts and relationships**.

---

## 1️⃣ Phases A & B – Architecture Vision & Business Architecture

- **Phase A (Architecture Vision)**: Define the **high-level business objectives** and the **scope of the Iris Classification service**. Focus on **stakeholders, external users, and business value**.
- **Phase B (Business Architecture)**: Map **business processes and services** to support these objectives.

For our use case:  

- **Business Layer**: “Iris Classification” as a **business service** consumed by external users.
- At this stage, the **application and technology layers are abstract**, ensuring we understand the **business needs** before technical design.

**Diagram (Business Vision):**

```mermaid
flowchart TD
    subgraph BUS ["Business Layer"]
        USER["External User / Consumer"]
        BUS_SVC[["Business Service: Iris Classification"]]
    end
    USER --> BUS_SVC
```
**Diagram (Business Business):**
```mermaid
flowchart TB
    %% Business Layer – Phase B pour IRIS (orange, sans training)
    style BUSINESS fill:#FFA500,stroke:#333,stroke-width:2px

    subgraph BUSINESS ["Business Layer (Phase B) – IRIS"]
        direction TB

        USER["Business Actor: End User / Client"]
        DATA_SCIENTIST["Business Role: Data Scientist"]
        PREDICTION_PROCESS["Business Process: Iris Prediction Process"]
        VALIDATION_RULES["Business Function: Input Validation / Rules"]
        REPORTING["Business Function: Reporting & Feedback"]
        MODEL_REGISTRY["Business Object: IRIS Model Registry (S3)"]
    end

    %% Relations Business Layer (IRIS)
    USER -->|triggers prediction| PREDICTION_PROCESS
    DATA_SCIENTIST -->|owns / monitors prediction| PREDICTION_PROCESS
    PREDICTION_PROCESS -->|applies| VALIDATION_RULES
    PREDICTION_PROCESS -->|generates| REPORTING
    PREDICTION_PROCESS -->|reads| MODEL_REGISTRY
```

## 2️⃣ Phase C – Information Systems & Data Architecture

Phase C focuses on **Application and Data Architecture**, while keeping strong **business alignment**.

- **Business Layer**: Maintains the **Iris Classification service**, linked to external users.
- **Application Layer**: The **Iris Classification API** and **predictor logic component**.
- **Data Layer**: The **Iris Model Artifact (.pkl)** and the **training dataset**.
- **Technology Layer (Capabilities)**: Abstract capabilities like **Model Serving**, **Data Storage**, **API Entry** — no runtime specifics yet.

**Phase C Diagram – Conceptual / Information Systems:**

```mermaid
flowchart TD

    %% --- BUSINESS LAYER ---
    subgraph BUS ["Business Layer"]
        USER["External User / Consumer"]
        BUS_SVC[["Business Service: Iris Classification"]]
    end

    %% --- APPLICATION LAYER ---
    subgraph APP ["Application Layer"]
        IRIS_SVC[["Application Service: Iris Classification Service"]]
        IRIS_COMP["Application Component: Iris Predictor Logic"]
    end

    %% --- DATA OBJECTS ---
    subgraph DATA ["Data Objects"]
        MODEL_ARTIFACT[("Iris Model Artifact (.pkl)")]
        TRAINING_DATA[("Iris Training Dataset")]
    end

    %% --- TECHNOLOGY / CAPABILITY LAYER ---
    subgraph TECH ["Technology Layer (Capabilities)"]
        SERVE_CAP["Capability: Model Serving"]
        STORAGE_CAP["Capability: Data Storage / Artifact Repository"]
        INGRESS_CAP["Capability: External Traffic / API Entry"]
    end

    %% --- RELATIONSHIPS ---
    USER --> BUS_SVC
    BUS_SVC -. "realized by" .-> IRIS_SVC
    IRIS_SVC --- IRIS_COMP
    IRIS_COMP -. "reads" .-> MODEL_ARTIFACT
    IRIS_COMP -. "reads" .-> TRAINING_DATA
    IRIS_SVC -- "exposed via" --> INGRESS_CAP
    IRIS_COMP -- "depends on" --> SERVE_CAP
    MODEL_ARTIFACT -- "stored in" --> STORAGE_CAP
    TRAINING_DATA -- "stored in" --> STORAGE_CAP
```

## 3️⃣ Phases D & E – Technology Architecture & Opportunities / Solutions

Phases D and E detail **how the solution is realized and deployed**, moving from **conceptual to operational**.

- **Business Layer**: Minimal — keep the **external user** and the **Iris Classification service** for traceability.
- **Application Layer**: API and ML inference logic, now considered **deployable artifacts**.
- **Platform / Runtime Layer**: Kubernetes cluster, **KServe**, **Knative**, **NGINX Ingress**, **Istio Service Mesh**, ArgoCD/GitOps.
- **Infrastructure & Observability**: Pods running the model, S3 storage for model weights, monitoring via Prometheus/Grafana.

**Phase D/E Diagram – Solution & DevOps Ready:**

Shows all explicit technical flows: HTTPS requests, proxies, auto-scaling, artifact pulls from S3, metrics exposure via Prometheus/Grafana.
Clear and immediately usable for development and operations teams.
Provides a clear view of who does what, where, and how, without relying on architectural terminology.

```mermaid
flowchart TB

    %% --- BUSINESS LAYER (minimal) ---
    subgraph BUS ["Business Layer"]
        BUS_SVC[["Business Service: Iris Classification"]]
    end

    %% --- USER LAYER ---
    subgraph USER ["User Layer"]
        CLIENT(["External Consumer"])
    end

    %% --- APPLICATION LAYER ---
    subgraph APP ["Application Layer"]
        IRIS_API[["Iris Classification API"]]
        MODEL_LOGIC{{"ML Inference Logic"}}
    end

    %% --- PLATFORM / RUNTIME LAYER ---
    subgraph PLATFORM ["Platform & Serving Layer"]
        
        subgraph TRAFFIC ["Traffic & Mesh"]
            INGRESS["NGINX Ingress"]
            ISTIO["Istio Service Mesh"]
            VS_DR["VirtualService (Routing Rules)"]
        end

        subgraph SERVING ["Model Serving (KServe)"]
            KSERVE["KServe Controller"]
            KNATIVE["Knative Runtime"]
            POD_ML["Model Pod (Instance)"]
        end

        subgraph AUTOMATION ["GitOps & Delivery"]
            ARGO["ArgoCD"]
            GIT[("Git Repos")]
        end
    end

    %% --- INFRASTRUCTURE LAYER ---
    subgraph INFRA ["Infrastructure & Data"]
        K8S[["Kubernetes Cluster"]]
        S3_MODEL[("S3: Model Weights (.pkl)")]
        PROM["Prometheus / Grafana"]
    end

    %% --- RELATIONSHIPS ---
    CLIENT --> BUS_SVC
    BUS_SVC -. "realized by" .-> IRIS_API
    IRIS_API -- "HTTPS Request" --> INGRESS
    INGRESS -- "Proxies to" --> ISTIO
    ISTIO -- "Applies Routing" --> VS_DR
    VS_DR -- "Balances Traffic" --> POD_ML
    IRIS_API -- "Executes" --> MODEL_LOGIC
    MODEL_LOGIC -- "Runs inside" --> POD_ML
    KSERVE -- "Orchestrates" --> KNATIVE
    KNATIVE -- "Scales (0->N)" --> POD_ML
    ARGO -- "Synchronizes" --> K8S
    GIT -- "Defines State" --> ARGO
    POD_ML -- "Pulls Artifact" --> S3_MODEL
    POD_ML -- "Exposes Metrics" --> PROM
    ISTIO -- "Reports Telemetry" --> PROM
```
**Phase D/E Diagram – Archimate Compliant and readable by DevOps:**

Adheres to ArchiMate layers (Business / Application / Technology / Infrastructure).
Uses exact ArchiMate terms but adds technical labels readable by DevOps.
Ensures TOGAF traceability, while remaining understandable for technical teams.
Clarifies the separation of Technology Services vs Technology Components, and integrates observability as a technology service.
Practical Challenge
This is often where friction appears:
- Architects insist on ArchiMate vocabulary and TOGAF compliance, which is important for traceability and official documentation.
- Technicians / DevOps teams want clear arrows and concrete labels to immediately understand runtime behavior and deployment pipelines.

Even with double-labeling, some teams may “pick apart” the vocabulary or find it unnecessarily abstract for operational purposes.

```mermaid
flowchart TB
    %% --- BUSINESS LAYER ---
    style BUS fill:#FF9900,stroke:#333,stroke-width:2px
    subgraph BUS ["Business Layer"]
        CLIENT["Business Actor: External Consumer"]
        BUS_SVC[["Business Service: Iris Classification"]]
    end

    %% --- APPLICATION LAYER ---
    style APP fill:#00B0F0,stroke:#333,stroke-width:2px
    subgraph APP ["Application Layer"]
        IRIS_API[["Application Service: Iris Classification API"]]
        MODEL_LOGIC["Application Component: ML Inference Logic"]
    end

    %% --- TECHNOLOGY / PLATFORM LAYER ---
    style PLATFORM fill:#98FB98,stroke:#333,stroke-width:2px
    subgraph PLATFORM ["Technology / Platform Layer"]
        
        subgraph TRAFFIC ["Traffic & Mesh"]
            INGRESS["Technology Service: NGINX Ingress"]
            ISTIO["Technology Service: Istio Service Mesh"]
            VS_DR["Technology Object: VirtualService / Routing Rules"]
        end

        subgraph SERVING ["Model Serving"]
            KSERVE["Technology Service: KServe Controller"]
            KNATIVE["Technology Service: Knative Runtime (scales 0->N)"]
            POD_ML["Technology Component: Model Pod (Instance)"]
        end

        subgraph AUTOMATION ["GitOps & Delivery"]
            ARGO["Technology Service: ArgoCD"]
            GIT[("Data Object: Git Repos")]
        end

        %% Observability
        PROM["Technology Service: Prometheus / Grafana"]
    end

    %% --- INFRASTRUCTURE / DATA LAYER ---
    style INFRA fill:#A9DFBF,stroke:#333,stroke-width:2px
    subgraph INFRA ["Infrastructure & Data"]
        K8S[["Technology Component: Kubernetes Cluster"]]
        S3_MODEL[("Data Object: Model Weights (.pkl)")]
    end

    %% --- RELATIONSHIPS (double label technique / ArchiMate) ---

    %% Business -> Application
    CLIENT -->|triggers / triggers| BUS_SVC
    BUS_SVC -.->|realized by / realized by| IRIS_API

    %% Application internal
    IRIS_API -->|executes / assigned to| MODEL_LOGIC

    %% Application -> Technology / Platform
    IRIS_API -->|HTTPS request / accesses| INGRESS
    INGRESS -->|Proxies to / serves| ISTIO
    ISTIO -->|Applies routing / serves| VS_DR
    VS_DR -->|Balances traffic / serves| POD_ML
    MODEL_LOGIC -->|Runs inside / deployed on| POD_ML
    KSERVE -->|Orchestrates / serves| KNATIVE
    KNATIVE -->|Scales pods / serves| POD_ML

    %% DevOps / Automation
    ARGO -->|Synchronizes / serves| K8S
    GIT -->|Defines state / accesses| ARGO

    %% Data -> Technology
    POD_ML -->|Pulls artifact / accesses| S3_MODEL

    %% Observability
    POD_ML -->|Exports metrics / serves| PROM
    ISTIO -->|Reports telemetry / serves| PROM
    KNATIVE -->|Exports metrics / serves| PROM

```





**Final Question**

In this context, which representation should be preferred for Phases D/E :
The DevOps Ready version, clear and operational, but less formally ArchiMate?
The ArchiMate Compliant version with double labeling, more TOGAF-compliant but slightly denser?
Or a hybrid approach, where the ArchiMate diagram serves as a reference, and a DevOps Ready excerpt is provided to operational teams?
The goal is to find a balance between architectural compliance and operational practicality so that all stakeholders can collaborate effectively.

## Conclusion

Through this Iris Classification use case, we have walked the architecture **from business vision to deployed solution**, highlighting how the **Business, Application, and Technology layers** evolve across the **TOGAF ADM cycle**. By progressively elaborating artefacts and diagrams, we can **maintain clarity, traceability, and alignment with business objectives**, while preparing for deployment and observability.

This exercise also illustrates an important point for IT architects: **not every operational detail needs to be represented**. By abstracting certain processes, such as model training in this case, we keep diagrams **focused and comprehensible**, while still conveying the essence of how business services are realized in deployable systems.

Now, we would like to open the discussion to the broader **enterprise and solutions architecture community**:  

- How do you approach **representing the business, application, and technology layers** in your ADM cycle?  
- Which **phase do you focus on most** when creating diagrams for stakeholders versus DevOps teams?  
- How do you balance **conceptual clarity** with the **practical details required for deployment**?  
- Are there **specific artefacts or visualizations** you prefer at different phases of TOGAF ADM?

Your insights and experiences are invaluable to ensure that architecture work is **both rigorous and actionable**, bridging the gap between formal enterprise architecture and practical implementation.
