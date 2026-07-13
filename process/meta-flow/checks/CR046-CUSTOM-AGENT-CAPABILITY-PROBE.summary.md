# CR-046 Custom Agent Capability Probe

## Conclusion

`BLOCKED_FOR_CUSTOM_AGENT_ATTESTED_IMPLEMENTATION`

- D2 configuration: `PASS` — 8 project custom-agent TOMLs parse and have stable hashes.
- D1 callable schema: `PARTIAL` — generic spawn/followup exists, but no profile selector or receipt fields.
- D0 session discovery: `UNAVAILABLE` — no list/resolve capability is exposed.
- Runtime profile/model attestation: `UNAVAILABLE`.

Configuration presence proves installation only. It does not prove the current session discovered or applied `meta-dev`, `meta-dev-debugger`, `meta-qa-critical`, or their declared models.

CP5 design review may continue, but CP6 implementation must not start while the user requires verified custom-agent execution. Restart/reload Codex after ensuring project agents are installed, use a surface that can select the agent by its `name`, and rerun the probe. A valid result must include current-session discovery and a platform-resolved runtime identity/receipt; task names, prompts and agent self-report are insufficient.
