# whiteboard +node-update

> **前置条件:** 先阅读 [`../../lark-shared/SKILL.md`](../../lark-shared/SKILL.md) 了解认证、全局参数和安全规则。命令同时支持 `user` 和 `bot`；按目标资源权限确定 `<identity>`，并在读取、请求预览、写入和验证中保持一致。

按 node id 更新已有节点字段。CLI 输入是批量形态，执行层会发起一次 `batch_update` OpenAPI 请求。

## 适用场景

- 已经知道 `whiteboard-token`，且所选身份拥有画板编辑权限。
- 已经知道目标 node id。
- 需要局部修改文字、颜色、样式、位置等节点字段。

## 不适用场景

- 不知道或无法唯一确认目标 node id。
- 需要重绘复杂图表、重新布局整板，或在未知几何状态下承诺无碰撞。
- 想替换整个画板内容。

## 定位节点

先导出 raw 节点结构:

```bash
lark-cli whiteboard +export \
  --whiteboard-token <whiteboard_token> \
  --output-type raw \
  --as <identity>
```

从返回的 `data.nodes[].id` 读取目标 node id，再构造只包含该 id 和待修改字段的最小输入。省略字段表示不修改，不要用默认值填充未指定字段。

## 参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `--whiteboard-token` | 是 | 画板 token。 |
| `--source` | 是 | JSON，必须包含非空 `nodes` 数组，每个 node 必须包含 `id`。支持 `@path` 文件读取或 `-` stdin。 |
| `--idempotent-token` | 否 | 幂等 token，最少 10 个字符；非空时作为 `client_token` 随 `batch_update` 请求发送。 |

## 输入

CLI 输入保持批量形态:

```json
{
  "nodes": [
    {
      "id": "o2:5",
      "style": {
        "fill_color": "#F54A45"
      }
    }
  ]
}
```

执行时所有节点保持在同一请求中:

- `PUT /open-apis/board/v1/whiteboards/:whiteboard_id/nodes/batch_update`。
- body 为 `{"nodes": [...]}`，节点内的 `id` 会保留。
- `--idempotent-token` 非空时，query 参数带 `client_token=<token>`。

## 示例

```bash
lark-cli whiteboard +node-update \
  --whiteboard-token <whiteboard_token> \
  --source @./node-updates.json \
  --idempotent-token <10+字符唯一串> \
  --as <identity> \
  --dry-run

lark-cli whiteboard +node-update \
  --whiteboard-token <whiteboard_token> \
  --source @./node-updates.json \
  --idempotent-token <同一个幂等串> \
  --as <identity>
```

## 输出

```json
{
  "data": {
    "ids": "o2:5",
    "count": 1
  }
}
```

## Safety

- 多节点更新前先使用 `--dry-run` 检查 `batch_update` method、URL、params 和 body。dry-run 只执行本地校验并打印请求预览，不请求画板 OpenAPI。
- 请求预览、真实执行和重试复用同一 `node-updates.json`、幂等 token 和身份。
- `batch_update` 写前会统一校验，但结构性字段可能分阶段应用。如服务端提示未完整完成，必须读回所有请求 node id 确认状态。
- 真实执行失败时，只能修正同一 payload 中可验证的 schema 错误后重试一次，或停止并报告能力边界。
- 不要在节点更新失败时自动回退到 `+node-create` 遮罩、SVG Edit、raw create 或 `+update --overwrite`。只有用户另行明确要求替换整个画板时，才能进入 replace workflow。
