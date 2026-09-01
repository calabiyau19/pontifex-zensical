---
title: linux
hide:
  - navigation
---

# Linux

```python exec="on"
import os

folder = "docs/linux"
for f in sorted(os.listdir(folder), reverse=True):
    if f.endswith(".md") and f != "index.md":
        title = f.replace(".md", "").replace("-", " ").title()
        print(f"- [{title}]({f})")
```
