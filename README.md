# LAAL - RedAmon Recon Engine (Standalone)

A fully functional, 1:1 replica of the RedAmon reconnaissance pipeline, optimized for standalone execution.

## 🚀 How to Run

### Option 1: Docker Mode (Recommended)
This is the easiest way as it handles all dependencies (Subfinder, Naabu, Httpx, Nuclei) automatically.

```bash
# Run one-off scan with arguments
docker-compose run recon --domain example.com
```

---

### Option 2: CLI Mode (Native Python)
Run directly from your host machine. Requires Go binaries (`subfinder`, `naabu`, etc.) to be in your PATH.

```bash
cd recon
pip install -r requirements.txt
python3 main.py --domain example.com
```

**Advanced Usage:**
- **Specific Subdomains:** `python3 main.py --domain example.com --subdomains testphp,www`
- **IP Scanning:** `python3 main.py --ip 1.2.3.4,8.8.8.8`

---

## 📂 Results
All findings are saved in the `recon/output/` directory as `recon_default_project.json`. 

## 🛠 Features
- **Standalone**: No Neo4j or PostgreSQL required.
- **Full Pipeline**: Subdomain discovery -> Port scan -> HTTP probe -> Resource enum -> Vuln scan.
- **Live Output**: Real-time terminal feedback for all long-running scans.
- **MITRE Enrichment**: Vulnerabilities are automatically enriched with CWE/CAPEC data.
