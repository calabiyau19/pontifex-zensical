---
title: websites
hide:
  - navigation
---

# Websites

```python exec="on"
import os

folder = "docs/websites"
for f in sorted(os.listdir(folder), reverse=True):
    if f.endswith(".md") and f != "index.md":
        title = f.replace(".md", "").replace("-", " ").title()
        print(f"- [{title}]({f})")
```
