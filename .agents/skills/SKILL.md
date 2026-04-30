---
name: add-image-sync
description: Add Docker image sync tasks to sync/images.yaml. Use when user requests to add image sync, sync images, add mirror sync, add a mirror synchronization, or mentions synchronizing/mirroring Docker images. Supports various formats like 'nginx:1.21', 'postgres:14', 'gcr.io/kaniko-project/executor:debug'.
---

## Workflow

1. Parse image reference from user input (auto-add `docker.io/library/` prefix if missing)
2. Generate YAML config entry mapping source image to Aliyun registry
3. Append to `sync/images.yaml`
4. Show git diff
5. Ask for confirmation to push
6. Commit and push (with proxy fallback if network issues)

## Output Format

```yaml
<source-registry>/<path>/<image>:<tags>:
  - "registry.cn-shenzhen.aliyuncs.com/lipaysam/<image-name>"
```

## Examples

Input: `postgres:14`
Output:
```yaml
docker.io/library/postgres:14:
  - "registry.cn-shenzhen.aliyuncs.com/lipaysam/postgres"
```

Input: `gcr.io/kaniko-project/executor:debug`
Output:
```yaml
gcr.io/kaniko-project/executor:debug:
  - "registry.cn-shenzhen.aliyuncs.com/lipaysam/kaniko"
```
