# whiteboard +export（导出画板）

> **前置条件：** 先阅读 [`../../lark-shared/SKILL.md`](../../lark-shared/SKILL.md) 了解认证、全局参数和安全规则，并按目标资源权限确定 `<identity>` 为 `user` 或 `bot`。以下命令在同一条操作链中复用该身份。

导出画板内容，支持导出为预览图片、SVG 矢量图、提取 PlantUML/Mermaid 代码，或获取飞书 OpenAPI 原生画板节点格式。

## 参数

| 参数                   | 必填 | 说明                                                                     |
|----------------------|----|------------------------------------------------------------------------|
| `--whiteboard-token` | 是  | 画板 token，需要拥有画板的读权限                                                    |
| `--output-type`      | 是  | 输出格式：`preview`（预览图片）、`svg`（SVG 矢量图）、`source`（PlantUML/Mermaid 代码）、`raw`（OpenAPI 原生画板节点格式） |
| `--output`           | 否  | 输出路径。当 `--output-type preview` 时必填，推荐传入无后缀路径；当 `--output-type svg/source/raw` 时可选，不填则直接输出到终端 |
| `--overwrite`        | 否  | 覆盖已存在的文件，默认为 false                                                     |

这里的 `+export --overwrite` 只覆盖 `--output` 指向的本地输出文件，不改变远端画板。它与 [`+update --overwrite`](./lark-whiteboard-update.md#replace非空画板) 的整板替换语义完全不同。

## 输出格式

- `preview`：预览图片。推荐使用 `--output ./preview` 这类无后缀路径；CLI 会根据实际 `Content-Type` 补齐 `.png` 或 `.jpg`。
- `svg`：导出画板为标准 SVG 矢量图。可用于 SVG 编辑后回写画板（见 [`routes/svg-edit.md`](../routes/svg-edit.md)）。注意：导出为纯视觉快照，思维导图层级、表格结构、连接器绑定等语义信息会丢失。
- `source`：PlantUML/Mermaid 代码。仅限画板内有且仅有一个 PlantUML/Mermaid 图时，才可导出代码，否则会在返回值中告知不存在/有多个节点。
- `raw`：飞书 OpenAPI 原生画板节点格式。用于判定 board state、定位 `data.nodes[].id` 和核对字段。已知 node id 的局部修改用 [`+node-update`](./lark-whiteboard-node-update.md)，删除用 [`+node-delete`](./lark-whiteboard-node-delete.md)；不要把完整导出快照手改后交给 `+update raw`，因为它会创建新节点并重新分配 ID。复杂设计/修改参考 [渲染 & 写入画板](./lark-whiteboard-workflow.md#渲染--写入画板)。

## 示例

### 示例 1：导出画板为预览图片

```bash
lark-cli whiteboard +export \
  --whiteboard-token "wbcnxxxxxxxx" \
  --output-type preview \
  --output ./preview \
  --as <identity>
```

### 示例 2：提取画板中的代码并直接输出

```bash
lark-cli whiteboard +export \
  --whiteboard-token "wbcnxxxxxxxx" \
  --output-type source \
  --as <identity>
```

### 示例 3：导出画板为 SVG 矢量图

```bash
lark-cli whiteboard +export \
  --whiteboard-token "wbcnxxxxxxxx" \
  --output-type svg \
  --output ./whiteboard.svg \
  --as <identity>
```

### 示例 4：导出画板原始节点结构到文件

```bash
lark-cli whiteboard +export \
  --whiteboard-token "wbcnxxxxxxxx" \
  --output-type raw \
  --output ./nodes.json \
  --overwrite \
  --as <identity>
```
