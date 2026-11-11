---
name: aeo-audit
description: Answer Engine Optimization (AEO) workflow. Audits what prospective buyers ask, analyzes brand recommendations across ChatGPT, Gemini, Perplexity, and Google AI Mode, reverse-engineers cited sources, and builds creator/content outreach plans.
---

# Answer Engine Optimization (AEO) Skill

A 5-step engineering and marketing workflow to audit, reverse-engineer, and optimize brand recommendations across generative AI answer engines (ChatGPT, Gemini, Perplexity, Google AI Overviews).

## Workflow Pipeline & Decision Gates

- **Step 1: Demand Discovery** (Buyer question mining & intent scoring $\rightarrow$ Gates 1 & 2)
- **Step 2: AI Recommendation Audit** (Parallel engine queries & citation metrics $\rightarrow$ Gates 3 & 4)
- **Step 3: Source Reverse Engineering** (Domain authority & opportunity scoring $\rightarrow$ Gates 5 & 6)
- **Step 4: Creator & Content Outreach** (Reddit, YouTube & owned FAQ remediation $\rightarrow$ Gates 7 & 8)
- **Step 5: Movement & Delta Audit** (3–4 weeks post-execution re-run $\rightarrow$ Gates 9 & 10)

---

## Step 01: What Customers Ask (Demand Discovery)

### Agent Instructions
1. Accept seed keywords from the user.
2. Query SE Ranking for keyword ideas and volume:
   - Endpoint: `https://api.seranking.com/v2/keywords/ideas` (GET, limit=40, sort=volume).
3. Query DataForSEO for search intent:
   - Endpoint: `https://api.dataforseo.com/v3/dataforseo_labs/google/search_intent/live`
   - Weightings: Transactional = 1.0, Commercial = 0.8, Informational = 0.2, Navigational = 0.0.
   - Formula:
     $$\text{BuyerScore} = \text{Intent} \times (0.50 C + 0.35 V + 0.15 E)$$
     where $C = \operatorname{minmax}(\text{CPC})$, $V = \operatorname{minmax}(\log_{10}(1 + \text{vol}))$, $E = 1 - (\text{diff} / 100)$.
   - Retain keywords with $\text{Intent} \ge 0.8$.
4. **Gate 1**: Present keyword candidate table with top 5 dropped rows. Await approval.
5. Ingest People Also Ask (PAA) via DataForSEO Google SERP API (`/v3/serp/google/organic/live/advanced`).
   - Formula: $\text{QuestionScore} = \max(\text{BuyerScore}) \times (1 + 0.1(F - 1))$
   - Rewrite up to 10 questions in format: `[situation] + [need] + "which should I buy?"`.
6. **Gate 2**: Present question ranking table. Await approval.

---

## Step 02: Who Does AI Recommend? (AI Audits)

### Agent Instructions
1. Run parallel queries (timeout: 300s) across target answer engines for the top approved questions:
   - **ChatGPT**: Cloro API (`/cloro/ai-search/chatgpt/scrape`)
   - **Gemini**: Cloro API (`/cloro/ai-search/gemini/scrape`)
   - **Google AI Mode**: Cloro API (`/cloro/google/serp/ai_mode`)
   - **Perplexity**: DataForSEO API (`/v3/ai_optimization/perplexity/llm_responses/live`)
2. For each query response, calculate:
   - $m$: Brand mentioned (1 or 0)
   - $p$: Position of mention ($1 = \text{1st named}$; $m/p = 0$ if unmentioned)
   - $n$: Total competing brands named
   - $c$: User's own domain cited (1 or 0)
   - **Visibility Score (VS)**:
     $$\text{VS} = \operatorname{mean}\left(100 \times (0.50 m + 0.30 \frac{m}{p} + 0.20 c)\right)$$
   - **Mention Rate**: $\frac{\sum m}{N}$
   - **Share of Voice**: $\frac{\sum m}{\sum n}$
   - **Competitor Score**: $100 \times \frac{\sum (m / p)}{N}$
3. Calculate Citation Metrics:
   - $\text{Cite} = \sum \frac{1}{\text{source\_position}}$
   - $\text{Infiltration (Vis)} = 100 \times \frac{\text{Cite}}{N}$
   - $\text{Influence} = \frac{\text{Vis}}{\max(\text{Vis})}$
   - $\text{DomainScore} = \text{Cite} \times (1 + 0.1(Q - 1))(1 + 0.1(E - 1))$
4. **Gate 3**: Display single-question test run citations.
5. **Gate 4**: Display full engine grid, competitor share of voice, and top 15 cited domains.

---

## Step 03: Reverse Engineer AI Answers

### Agent Instructions
1. Clean and normalize all cited URLs, discarding competitor sales pages and generic retailers.
2. Query Page Authority (PA) and Domain Authority (DA) via SE Ranking / Moz API (`/v2/analytics/backlinks/overview`).
3. **Gate 5**: Render scatter plot (PA vs Citation Visibility) highlighting low-difficulty, high-visibility wins.
4. Extract text from top 15 cited URLs using a headless scraper (Firecrawl / Playwright).
5. Calculate Opportunity Score (Opp):
   $$\text{Opp} = 0.40 \text{Infl} + 0.20 \text{Gap} + 0.15 \left(1 - \frac{\text{PA}}{100}\right) + 0.15 \text{Access} + 0.10 \text{Fresh}$$
   - **Gap**: 1.0 (competitor present, user absent) | 0.5 (both absent) | 0.0 (user present)
   - **Access**: 1.0 (affiliate / contact route) | 0.5 (author email) | 0.0 (no route)
   - **Fresh**: 1.0 ($\le 12\text{mo}$) | 0.5 ($\le 24\text{mo}$) | 0.0 ($> 24\text{mo}$)
6. **Gate 6**: Present stacked opportunity rankings for top 10 pages.

---

## Step 04: Get on the Map (Outreach & Content Engine)

### Agent Instructions
1. **Gate 7**: Validate user product facts and topical fit.
2. Target Review Sites: Pitch updates ($\le 120$ words) based on calculated Opp score.
3. Reddit Discovery (ScrapeCreators API: `/v1/reddit/search/posts`):
   $$\text{Impact}_{\text{reddit}} = 0.4 \operatorname{minmax}(\log_{10}(1 + \text{ups} + 2\cdot\text{comments})) + 0.3 e^{-\text{days}/180} + 0.2 \text{Rel} + 0.1 \text{Cited}$$
   Draft helpful replies with disclosure for the top 10 threads.
4. YouTube Creator Discovery (YouTube Data API v3: `/youtube/v3/search`):
   $$\text{Impact}_{\text{youtube}} = 0.4 \operatorname{minmax}(\log_{10}(1 + \text{views})) + 0.2 e^{-\text{days}/365} + 0.2 \text{Rel} + 0.2 \text{Cited}$$
   Draft creator collaboration outreach for the top 5 channels with 3 content angles.
5. Owned Page Remediation: Generate draft FAQ answers matching top questions with JSON-LD `FAQPage` schema.
6. **Gate 8**: Output prioritized action list sorted by $\text{Priority} = \frac{\text{Impact}}{\text{Effort}}$ (Effort: Reply = 1, Pitch = 2, Content = 3).

---

## Step 05: Movement & Delta Audit (3–4 Weeks Post-Execution)

### Agent Instructions
1. Load unchanged baseline from initial audit.
2. **Gate 9**: Verify completed actions and check elapsed time ($< 21\text{ days}$ warns user).
3. Compute engine noise floor over 3 runs of Question 1:
   $$\text{Noise} = \operatorname{mean}(\max(\text{VS}) - \min(\text{VS}))$$
   Disregard any $|\Delta| \le \text{Noise}$ as statistical chance.
4. Re-run Step 2 across all engines and compute $\Delta \text{VS}$, $\Delta \text{Mention Rate}$, and new cited targets.
5. **Gate 10**: Deliver final delta report comparing baseline to present metrics.
