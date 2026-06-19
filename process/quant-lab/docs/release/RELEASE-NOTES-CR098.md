# CR098 Release Notes

## Release Decision

- Decision: `READY_WITH_RISK`
- Version: `0.1.1-cr098-runner-readonly-integration`
- Scope: internal runner readonly integration only

## Changes

- Added `ReadonlyGatewayRuntimeConfig` and injected `QmtClient` readonly path for runner integration.
- Kept default runner behavior fake/offline.
- Extended evidence summary with readonly health/capabilities/position summary fields.
- Added CR098 contract tests and reran CR091 / CR020 regressions.

## Known Risks

- Real runner runtime smoke was not authorized or executed.
- Non-empty positions and trading-day readonly path remain unproven.

## Not Authorized

CP8 approval does not authorize HMAC secret reads, Windows `.env`, gateway startup, runner runtime, account originals, NAS, trading write, simulation/live, provider/lake/publish, or real release execution.
