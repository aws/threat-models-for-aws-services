# Threat Models for AWS Services

Release version 1.2.0, 10 services.

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

| Service | Threats | Mitigations | Implementations | Applicable mitigation goals |
|---|---|---|---|---|
| [bedrock-agentcore](reports/services/bedrock-agentcore.md) | 24 | 15 | 24 | 7 of 38 |
| [bedrock-agentcore-control](reports/services/bedrock-agentcore-control.md) | 148 | 117 | 300 | 24 of 38 |
| [dsql](reports/services/dsql.md) | 17 | 14 | 37 | 13 of 38 |
| [dynamodb](reports/services/dynamodb.md) | 28 | 24 | 90 | 16 of 38 |
| [ecs](reports/services/ecs.md) | 93 | 59 | 160 | 17 of 38 |
| [efs](reports/services/efs.md) | 27 | 22 | 65 | 21 of 38 |
| [eks](reports/services/eks.md) | 64 | 45 | 122 | 20 of 38 |
| [lambda](reports/services/lambda.md) | 52 | 39 | 116 | 21 of 38 |
| [s3](reports/services/s3.md) | 41 | 32 | 124 | 20 of 38 |
| [sns](reports/services/sns.md) | 26 | 21 | 65 | 16 of 38 |
