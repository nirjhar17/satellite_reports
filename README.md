# Satellite Custom Report Templates

Custom ERB report templates for Red Hat Satellite 6.18+ to address regulatory compliance, SLA tracking, and vulnerability management.

## Reports

### 1. Host - Compliance Report (`host-compliance-report.erb`)

Consolidated compliance view for regulatory audits — shows all outstanding security vulnerabilities with severity, CVSS scores, and CVE details per host.

**Columns:** Host, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Patch Release Date, Available Since, Packages, CVEs, CVE Links, Reboot Suggested

---

### 2. Host - Patch Tracking & SLA Report v2 (`host-patch-tracking-sla-report-v2.erb`)

Calculates "Days Unpatched" per host/errata and flags SLA breaches based on severity thresholds (Critical: 30 days, Important: 60 days, Moderate/Low: 90 days).

**Columns:** Host, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Released, Available Since, Status, Days Unpatched, SLA Threshold, SLA Breach (YES/NO), CVEs, Packages

---

### 3. Host - Severity Lookup Report (`host-severity-lookup-report.erb`)

Filter hosts by a specific severity level (Critical/Important/Moderate/Low) and get their IP addresses — for network team prioritization and firewall rules.

**Columns:** Host, IP Address, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Patch Release Date, Available Since, Packages, CVEs, Reboot Suggested

---

### 4. Host - CVE Impact Lookup Report (`host-cve-impact-lookup-report.erb`)

Enter a specific CVE ID (e.g., CVE-2024-1234) and get all affected hosts with their IP addresses — for incident response when a critical vulnerability is published.

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
CONTENT=$(python3 -c "import sys,json; print(json.dumps(sys.stdin.read())[1:-1])" < host-compliance-report.erb)

curl -sk -u admin:password -X POST \
  -H 'Content-Type: application/json' \
  "${SAT_URL}/api/v2/report_templates/import" \
  -d "{\"report_template\": {\"name\": \"Host - Compliance Report\", \"template\": \"${CONTENT}\"}}"
```

**Note:** Always use the `/import` endpoint (not `/create`) — it auto-creates template inputs from the ERB header.

## How to Generate

- **GUI:** Monitor → Report Templates → Select report → Generate → Fill inputs → Generate
- **API:** `POST /api/v2/report_templates/<ID>/generate` with `input_values`

## Compatibility

- Red Hat Satellite 6.18+
- Requires Katello plugin (content management enabled)
- Hosts must be registered with applicable errata for reports to return data
