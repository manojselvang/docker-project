# Docker Writable Layer Demo (overlay2)

## 🔧 Environment

* Container runtime: Docker
* Storage driver: overlay2

---

## 🚀 Steps

### 1. Run a container

```bash
docker run -dit --name data-miner alpine sh -c "sleep 3600"
```

---

### 2. Verify container is running

```bash
docker ps
```

---

### 3. Create a file inside container (Writable Layer)

```bash
docker exec data-miner sh -c "echo FOUND_ME > /evidence.txt"
```

---

### 4. Verify file inside container

```bash
docker exec data-miner cat /evidence.txt
```

Expected output:

```
FOUND_ME
```

---

### 5. Inspect container storage

```bash
docker inspect data-miner
```

Look for:

```json
"GraphDriver": {
  "Data": {
    "UpperDir": "...",
    "LowerDir": "...",
    "MergedDir": "..."
  }
}
```

---

### 6. Locate Writable Layer on Host

From `UpperDir`:

```
/var/lib/docker/overlay2/<container-layer-id>/diff
```

---

### 7. Verify file on HOST

```bash
sudo cat /var/lib/docker/overlay2/<container-layer-id>/diff/evidence.txt
```

Expected output:

```
FOUND_ME
```

---

### 8. Delete container (clean-up)

```bash
docker rm -f data-miner
```

---

### 9. Check if writable layer still exists

```bash
ls /var/lib/docker/overlay2/
```

👉 Observation:

* The specific layer directory disappears (or becomes orphaned and cleaned later)

---

### 10. Try accessing file again (should fail)

```bash
sudo cat /var/lib/docker/overlay2/<container-layer-id>/diff/evidence.txt
```

👉 Result:

* File is gone ❌

---

## 🧠 Key Concepts

* Containers use a **layered filesystem (overlay2)**
* Image layers → Read-only (`LowerDir`)
* Container layer → Writable (`UpperDir`)
* All runtime changes go to **Writable Layer**

---

## ⚠️ Important Insight (Interview Gold)

* Writable layer is **ephemeral**
* Deleting container = **data loss**
* Not suitable for persistence

---

## ✅ Solution for Persistence

Use:

* Docker Volumes
* Bind Mounts

Example:

```bash
docker run -dit -v myvolume:/data alpine
```

---

## 🔥 One-Line Summary

> "Docker containers store runtime changes in the writable layer (UpperDir), which is ephemeral and deleted along with the container unless external storage like volumes is used."
