# Answers — append-only Q&A memory

Lightweight memory for application questions. This is the "RAG" for Phase 1 — just a file.

**Rules**
- Never overwrite. A human edit becomes a **new** entry.
- At retrieval, the newest and most company/role-specific match wins.
- Seeded from `profile.md`; grows every time you apply.

**Entry format**

```
## Q: <question exactly as it appeared on the form>
- A: <answer>
- tags: <company, role, topic keywords>
- updated: <YYYY-MM-DD>
- source: profile | human-edit | best-guess
```

---

<!-- entries below, newest at the bottom -->
