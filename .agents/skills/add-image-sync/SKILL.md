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
   - 已有启用 key → 扩展该 key 的 tag 列表（见约束 2、4）
   - 仅有注释 key → 新增启用条目，注释历史原样保留（见约束 3、5）
   - 仓库无 key → 新增启用条目（见约束 5）
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
  完整 registry 路径原样保留。
- `TARGET` 永远不带 tag。同步按源 key 的每个 tag 逐个复制到同一 target 名下；
  要固定版本就改 `SOURCE` 的 tag 列表，不在 target 上拼 `:1.2.3`。
- 不做重命名：target 末段与源仓库末段一致，不加 `-mirror` 之类后缀。
- target 一律用单引号 `'...'`，缩进 2 空格。

示例：

```yaml
ghcr.io/org/app:v1.2.3,latest:
  - 'registry.cn-shenzhen.aliyuncs.com/lipaysam/app'
```

## 约束

1. 添加前与用户确认镜像源和 tag 名称。
2. 同一源仓库至多一个**启用**（无 `#`）key。新增 tag = 扩展现有启用 key 的 tag
   列表，绝不新建第二个启用 key；已有启用条目时向用户汇报并询问是否更新。
3. 同一源仓库允许存在多个注释 key（如 `# …dify-web:1.14.0` 与 `# …dify-web:1.15.0`），
   它们是历史归档：不合并、不改写、不删除，除非用户明确要求清理。
4. 扩展现有 key 时保留其 target 与缩进；target 行改为单引号 `'...'`，未涉及改动的
   行保持原样。
5. 新条目默认启用（无 `#` 前缀）；未涉及改动的条目保持注释状态（带 `#`），只让
   本次变化的条目处于启用状态。
6. `TARGET` 不带 tag，也不做重命名（见「条目格式」）。

## ACR 限制：OCI attestation

ACR（`registry.cn-shenzhen.aliyuncs.com`）拒绝 OCI index 里的 attestation manifest
（引用 `application/vnd.oci.empty.v1+json`），报
`denied: unknown manifest class for application/vnd.oci.empty.v1+json`，导致整个
index 推不上去、不建 tag。上游用 buildx 默认 provenance/SBOM 时必然命中。

处理：经用户同意后把源固定到单个平台 manifest digest，只同步该 manifest：

```yaml
docker.io/org/app:v1.2.3@sha256:<linux/amd64 digest>:
  - 'registry.cn-shenzhen.aliyuncs.com/lipaysam/app'
```

digest 用 `docker manifest inspect SOURCE:TAG` 取，选 `platform.architecture == amd64`
且 `os == linux` 的条目。代价是丢多架构，必须先说明。

### digest 固定源不会建 tag

`tag@digest` 只按 digest 复制 manifest，**不会在 target 上创建 tag**：同步成功后
`lipaysam/<repo>` 只有 digest，`tags/list` 里没有该版本（CI 变绿也不代表 tag 存在）。
所以要么手工补 tag，要么明确告知用户只能按 digest 拉。

补 tag（blob 与原 manifest 已在 ACR，只是新增一个 tag 引用；不要重新打包）：

```bash
D=sha256:<digest>; TAG=<version>
# 1. 取源 manifest 原始字节，并校验 sha256 等于 $D
curl -H "Authorization: Bearer $SRC_TOKEN" \
  -H 'Accept: application/vnd.oci.image.manifest.v1+json' \
  -o m.json "https://<src-registry>/v2/<org>/<repo>/manifests/$D"
sha256sum m.json
# 2. 以原 mediaType 推成 tag（ACR 需 pull,push scope token）
curl -X PUT -H "Authorization: Bearer $ACR_TOKEN" \
  -H 'Content-Type: application/vnd.oci.image.manifest.v1+json' \
  --data-binary @m.json \
  "https://registry.cn-shenzhen.aliyuncs.com/v2/lipaysam/<repo>/manifests/$TAG"
```

3. 校验 `GET /v2/lipaysam/<repo>/tags/list` 含该 tag，且 `GET …/manifests/<tag>` 的
   `docker-content-digest` 等于 `$D`。

## 验证

编辑后运行 `git diff --check`，并 review `git diff -- sync/images.yaml`，确认只有
预期改动。若本次含 `@sha256:` 源，额外确认 ACR `tags/list` 出现预期 tag。