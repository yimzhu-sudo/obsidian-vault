---
title: analyze_models
date: 2026-07-04
tags:
  - python
  - openrouter
  - analysis
---

#!/usr/bin/env python3
import urllib.request, json, sys, time

url = "https://openrouter.ai/api/v1/models"
req = urllib.request.Request(url)
with urllib.request.urlopen(req, timeout=30) as resp:
    data = json.loads(resp.read().decode())

models = data.get("data", [])

# Sort by created desc
models.sort(key=lambda m: m.get("created", 0), reverse=True)

print(f"=== TOTAL MODELS: {len(models)} ===\n")

# Q1: Top 15 newest
print("=== TOP 15 NEWEST MODELS ===")
for i, m in enumerate(models[:15]):
    created = m.get("created", 0)
    p = m.get("pricing", {})
    is_free = (p.get("prompt") == "0" and p.get("completion") == "0")
    free_str = "YES" if is_free else "NO"
    date_str = time.strftime('%Y-%m-%d %H:%M UTC', time.gmtime(created))
    print(f"  {i+1}. {m['id']}")
    print(f"     Created: {date_str} (ts={created})")
    print(f"     Free: {free_str}")
    print(f"     Prompt/Completion pricing: {p.get('prompt','?')}/{p.get('completion','?')}")
    print()

# Q2: All free models
print("\n=== ALL FREE MODELS (prompt=='0' AND completion=='0') ===")
free_models = []
for m in models:
    p = m.get("pricing", {})
    if p.get("prompt") == "0" and p.get("completion") == "0":
        free_models.append(m)
        created = m.get("created", 0)
        date_str = time.strftime('%Y-%m-%d %H:%M UTC', time.gmtime(created))
        print(f"  - {m['id']} (created: {date_str})")
print(f"\nTotal free models: {len(free_models)}")

# Q3: nvidia/nemotron-3-ultra-550b-a55b:free
print("\n=== Q3: nvidia/nemotron-3-ultra-550b-a55b:free ===")
m3 = next((m for m in models if m['id'] == 'nvidia/nemotron-3-ultra-550b-a55b:free'), None)
if m3:
    p3 = m3.get("pricing", {})
    is_free3 = (p3.get("prompt") == "0" and p3.get("completion") == "0")
    print(f"  EXISTS: YES")
    print(f"  Name: {m3.get('name')}")
    print(f"  Pricing: prompt={p3.get('prompt')}, completion={p3.get('completion')}")
    print(f"  Is Free: {'YES' if is_free3 else 'NO'}")
else:
    print("  EXISTS: NO (not found in model list)")

# Q4: nvidia/nemotron-3.5-content-safety:free
print("\n=== Q4: nvidia/nemotron-3.5-content-safety:free ===")
m4 = next((m for m in models if m['id'] == 'nvidia/nemotron-3.5-content-safety:free'), None)
if m4:
    p4 = m4.get("pricing", {})
    is_free4 = (p4.get("prompt") == "0" and p4.get("completion") == "0")
    print(f"  EXISTS: YES")
    print(f"  Name: {m4.get('name')}")
    print(f"  Pricing: prompt={p4.get('prompt')}, completion={p4.get('completion')}")
    print(f"  Is Free: {'YES' if is_free4 else 'NO'}")
else:
    print("  EXISTS: NO (not found in model list)")

# Q5: Already printed above as TOTAL MODELS

# Q6: Models in last 7 days from reference timestamp ~1770070000
ref_ts = 1770070000
window_start = ref_ts - (7 * 24 * 3600)
window_end = ref_ts
print(f"\n=== Q6: MODELS IN WINDOW [{window_start}, {window_end}] ===")
print(f"    (7 days before reference timestamp {ref_ts})")
print(f"    Window: {time.strftime('%Y-%m-%d %H:%M UTC', time.gmtime(window_start))} to {time.strftime('%Y-%m-%d %H:%M UTC', time.gmtime(window_end))}")
recent = [m for m in models if window_start <= m.get("created", 0) <= window_end]
if recent:
    for m in recent:
        created = m.get("created", 0)
        date_str = time.strftime('%Y-%m-%d %H:%M UTC', time.gmtime(created))
        print(f"  - {m['id']} (created: {date_str})")
else:
    print("  No models found in this time window.")
print(f"\n  Count: {len(recent)}")
