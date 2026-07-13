# CR-164 Rollback

Rollback is source-level and additive: remove the three new evidence modules, remove the optional projection hooks from the three existing consumers, and remove the seven CR-164 research tests/provenance entries. Preserve all pre-existing admission fail-closed behavior.

Trigger rollback if a mandatory method can yield aggregate PASS while another method is failed/blocked/unavailable, if raw-count DSR is exposed as effective-count DSR, if an untrusted serialized mapping is accepted as trusted evidence, or if any runtime authorization is improved by this evidence.

After rollback, run the existing CR151/CR154/CR155 admission regression. No data migration or historical artifact rewrite is required because CR-164 performed no real run and persisted no production evidence.
