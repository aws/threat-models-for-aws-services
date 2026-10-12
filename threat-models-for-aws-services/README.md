# Threat Models for AWS Services

Release version 1.0.0, 0 services.

## Layout

| Path | Contents |
|---|---|
| `manifest.json` | Release version, and the path and SHA-256 digest of every file listed here |
| `schema/manifest.schema.json` | JSON Schema of the manifest |
| `data/mitigation-goals.json` | Mitigation goal catalog (machine-readable) |
| `data/potential-threats.json` | Potential threat catalog (machine-readable) |
| `data/threat-impacts.json` | Threat impact catalog (machine-readable) |
| `data/services/` | Per-service threat models (machine-readable) |
| `schema/mitigation-goals.schema.json` | JSON Schema of the mitigation goal catalog |
| `schema/potential-threats.schema.json` | JSON Schema of the potential threat catalog |
| `schema/threat-impacts.schema.json` | JSON Schema of the threat impact catalog |
| `schema/threat-model.schema.json` | JSON Schema of the per-service threat models |
| `reports/` | Threat model catalog and per-service threat models (human-readable) |

## Reference catalogs

- [Mitigation goals](reports/mitigation-goals.md)
- [Potential threats](reports/potential-threats.md)
- [Threat impacts](reports/threat-impacts.md)

## Services

| Service | Threats | Mitigations | Implementations | Goals covered |
|---|---|---|---|---|
