# Packweave Traits Catalog

The **Packweave Traits Catalog** is the specification-first registry of reusable
behavioral traits used across Packweave-based systems — including OTAJet, ESP32
services, offline-first agents, and future Packweave applications.

Traits describe **what a system must do**, not how it’s implemented.  
Each trait expands into concrete guarantees, required fields, and evaluation
checks. Use-cases declare the traits they need; the catalog resolves those traits
into a final, explicit `.spec` file suitable for code generation, CI validation,
and deployment governance.

This repository is intentionally simple and self-contained:
- No firmware
- No servers
- No UI
- Just **traits, specs, resolved files, and tiny tooling**

---

## Why Traits?

Traits provide a middle layer between *intent* and *implementation*:

- **Reusable** — one trait can apply to multiple use-cases  
- **Governable** — each trait defines guarantees and required fields  
- **Spec-first** — traits feed into the Packweave Spec → Plan → Tasks workflow  
- **Deterministic** — updates to a trait update all dependent specs  
- **Implementation-agnostic** — traits work across ESP32, server, and agent layers

Example trait IDs:

- `time.authority.lan`
- `time.upstream_sync.bounded_polling`
- `security.manifest_signed_only`
- `logging.udls_structured`

---

## Repository Structure

```
docs/
  Packweave-Traits-Catalog-Spec-v1.md   # Human-readable trait documentation

.specify/
  traits_catalog.yaml                    # Index of all trait definitions

  traits/
    time.yaml                            # Time/NTP-related traits
    security.integrity.yaml              # Security traits (e.g. signed manifests)
    logging.yaml                         # UDLS logging trait
    telemetry.yaml                       # Telemetry/metrics traits

  use_cases/
    time_authority_esp32.yaml            # Authoring UC with trait list
    time_authority_esp32.spec.yaml       # Final, explicit .spec (generated)

  resolved/
    use_cases/
      time_authority_esp32.resolved.yaml # Flattened guarantees/fields/evals

scripts/
  spec/
    resolved_to_spec.py                  # Tiny compiler: resolved → .spec
```

---

## The Trait → Resolved → Spec Pipeline

A use-case starts small:

```yaml
use_case:
  id: time_authority.esp32.lan_master
  traits:
    - time.authority.lan
    - time.ntp_server.udp_123
    - logging.udls_structured
```

### 1. Traits expand into a **resolved file**

```yaml
guarantees:
  - "Expose NTP on UDP/123"
  - "Compute time_health"
required_fields:
  - udls.time_health
  - udls.time_source
required_evals:
  - eval.time.monotonicity
```

### 2. The script turns it into a final `.spec` file

```bash
python scripts/spec/resolved_to_spec.py   .specify/resolved/use_cases/time_authority_esp32.resolved.yaml   .specify/use_cases/time_authority_esp32.spec.yaml
```

### 3. The `.spec` file is what downstream tools use

- Code generators  
- CI evaluation harness  
- OTAJet rollout gates  
- Device/server agents  
- Tests  
- Documentation exporters  

Traits disappear; **only their effects remain**.

---

## Current Example: ESP32 LAN Time Authority

Included in this repo is a reference use-case:

- **ESP32 LAN Time Authority**  
  A small ESP32-S3 node that serves NTP/SNTP on the LAN, computes time-health,
  logs using UDLS, and synchronizes upstream time securely.

Traits powering it:

- `time.authority.lan`
- `time.ntp_server.udp_123`
- `time.health_score.exposed`
- `time.upstream_sync.bounded_polling`
- `security.ntp_configured_sources_only`
- `logging.udls_structured`
- `ops.telemetry.time_health_logged`

This provides a clear, concrete example of how traits flow into resolved specs
and finally into `.spec` files.

---

## Roadmap

- Add a validation tool to detect unused or conflicting traits  
- Add schema for trait versioning  
- Add more examples from OTAJet and Packweave use-cases  
- Publish documentation site (MkDocs or Quartz)  
- Add GitHub Actions for auto-resolving use-cases  

---

## License

MIT. Traits and specs are intended to be used freely in open and closed systems.

---

## Contributing

Contributions are welcome.  
Traits should be:

- Small  
- Reusable  
- Intent-driven  
- Technology-agnostic  
- Accompanied by clear guarantees, fields, and eval expectations

Open issues or PRs with new traits, improvements, or examples.
