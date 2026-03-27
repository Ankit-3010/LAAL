# LAAL - LLM-Augmented Autonomous Attack Lifecycle

An autonomous AI framework that chains reconnaissance, exploitation, and post-exploitation into a single pipeline.

## 🚀 How to Run

### Docker Mode
This is the easiest way as it handles all dependencies (Subfinder, Naabu, Httpx, Nuclei) automatically.

```bash
# Run one-off scan with arguments
docker-compose run recon --domain example.com
```


### Orchestrator Communication Flow

```mermaid
sequenceDiagram
    participant Webapp as Webapp UI
    participant Orchestrator as Recon Orchestrator
    participant Recon as Recon Container
    participant API as Webapp API
    participant Neo4j as Neo4j

    Webapp->>Orchestrator: POST /recon/{projectId}/start
    Orchestrator->>Recon: docker run with PROJECT_ID, WEBAPP_API_URL
    Recon->>API: GET /api/projects/{projectId}
    API-->>Recon: Project settings (169+ params)
    Recon->>Recon: Execute scan pipeline
    Recon->>Neo4j: Update graph with results
    Orchestrator->>Webapp: SSE log stream
    Recon-->>Orchestrator: Container exits
    Orchestrator->>Webapp: Complete event
```


## 🏗️ Docker-in-Docker Architecture

The recon module uses a **Docker-in-Docker (DinD)** pattern where the main recon container orchestrates sibling containers for each scanning tool.

### How It Works

The recon container shares the **host's Docker daemon** via a socket mount, meaning all containers are **siblings** managed by the same host Docker daemon.

```mermaid
flowchart TB
    subgraph Host["🖥️ HOST MACHINE"]
        subgraph DockerDaemon["Docker Daemon (dockerd)"]
            Socket["/var/run/docker.sock"]
        end

        subgraph Containers["Sibling Containers"]
            Recon["redamon-recon<br/>Python Orchestrator<br/>📋 Coordinates all scans"]
            NaabuC["naabu<br/>projectdiscovery/naabu<br/>🔌 Port Scanner"]
            HttpxC["httpx<br/>projectdiscovery/httpx<br/>🌐 HTTP Prober"]
            NucleiC["nuclei<br/>projectdiscovery/nuclei<br/>🎯 Vuln Scanner"]
            KatanaC["katana<br/>projectdiscovery/katana<br/>🕸️ Web Crawler"]
            GAUC["gau<br/>sxcurity/gau<br/>📚 URL Archives"]
            PurednsC["puredns<br/>frost19k/puredns<br/>🧹 Wildcard Filter"]
        end

        Volume["📁 Shared Volume<br/>recon/output/"]
    end

    Socket -.->|socket mount| Recon
    Recon -->|docker run| NaabuC
    Recon -->|docker run| HttpxC
    Recon -->|docker run| NucleiC
    Recon -->|docker run| KatanaC
    Recon -->|docker run| GAUC
    Recon -->|docker run| PurednsC

    NaabuC --> Volume
    HttpxC --> Volume
    NucleiC --> Volume
    KatanaC --> Volume
    GAUC --> Volume
    Recon --> Volume
```


## 📂 Results
All findings are saved in the `recon/output/` directory as `recon_default_project.json`. 

## Tool Integration
Install and configure the following tools in the recon image:
*   **Discovery**: `subfinder`, `amass`, `knockpy`, `puredns`.
*   **Port Scanning**: `naabu`.
*   **HTTP Probing**: `httpx`, `wappalyzer`.
*   **Crawling**: `katana`, `hakrawler`, `gau`, `paramspider`.
*   **Vulnerability Scanning**: `nuclei` (with 9,000+ templates).
