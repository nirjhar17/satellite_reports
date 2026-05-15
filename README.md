# Satellite Custom Report Templates

Custom report templates for Red Hat Satellite 6.18+ to enhance patch compliance reporting.

## Host - Compliance Report

A single unified report template that extends the built-in "Host - Available Errata" with additional compliance-relevant columns.

## What It Adds

The built-in "Host - Available Errata" template provides 11 columns. This custom template adds:

- **CVSS Range** — Maps Red Hat severity (Critical/Important/Moderate/Low) to the corresponding CVSS v3 score range using Red Hat's official classification
- **CVE Links** — Direct URLs to the Red Hat CVE page for each CVE, enabling quick verification of exact CVSS scores

## CVSS Range Mapping

The CVSS Range column uses Red Hat's official severity-to-CVSS classification:

- Critical → 9.0–10.0
- Important → 7.0–8.9
- Moderate → 4.0–6.9
- Low → 0.1–3.9

Source: [Red Hat Severity Ratings](https://access.redhat.com/security/updates/classification/)

## Output Columns (13 total)

- Host
- Operating System
- Environment
- Erratum
- Type
- Severity
- CVSS Range
- Patch Release Date
- Available Since
- Packages
- CVEs
- CVE Links
- Reboot suggested

## How to Install

**Option A — Clone from GUI:**

- Navigate to Monitor → Reports → Report Templates
- Find "Host - Available Errata" → click the dropdown → Clone
- Rename to "Host - Compliance Report"
- Replace the ERB content with the contents of `host-compliance-report.erb`
- Click Submit

**Option B — Import via Hammer CLI:**

```
hammer report-template create \
  --name "Host - Compliance Report" \
  --file host-compliance-report.erb \
  --organizations "Your Org"
```

## How to Generate

**GUI:**

- Monitor → Reports → Report Templates → Host - Compliance Report → Generate
- Set Installability = applicable → Generate

**Hammer CLI:**

```
hammer report-template generate \
  --name "Host - Compliance Report" \
  --inputs "Installability=applicable"
```

**Schedule weekly email delivery:**

```
hammer report-template schedule \
  --name "Host - Compliance Report" \
  --inputs "Installability=applicable" \
  --generate-at "2026-05-19 09:00:00" \
  --mail-to "compliance@example.com" \
  --report-format csv
```

## Compatibility

- Red Hat Satellite 6.18+
- Requires Katello plugin 4.9.0+

## Why CVSS Range Instead of Exact Score

The exact CVSS score per CVE lives in the Red Hat Lightspeed Vulnerability service, which is a separate Insights-backed dashboard. The Katello ERB template engine cannot access Lightspeed data due to safe mode restrictions. The severity-to-CVSS range mapping provides a defensible compliance bucketing, and the CVE Links column allows auditors to verify exact scores with one click.
