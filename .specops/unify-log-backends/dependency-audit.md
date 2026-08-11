# Dependency Audit: Unified local log backends

**Verified:** 2026-08-11T11:20:19Z
**Threshold:** medium
**Result:** PASS

## Dependency Inventory

| Package | Version | Ecosystem | Source |
| --- | --- | --- | --- |
| Grafana Alloy | 1.18.1 | container | official GitHub release |
| Grafana | 13.1.3 | container | official GitHub release |
| Loki | 3.7.6 | container | official GitHub release |
| VictoriaLogs | 1.52.0 | container | official GitHub release |

## CVE Scan Results

No package-manager ecosystem is present in this component. The selected Alloy release includes dependency upgrades for two published Go advisories; live image compatibility is verified during implementation.

## EOL Status

All four versions are the current stable upstream releases as of 2026-08-11.

## Verification Method

- Layer 1: no package-manager audit applies
- Layer 2: official upstream release pages checked
- Layer 3: not needed

## Allowed Advisories

None.
