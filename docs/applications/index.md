---
title: applications
hide:
  - navigation
---

# Applications

```python exec="on"
import os

folder = "docs/applications"
for f in sorted(os.listdir(folder), reverse=True):
    if f.endswith(".md") and f != "index.md":
        title = f.replace(".md", "").replace("-", " ").title()
        print(f"- [{title}]({f})")
```
