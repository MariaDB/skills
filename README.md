# MariaDB Agent Skills

Skills for AI agents working with MariaDB.

> Open for improvements and more skills. Contributions welcome.

## Skills

### vector

Best practices for using MariaDB's built-in vector support for AI and semantic search applications. Covers RAG patterns, vector indexes, distance functions, embedding model selection, and framework integrations (LangChain, LlamaIndex, Spring AI). Unlike MySQL (no vector support) or PostgreSQL (which requires pgvector), MariaDB has native vector support since version 11.7 — no extensions or plugins needed.

**Install per project** (via npx skills):
```
npx skills add robertsilen/mariadb-skills/vector
```

**Install globally in Claude Code:**
```bash
mkdir -p ~/.claude/skills/vector
curl -o ~/.claude/skills/vector/SKILL.md \
  https://raw.githubusercontent.com/robertsilen/mariadb-skills/main/vector/SKILL.md
```

## About

These skills teach AI agents the MariaDB-specific knowledge they need to give correct advice — particularly where MariaDB differs from MySQL or PostgreSQL defaults.

Skills are maintained by [@robertsilen](https://github.com/robertsilen). Contributions and feedback welcome via issues.
