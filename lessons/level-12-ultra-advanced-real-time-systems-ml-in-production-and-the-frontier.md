# Level 12: Ultra-advanced: real-time systems, ML in production, and the frontier

**Track:** System Design · **Level:** 12/12 · **Difficulty:** `ultra-advanced`

📚 **Today's lesson** — published 2026-07-02

## TL;DR

You've got the foundations. Now the cutting edge: real-time collaboration (Figma, Google Docs), ML systems (recommendations, search ranking), and global-scale infrastructure (multi-region, edge compute).

## Real-world analogies

- Real-time collaboration is like a group of musicians playing together. Each has their own instrument (local state) but they all hear each other and stay in sync.
- ML in production is like a weather forecast model. You train it on past data, but the real challenge is running it reliably on new data every day.

## Key concepts

### `CRDTs`

Conflict-free Replicated Data Types. Data structures designed to merge automatically. Used in Figma, Notion, Apple Notes.

### `Feature stores`

Centralized store of ML features (user_age, last_purchase_date). Avoids recomputing the same feature in 10 different models.

### `Multi-region active-active`

Your service runs in multiple regions simultaneously, each serving local users. Hard. Replication is the easy part — conflict resolution is the hard part.

### `Event sourcing`

Store every state change as an event, not the current state. Replay events to reconstruct. Powerful for audit logs and time-travel debugging.

## Code with comments

Every line has a comment. Read it slowly.

```
# === Real-time collaboration: operational transform (OT) ===
# Two users edit the same document:
# User A types 'X' at position 5
# User B types 'Y' at position 7 (before seeing A's edit)
#
# Server receives both operations:
#   op_a: insert('X', 5)
#   op_b: insert('Y', 7)
#
# Without sync: position 7 in B's view is now position 8 in server's view.
# OT transforms op_b to: insert('Y', 8) so it appears after A's edit.

# === CRDT example: G-Counter (Grow-only Counter) ===
# Each replica has its own counter. Total is the sum.
# No conflicts, no coordination needed.
#
# class GCounter:
#     def __init__(self, replicas):
#         self.counts = {r: 0 for r in replicas}
#
#     def increment(self, replica):
#         self.counts[replica] += 1
#
#     def merge(self, other):
#         for r in self.counts:
#             self.counts[r] = max(self.counts[r], other.counts.get(r, 0))
#
#     def value(self):
#         return sum(self.counts.values())

# === ML in production: the full pipeline ===
#
# [Data Sources] → [Feature Store] → [Training] → [Model Registry] → [Inference]
#                                                          ↓
#                                                   [Monitoring] (drift, accuracy)
#
# Training is 5% of ML work. The other 95% is:
#   - Data pipelines (ingest, clean, feature engineering)
#   - Serving infrastructure (low-latency inference)
#   - Monitoring (model drift, accuracy degradation)
#   - Retraining (when does the model need to be retrained?)

# === Event sourcing example ===
# Instead of: UPDATE accounts SET balance = 100 WHERE id = 1
# Store events:
#   - account.created(id=1, balance=0)         # 2026-01-01
#   - money.deposited(id=1, amount=200)        # 2026-01-15
#   - money.withdrawn(id=1, amount=100)        # 2026-02-01
# Current balance = sum(events)
# Pros: full audit trail, time-travel debugging, can rebuild any view
# Cons: complex, eventual consistency, schema evolution is hard

# === Multi-region architecture ===
#
#              [DNS (Route 53 / Cloudflare)]
#                       |
#         +-------------+-------------+
#         v                           v
#    [us-east-1]                  [ap-south-1]
#    [us-east-1] --replicate---> [ap-south-1]
#    (primary for                (primary for
#     US users)                   Asia users)
#
# Each region has its own:
#   - Load balancers
#   - App servers
#   - Database (with async cross-region replication)
#   - Cache
#
# Hard problems:
#   - Data residency (EU data must stay in EU)
#   - Cross-region latency (us-east to ap-south = ~200ms)
#   - Conflict resolution (writes to same record from two regions)
#   - Disaster recovery (region goes down, others must take over)

# === Reading list ===
# - "Designing Data-Intensive Applications" by Martin Kleppmann — the bible
# - "System Design Interview" by Alex Xu (Vol 1 & 2) — pattern catalog
# - "Web Scalability for Startup Engineers" — practical scaling
# - Google SRE book — free online, covers reliability
# - High Scalability blog (highscalability.com) — case studies
```

## Try it yourself

Pick a real-time system (chat app, collaborative editor, multiplayer game). Design the data model and conflict resolution. How do you handle a user going offline and coming back with 1000 changes?

## Common pitfalls

- ⚠️ OT/CRDTs are research-level hard. Don't try to invent your own. Use proven libraries (Yjs, Automerge, ShareDB).
- ⚠️ Event sourcing when you just need CRUD. It's powerful but complex. Use it for audit logs, not normal data.
- ⚠️ Active-active without the operational maturity. Most teams should start active-passive (one region primary, others standby).

## What's next?

🎉 You've completed the track! Time to build something real.

---

_Generated by Hermes · Aby's learning cron · Track: System Design · Level 12_