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

**Diagram (Business Layer / Vision):**

```mermaid
flowchart TD
    subgraph BUS ["Business Layer"]
        USER["External User / Consumer"]
        BUS_SVC[["Business Service: Iris Classification"]]
    end
    USER --> BUS_SVC
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
```
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
