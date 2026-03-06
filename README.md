# claude-skills

A collection of [Claude Code](https://claude.ai/claude-code) skills for empirical research workflows — focused on the kind of data work common in economics and social science: wrangling large datasets, reading documentation, and working with survey microdata.

## Installing a skill

Copy the skill folder into your project's `.claude/skills/` directory (for project-level use) or into `~/.claude/skills/` (to make it available globally across all projects):

```bash
# Project-level
cp -r .claude/skills/skill-name /your/project/.claude/skills/

# Global
cp -r .claude/skills/skill-name ~/.claude/skills/
```

Then invoke with `/skill-name` in Claude Code.

## Skills

| Skill | Invoke | Description |
|-------|--------|-------------|
| `data_dict_convert` | `/data_dict_convert path/to/codebook.pdf` | Converts a PDF data dictionary or codebook into a searchable `.md` file. Splits the PDF, extracts all text verbatim, prepends a human-readable intro header, then cleans up. Useful for large survey codebooks (e.g. BES, BHPS, Understanding Society) that you want to query with Grep across sessions. |
| `split-pdf` | `/split-pdf path/or/search-query` | Downloads or locates an academic PDF, splits it into 4-page chunks, and reads it in batches of 3 (~12 pages at a time) to avoid context crashes. Produces structured reading notes covering research question, data, methods, findings, and replication feasibility. |
