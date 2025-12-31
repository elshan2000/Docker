# Docker Registry API v1 vs v2 – Differences

## Image Model Difference

### v2 Image Structure (Modern)

```
Tag
 └── Manifest
      ├── Config blob (sha256:aaa)
      ├── Layer blob (sha256:bbb)
      └── Layer blob (sha256:ccc)
```

**Explanation:**

* **Tag** → points to a **Manifest**
* **Manifest** → JSON document that references blobs
* **Blobs** → immutable content-addressed objects

  * Config blob → image metadata (env, cmd, entrypoint, arch)
  * Layer blobs → filesystem layers

---

## Conceptual Difference

| Aspect             | v1 (Old)         | v2 (Modern)                |
| ------------------ | ---------------- | -------------------------- |
| Image identity     | Image ID–based   | Content-addressed (SHA256) |
| Layers             | Implicit         | Explicit blobs             |
| Deduplication      | Weak             | Strong                     |
| Security           | No content trust | Supports signing           |
| Multi-arch         | No               | Yes                        |
| Garbage collection | Hard             | Clean & reliable           |

---

## Request-Level Difference

### v1 API (Deprecated)

```bash
/v1/images/...
```

* Monolithic image endpoints
* No clear separation of metadata vs layers
* Inefficient pulls/pushes

---

### v2 API (Current Standard)

```bash
/v2/<image_name>/manifests/<tag>
/v2/<image_name>/blobs/<digest>
```

**What happens internally:**

1. Client requests **manifest** by tag
2. Registry returns manifest JSON
3. Client pulls **config blob**
4. Client pulls **layer blobs** by digest

---

## Example

```bash
docker pull nginx:latest
```

Under the hood:

1. `GET /v2/library/nginx/manifests/latest`
2. `GET /v2/library/nginx/blobs/sha256:aaa`
3. `GET /v2/library/nginx/blobs/sha256:bbb`
4. `GET /v2/library/nginx/blobs/sha256:ccc`
