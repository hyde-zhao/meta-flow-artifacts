---
cr_id: "CR-050"
from_story: "ST-GB-004"
to_story: "ST-GB-003"
canonical_role: "meta-dev"
execution_mode: "inline-fallback"
status: "active"
context_ref: "process/context/stories/ST-GB-003.CP6.work-packet.json"
---

# CR-050 ST-GB-003 implementation handoff

ST-GB-004 已证明 2/2 projection gate。ST-GB-003 只能消费 current complete projection，仍必须 fresh reproof；recovery refs 必须先于删除，远端删除固定 artifact→project，两仓 remote 未同时 absent 前 local delete=0。
