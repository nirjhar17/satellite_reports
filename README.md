# Satellite Custom Report Templates

Custom report templates for Red Hat Satellite 6.18+ to enhance patch compliance reporting.

## Templates

- **host-compliance-report.erb** — Compliance-focused errata report with CVSS ranges and CVE links
- **host-patch-tracking-sla-report.erb** — End-to-end patch tracking with SLA breach detection

## Host - Compliance Report

Extends the built-in "Host - Available Errata" with additional compliance-relevant columns.

**What It Adds:**

- **CVSS Range** — Maps Red Hat severity to CVSS v3 score range
- **CVE Links** — Direct URLs to Red Hat CVE pages for exact CVSS verification

**Output Columns (13 total):** Host, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Patch Release Date, Available Since, Packages, CVEs, CVE Links, Reboot suggested

## Host - Patch Tracking & SLA Report

Tracks errata from release to current state with automatic SLA breach detection. Solves two problems in a single report: end-to-end patch visibility and SLA compliance monitoring.

**What It Does:**

- Shows all applicable/installable errata per host with full timeline data
- Calculates **Days Since Release** — how many days since Red Hat published the errata
- Applies **SLA Thresholds** based on severity (Critical: 30 days, Important/Moderate: 60 days, Low: 90 days)
- Flags **SLA Breach** as YES/NO when errata exceeds the threshold and remains unapplied
- Includes **CVSS Range** mapping (same as the compliance report)

**Output Columns (15 total):** Host, Operating System, Environment, Erratum, Type, Severity, CVSS Range, Released, Available Since, Status, Days Since Release, SLA Threshold, SLA Breach, CVEs, Packages

**SLA Threshold Defaults:**

- Critical → 30 days
- Important → 60 days
- Moderate → 60 days
- Low → 90 days

These thresholds are defined at the top of the template and can be adjusted to match your organization's patching policy.

## CVSS Range Mapping

Both templates use Red Hat's official severity-to-CVSS classification:

- Critical → 9.0–10.0
- Important → 7.0–8.9
- Moderate → 4.0–6.9
- Low → 0.1–3.9

Source: [Red Hat Severity Ratings](https://access.redhat.com/security/updates/classification/)

## How to Install

**Option A — Clone from GUI:**

- Navigate to Monitor → Reports → Report Templates
- Click "Create Template"
- Paste the ERB content from the desired template file
- Go to the Inputs tab and add the required inputs (see template header for input definitions)
- Click Submit

**Option B — Import via Hammer CLI:**

```
hammer report-template create \
  --name "Host - Compliance Report" \
  --file host-compliance-report.erb \
  --organizations "Your Org"

hammer report-template create \
  --name "Host - Patch Tracking & SLA Report" \
  --file host-patch-tracking-sla-report.erb \
  --organizations "Your Org"
```

**Note:** When importing via `hammer`, you must manually add the template inputs in the Satellite GUI (Edit Template → Inputs tab) since `hammer` does not auto-parse input definitions from the ERB header.

## How to Generate

**GUI:**

- Monitor → Reports → Report Templates → select template → Generate
- Set Installability = applicable (or installable) → Generate

**Hammer CLI:**

```
hammer report-template generate \
  --name "Host - Patch Tracking & SLA Report" \
  --inputs "Installability=applicable,Errata filter=,Hosts filter="
```

**Schedule weekly email delivery:**

```
hammer report-template schedule \
  --name "Host - Patch Tracking & SLA Report" \
  --inputs "Installability=applicable" \
  --generate-at "2026-05-19 09:00:00" \
  --mail-to "compliance@example.com" \
  --report-format csv
```

## Compatibility

- Red Hat Satellite 6.18+
- Requires Katello plugin 4.9.0+

## Safemode Considerations

Satellite ERB templates run in Ruby safemode, which restricts access to certain classes like `Date` and `Time`. These templates work around this by:

- Using `to_i` (epoch seconds) for date arithmetic instead of `Date.today` or `Time.now`
- Calculating days elapsed as `(available_since.to_i - released.to_i) / 86400`
- Avoiding `begin/rescue/end` blocks (not allowed in safemode)

## Why CVSS Range Instead of Exact Score

The exact CVSS score per CVE lives in the Red Hat Lightspeed Vulnerability service, which is a separate Insights-backed dashboard. The Katello ERB template engine cannot access Lightspeed data due to safemode restrictions. The severity-to-CVSS range mapping provides a defensible compliance bucketing, and the CVE Links column (in the compliance report) allows auditors to verify exact scores with one click.
