---
name: add-image-sync
description: Adds enabled Docker image mirror mappings to sync/images.yaml in this repository, preserving its source/tag and target registry format while checking duplicates. Use when a user asks to add, batch-add, or enable image mirror synchronization for Docker Hub, GHCR, GCR, Quay, or other container registries.
---

# Add Image Sync

直接编辑 `sync/images.yaml`。不动 `sync/auth.yaml`、凭据、注释和无关条目。

## 工作流

1. **确认**：添加前与用户确认每个镜像源和 tag 名称。
2. **查重**：读取文件，确认该源仓库是否已存在 key（无论注释与否）。
3. **决定动作**：
   - 仓库已有 key → 更新现有 key（见约束 2、3）
   - 仓库无 key → 新增启用条目（见约束 4）
4. **编辑**：按「条目格式」写入。
5. **验证**：`git diff --check`，并 review `git diff -- sync/images.yaml`。

## 条目格式

```yaml
SOURCE:TAG[,TAG...]:
  - 'TARGET'
```

- `SOURCE` 为完整源仓库。无 registry 的镜像归一化到 Docker Hub：`nginx` 成为
  `docker.io/library/nginx`，`org/app` 成为 `docker.io/org/app`；首段含 `.`、`:`
  或 `localhost` 的 registry（GHCR/GCR/Quay 等）原样保留。
- 同一源的多 tag 合并在一个 key 上，逗号分隔、无空格。
- `TARGET` 默认取源仓库最后一段，前缀 `registry.cn-shenzhen.aliyuncs.com/lipaysam/`；
  完整 registry 路径原样保留。同步只镜像 tag，不做重命名。

示例：

```yaml
ghcr.io/org/app:v1.2.3,latest:
  - 'registry.cn-shenzhen.aliyuncs.com/lipaysam/app'
```

## 约束

1. 添加前与用户确认镜像源和 tag 名称。
2. 一个源仓库在文件中只允许一个 key。新增 tag = 扩展现有 key 的 tag 列表，绝不
   新建第二个 key；已有活动条目时向用户汇报并询问是否更新。
3. 扩展现有 key 时保持原有 target 与书写风格（引号、缩进等）。
4. 新条目默认启用（无 `#` 前缀）；未涉及改动的条目保持注释状态（带 `#`），只让
   本次变化的条目处于启用状态。
5. 镜像同步只镜像 tag，不做重命名。

## 验证

编辑后运行 `git diff --check`，并 review `git diff -- sync/images.yaml`，确认只有
预期改动。