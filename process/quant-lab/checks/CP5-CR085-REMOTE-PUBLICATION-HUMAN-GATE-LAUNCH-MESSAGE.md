# CP5 CR085 Remote Publication Readiness 人工门禁发起消息

请审查：`process/checkpoints/CP5-CR085-REMOTE-PUBLICATION-READINESS.md`

自动预检结论：PASS，阻断项 0。

Context Capsule Summary：`process/context/CP5-CR085-REMOTE-CLEANUP-CONTEXT.yaml`，read_profile=compact。

决策收集覆盖：已扫描 5 个来源，发现候选问题 10 个，纳入待决策 3 个；N/A / 缺失来源 0 个，原因见 checklist 的 Decision Collection Coverage。

本轮待人工决策项：3。

如果你回复 approve，表示你接受以下 3 项推荐方案，不表示授权以下禁止操作。

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣摘要 | 影响 / 风险 |
|---|---|---|---|---|---|---|
| DQ-CP5-CR085-PUB-01 | implementation | README.md 和 `docs/USER-MANUAL.md` 如何处理？ | `sanitize-docs`：先清理 / 替换文档中的 topology blockers，再重扫。 | overlay A：保留现状；overlay B：删除 README/USER-MANUAL；stop。 | 推荐兼顾文档可用性与安全；A 风险残留；B 文档缺失。 | 影响远端文档、public-safety 和后续 push 候选。 |
| DQ-CP5-CR085-PUB-02 | risk_acceptance | 4 个测试 fixture warning 是否可接受？ | 接受为非阻断 warning，保留测试。 | 删除相关测试；停止发布。 | 推荐保留测试覆盖；删除测试会降低验证能力。 | warning 仅在测试中，未发现精确密钥。 |
| DQ-CP5-CR085-PUB-03 | runtime_authorization | 本门禁是否授权远端写入？ | 不授权；只确认下一步文档处理 / 重扫路线。 | 直接授权 overlay A push；直接授权 overlay B push。 | 推荐保持 fail-closed；直接 push 需要更精确执行计划。 | 远端仍不变。 |

不授权项：

- 不授权 `git push`、`git push -u`、remote deletion、remote add / set-url / remove。
- 不授权 tag、force push、history rewrite、default branch governance。
- 不授权上传、读取、打印或保存 `.env`、token、password、private key、cookie、session。
- 不授权 data/reports 内容读取、NAS 内容操作、provider/lake/catalog、runtime、CR046 recovery、old root retirement 或 cleanup。

请回复以下三者之一：

approve

修改: <具体修改点>

reject
