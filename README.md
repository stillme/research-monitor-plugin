# Research Monitor Plugin (`v1.4.0`)

Official plugin and skills for [Research Monitor](https://research-monitor.com), an independent nightly literature monitor covering three maintained life-science fields:

- Gut biology and mucosal immunology (`gut_biology`)
- Cancer cell therapy (`cancer_cell_therapy`)
- Single-cell foundation models and perturbation prediction (`foundation_models`)

Each finding is linked to its paper and carries its verification status against the source text. Quotes from full text are returned only where the paper's licence permits reuse.

## Install in Claude Code

```text
/plugin marketplace add stillme/research-monitor-plugin
/plugin install research-monitor@research-monitor
```

Or connect the MCP server directly:

```bash
claude mcp add --scope user --transport http research-monitor https://api.research-monitor.com/api/v1/mcp/app
```

## Connect in Claude and ChatGPT

- **Claude** (claude.ai, Desktop, mobile): add Research Monitor from the Connectors directory, or connect `https://api.research-monitor.com/api/v1/mcp/app` with OAuth sign-in.
- **ChatGPT & Codex**: install Research Monitor from the Plugins directory (the submission package mirrors `plugins/research-monitor/`).

Full setup guides and the list of all sixteen chat-app tools live at [research-monitor.com/docs/agents](https://research-monitor.com/docs/agents).

## Included skills

- `get-started` (`plugins/research-monitor/skills/get-started/SKILL.md`)
- `read-paper` (`plugins/research-monitor/skills/read-paper/SKILL.md`)
- `stress-test` (`plugins/research-monitor/skills/stress-test/SKILL.md`)

## Privacy and data boundaries

- Sign-in uses OAuth 2.0 with PKCE (`S256`). Read access (`mcp:read`) is separated from write access (`kb:write`), which stays off unless you opt in at consent.
- `read-paper` reads PDFs and manuscripts locally on your own model and sends only the paper title, optional DOI or PMID, optional gene or pathway terms, and up to eight extracted one-sentence claims to `place_paper` (never the PDF or manuscript body). Those strings are sent to Research Monitor and its embedding provider to be matched and are not kept unless you explicitly confirm saving the paper to your Brought in list with `save_brought_paper`.
- `stress-test` sends only the numbered claims you approve (never your draft) to `stress_test`, where they are sent to Research Monitor and its model providers to be judged. The claims are removed from the round's record once read for judging, and the record expires one hour after the round starts.

## Links

- Documentation: [research-monitor.com/docs/agents](https://research-monitor.com/docs/agents)
- Privacy Policy: [research-monitor.com/privacy](https://research-monitor.com/privacy)
- Terms of Service: [research-monitor.com/terms](https://research-monitor.com/terms)
- Support: [contact@research-monitor.com](mailto:contact@research-monitor.com)
