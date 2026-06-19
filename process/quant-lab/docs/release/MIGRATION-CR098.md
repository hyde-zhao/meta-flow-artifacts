# CR098 Migration

## Migration Decision

No data, state schema, install path, dependency, platform rule, agent, skill, or external configuration migration is required.

## Compatibility Notes

- `ReadonlyGatewayClient()` default behavior remains fake/offline.
- Existing CR091 tests continue to pass.
- `QmtClient.health()` and `QmtClient.capabilities()` accept optional `authorization_ref`; existing callers remain compatible.

## Not Migrated

No `.env`, HMAC secret, account configuration, NAS package state, provider data, lake catalog, or runtime service state was migrated.
