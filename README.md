# MariaDB Agent Skills

Agent skills for [Claude Code](https://claude.ai/code) and other AI agents working with MariaDB.

## Skills

### mariadb-vector

Best practices for using MariaDB's built-in vector support for AI and semantic search applications. MariaDB has native vector support since version 11.7 — no extensions or plugins needed. Covers RAG patterns, vector indexes, distance functions, embedding model selection, and framework integrations (LangChain, LlamaIndex, Spring AI).

Install per project (via npx skills):
```
npx skills add robertsilen/mariadb-skills/mariadb-vector
```

Install globally in Claude Code:
```bash
mkdir -p ~/.claude/skills/mariadb-vector
curl -o ~/.claude/skills/mariadb-vector/SKILL.md \
  https://raw.githubusercontent.com/robertsilen/mariadb-skills/main/mariadb-vector/SKILL.md
```

## About

These skills teach AI agents the MariaDB-specific knowledge they need to give correct advice — particularly where MariaDB differs from MySQL or PostgreSQL defaults.

Skills are maintained by [@robertsilen](https://github.com/robertsilen). Contributions and feedback welcome via issues.
