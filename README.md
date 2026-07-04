# hotel-safety-check

An AI skill for TRAE IDE / Claude Code that analyzes hotel reviews to assess female safety risks. Helps women travelers make informed decisions before booking.

## What It Does

Given a hotel URL or name, this skill:

1. **Collects reviews** from platforms like Ctrip, Dianping, Xiaohongshu
2. **Scans for safety keywords** across 30+ risk indicators (harassment, hidden cameras, lock issues, soundproofing, etc.)
3. **Evaluates 6 safety dimensions**: security facilities, door/lock safety, privacy, location, staff reliability, review sentiment
4. **Generates a structured report** with risk ratings and actionable advice

## Install

### TRAE IDE

Copy `SKILL.md` to your TRAE workspace skill directory:

```bash
mkdir -p .trae/skills/hotel-safety-check
cp SKILL.md .trae/skills/hotel-safety-check/
```

### Claude Code

Copy `SKILL.md` to your Claude Code skill directory:

```bash
mkdir -p ~/.claude/skills/hotel-safety-check
cp SKILL.md ~/.claude/skills/hotel-safety-check/
```

## Usage

Simply describe what you need in natural language:

```
Check female safety for this hotel: https://hotels.ctrip.com/hotels/480966.html
```

```
这个酒店一个人住安全吗？https://hotels.ctrip.com/hotels/123456.html
```

```
Compare safety between these two hotels for a solo female traveler:
- https://hotels.ctrip.com/hotels/480966.html
- https://hotels.ctrip.com/hotels/789012.html
```

The skill will be automatically triggered when your query involves hotel safety for women.

## Safety Assessment Dimensions

| Dimension | What It Checks |
|-----------|---------------|
| Security Facilities | Access control, CCTV, security guards |
| Door & Lock Safety | Lock quality, deadbolt, peephole, door gap |
| Privacy (Soundproofing) | Noise isolation, thin walls |
| Location Safety | Neighborhood, lighting, proximity to transit |
| Staff Reliability | 24h front desk, responsiveness, helpfulness |
| Review Sentiment | Safety-related complaints in negative reviews |

## Keyword Categories

The skill scans reviews across 30+ keywords in 4 categories:

- **Direct Safety**: harassment, hidden cameras, lock issues, break-ins
- **Indirect Safety**: soundproofing, lighting, staff attitude
- **Female-Specific**: solo travel, women alone, female guest
- **Positive Signals**: good security, helpful staff, quiet

## Data Sources

| Source | Method | Why It Matters |
|--------|--------|--------------|
| Ctrip | WebFetch / CDP | Ratings, review counts, facility info |
| Xiaohongshu | CDP (recommended) / WebSearch | **Critical** — real user safety experiences often posted here but filtered on booking platforms |
| WebSearch | Built-in | Brand history, news incidents |

## Optional Dependencies

- **web-access skill**: Required for CDP browser mode (recommended for Xiaohongshu and Ctrip anti-scraping)
- Without web-access, the skill falls back to built-in WebSearch and WebFetch tools

## Report Sample

```
## Hotel Name - Female Safety Assessment

### Verdict: SAFE / CAUTION / AVOID

### Overview
- Rating: 4.7/5
- Reviews: 1,555
- Negative reviews: 24 (1.5%)

### Keyword Scan Results
| Category    | Matches | Risk Level |
|-------------|---------|------------|
| Direct Safety | 0 | None |
| Indirect Safety | 3 | Low |
| Positive Signals | 12 | Good |

### Recommendations
- ...
```

## License

MIT
