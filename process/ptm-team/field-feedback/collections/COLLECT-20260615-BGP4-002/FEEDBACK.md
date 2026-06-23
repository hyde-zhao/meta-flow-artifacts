# ptm-tde Field Feedback Collection

| Field | Value |
|---|---|
| collection_id | `COLLECT-20260615-BGP4-002` |
| feature | `BGP4+` |
| platform | `claude` |
| workspace | `/home/hyde/projects/tcs/bpg4+` |
| result | `partial` |
| gate | `GATE-4` |
| collected_at | `2026-06-15T09:33:04+00:00` |

## Summary

MFQ分析过程中丢失了TSP分析，M分析未生成tsp/<M>-tsp.md文件，导致F/Q/integration无法按逐TSP驱动模式执行

## Expected Result

MFQ阶段产出6个TSP文件 + scenario-tsp-coverage.md + 步骤标签[M]/[F→]/[Q→]

## Actual Result

M分析跳过步骤3(TSP描述生成)和步骤2子步骤D(场景步骤标签)，直接生成测试点；F/Q分析降级为手动推理；14个交付文档因TSP缺失而连锁缺失

## Command Or Entry

```text
N/A
```

## Notes

N/A
