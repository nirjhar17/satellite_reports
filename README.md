# Satellite Custom Report Templates

Custom ERB report templates for Red Hat Satellite 6.18+ to address regulatory compliance, SLA tracking, and vulnerability management.

## Reports

### 1. Host - CVE Exposure Report with CVSS Scores & Reboot Status (`host-cve-exposure-report-with-cvss-scores-and-reboot-status.erb`)

Consolidated compliance view for regulatory audits — shows all outstanding security vulnerabilities with severity, CVSS scores, and CVE details per host.

**What's special:** Adds CVSS Range, CVE Links (clickable URLs to Red Hat CVE pages), and Reboot Suggested — columns not available in any built-in Satellite report.

**Columns:** Host, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Patch Release Date, Available Since, Packages, CVEs, CVE Links, Reboot Suggested

---

### 2. Host - Patch SLA Breach Report with Days Unpatched (`host-patch-sla-breach-report-with-days-unpatched.erb`)

Calculates "Days Unpatched" per host/errata and flags SLA breaches based on severity thresholds (Critical: 30 days, Important: 60 days, Moderate/Low: 90 days).

**What's special:** Auto-calculates Days Unpatched and gives a YES/NO SLA Breach flag per errata — ready to hand to the regulator without manual Excel work.

**Columns:** Host, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Released, Available Since, Status, Days Unpatched, SLA Threshold, SLA Breach (YES/NO), CVEs, Packages

---

### 3. Host - Severity Impact Report with IP Addresses & Reboot Status (`host-severity-impact-report-with-ip-addresses-and-reboot-status.erb`)

Filter hosts by a specific severity level (Critical/Important/Moderate/Low) and get their IP addresses — for network team prioritization and firewall rules.

**What's special:** Includes IP Address per host and a severity dropdown filter — lets you instantly answer "which servers have Critical vulnerabilities and what are their IPs?"

**Columns:** Host, IP Address, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Patch Release Date, Available Since, Packages, CVEs, Reboot Suggested

---

### 4. Host - CVE Lookup Report with Affected IPs & Reboot Status (`host-cve-lookup-report-with-affected-ips-and-reboot-status.erb`)

Enter a specific CVE ID (e.g., CVE-2024-1234) and get all affected hosts with their IP addresses — for incident response when a critical vulnerability is published.

**What's special:** Reverse lookup by CVE ID — when a zero-day drops, instantly find every affected server and its IP without manually searching host by host.

**Columns:** Host, IP Address, Operating System, Lifecycle Environment, Erratum ID, Erratum Type, Severity, Packages, CVEs, Reboot Suggested

---

## How to Install

### Option A — GUI (Recommended)

1. Go to **Monitor → Report Templates → Create Report Template**
2. Paste the entire ERB file content (including the `<%# ... -%>` header)
3. Click **Submit** — template inputs are auto-created from the header
4. Assign to your Organization/Location via the **Edit** page

### Option B — API

```bash
SAT_URL="https://your-satellite.example.com"

# Escape template content and import
CONTENT=$(python3 -c "import sys,json; print(json.dumps(sys.stdin.read())[1:-1])" < host-cve-exposure-report-with-cvss-scores-and-reboot-status.erb)

curl -sk -u admin:password -X POST \
  -H 'Content-Type: application/json' \
  "${SAT_URL}/api/v2/report_templates/import" \
  -d "{\"report_template\": {\"name\": \"Host - Compliance Report\", \"template\": \"${CONTENT}\"}}"
```

**Note:** Always use the `/import` endpoint (not `/create`) — it auto-creates template inputs from the ERB header.

## Quick Start - Upload via Hammer CLI

Download all report templates:

```bash
curl -O https://raw.githubusercontent.com/nirjhar17/satellite_reports/main/host-cve-exposure-report-with-cvss-scores-and-reboot-status.erb
curl -O https://raw.githubusercontent.com/nirjhar17/satellite_reports/main/host-patch-sla-breach-report-with-days-unpatched.erb
curl -O https://raw.githubusercontent.com/nirjhar17/satellite_reports/main/host-severity-impact-report-with-ip-addresses-and-reboot-status.erb
curl -O https://raw.githubusercontent.com/nirjhar17/satellite_reports/main/host-cve-lookup-report-with-affected-ips-and-reboot-status.erb
```

Upload to Satellite (run on Satellite server):

```bash
hammer report-template create \
  --name "Host - CVE Exposure Report with CVSS Scores & Reboot Status" \
  --file host-cve-exposure-report-with-cvss-scores-and-reboot-status.erb \
  --organizations "Default Organization" \
  --locations "Default Location"

hammer report-template create \
  --name "Host - Patch SLA Breach Report with Days Unpatched" \
  --file host-patch-sla-breach-report-with-days-unpatched.erb \
  --organizations "Default Organization" \
  --locations "Default Location"

hammer report-template create \
  --name "Host - Severity Impact Report with IP Addresses & Reboot Status" \
  --file host-severity-impact-report-with-ip-addresses-and-reboot-status.erb \
  --organizations "Default Organization" \
  --locations "Default Location"

hammer report-template create \
  --name "Host - CVE Lookup Report with Affected IPs & Reboot Status" \
  --file host-cve-lookup-report-with-affected-ips-and-reboot-status.erb \
  --organizations "Default Organization" \
  --locations "Default Location"
```

> **Note:** Replace `"Default Organization"` and `"Default Location"` with your actual organization and location names. The `--file` flag ensures Hammer parses the ERB header and auto-creates all template inputs.

## How to Generate

- **GUI:** Monitor → Report Templates → Select report → Generate → Fill inputs → Generate
- **API:** `POST /api/v2/report_templates/<ID>/generate` with `input_values`

## Compatibility

- Red Hat Satellite 6.18+
- Requires Katello plugin (content management enabled)
- Hosts must be registered with applicable errata for reports to return data
