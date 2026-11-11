# aeo-audit

An agent skill for Claude Code, Codex and other coding agents. It audits how ChatGPT, Gemini, Perplexity and Google AI Mode recommend your brand, finds the sources they cite, and plans the Reddit, YouTube and on-site work to get you into those answers. Every step stops at a gate for your approval.

## Demo

https://github.com/SneharupMukherjee/aeo-audit-skill/raw/main/media/demo.mp4

## Steps

1. **Demand discovery**: buyer questions scored by volume, CPC, intent and difficulty (SE Ranking, DataForSEO)
2. **AI recommendation audit**: visibility and share of voice per engine
3. **Source reverse engineering**: cited sites scored by authority and opportunity
4. **Get on the map**: Reddit threads and YouTube videos worth pitching, plus FAQ fixes
5. **Delta audit**: re-run 3 to 4 weeks later, with a noise floor

## Install

```bash
git clone https://github.com/SneharupMukherjee/aeo-audit-skill ~/.claude/skills/aeo-audit
```

Needs keys for SE Ranking, DataForSEO, Cloro and ScrapeCreators. Full spec and formulas: [docs/aeo-workflow.md](docs/aeo-workflow.md).

## Files

- `SKILL.md`: the skill the agent reads
- `docs/`: workflow spec and formulas
