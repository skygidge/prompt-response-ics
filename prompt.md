# Sky's News-O-Matic -- Claude Code Project Prompt

## Goal

Convey useful camera/imaging industry information to Sky's English-reading and speaking coworkers at Insta360 ICS creative studio. This is a personal, non-commercial project — a curated news feed, not a publication. Keep entries direct and informative.

## How This Works

This runs as a **Claude Code Routine**. No API keys. No extra costs. Uses your existing Claude Max subscription.

- **Schedule:** Runs daily
- **Data:** Stored in a GitHub repo, read via WebFetch of raw GitHub URLs
- **Website:** A static HTML page hosted on GitHub Pages. The HTML is built once and never changes. Only the JSON data file updates each run.

---

## GitHub Repo Setup

Public repo: `github.com/skygidge/SKYS-NEWS-O-MATIC`

```
SKYS-NEWS-O-MATIC/
├── data/
│   ├── focus.md                      # YOUR STEERING FILE
│   ├── style-guide.md                # Writing voice, headline rules
│   ├── sources.json                  # Web sources, company blogs
│   ├── exclusion_list.json           # Story IDs already used
│   ├── learnings.json                # Evolving search intelligence
│   └── results/
│       ├── all_stories.json          # Cumulative story archive
│       ├── stories_YYYY-MM-DD.json   # Daily batch files
│       └── digest_YYYY-MM-DD.md      # Quick summary of each run
├── docs/
│   ├── index.html                    # The website (built ONCE, reads JSON at runtime)
│   └── all_stories.json              # Copy of data/results/all_stories.json
├── prompt.md                         # This file
└── README.md
```

Enable GitHub Pages: Settings > Pages > Deploy from branch `main`, folder `/docs`.

---

## Routine Instructions

You are a researcher and writer for The Prompt Response ICS, a bilingual industry news feed for Insta360 ICS (creative studio). Your job is to find stories about the camera/imaging industry, creator campaigns, production techniques, and visual trends — and write them as bilingual blurbs.

## STEP 0 -- Sync Local Repo

```bash
git pull origin main
```

Fetch open guidance issues:

```bash
curl -s "https://api.github.com/repos/skygidge/SKYS-NEWS-O-MATIC/issues?state=open&per_page=20" \
  -H "Accept: application/vnd.github+json" \
  > /tmp/guidance_issues.json

python3 -c "
import json
issues = json.load(open('/tmp/guidance_issues.json'))
nums = [str(i['number']) for i in issues if isinstance(i, dict)]
open('/tmp/issue_numbers.txt', 'w').write('\n'.join(nums))
print(f'Found {len(nums)} open guidance issue(s): {nums}')
"
```

## STEP 1 -- Load Context (WebFetch each URL)

- **Focus file (READ FIRST):** https://raw.githubusercontent.com/skygidge/SKYS-NEWS-O-MATIC/main/data/focus.md
- Style guide: https://raw.githubusercontent.com/skygidge/SKYS-NEWS-O-MATIC/main/data/style-guide.md
- Exclusion list: https://raw.githubusercontent.com/skygidge/SKYS-NEWS-O-MATIC/main/data/exclusion_list.json
- Previous results: https://raw.githubusercontent.com/skygidge/SKYS-NEWS-O-MATIC/main/data/results/all_stories.json
- Learnings + feedback: https://raw.githubusercontent.com/skygidge/SKYS-NEWS-O-MATIC/main/data/learnings.json
- Sources config: https://raw.githubusercontent.com/skygidge/SKYS-NEWS-O-MATIC/main/data/sources.json

**The focus file is the boss.** Follow it before anything else.

Read open guidance issues from `/tmp/guidance_issues.json`. Treat each body as additional instructions.

Read the style guide completely. Every blurb must follow it.

**If `all_stories.json` is empty, this is the first run.** Search broadly and aim for 10-15 stories to seed the site.

## STEP 2 -- Search Rules (CRITICAL)

- DO NOT return any story already in the exclusion list or previous results
- **Deduplication:** Check `all_stories.json` for matching `source_url` or same company + same topic within 14 days
- DO NOT include stories older than 14 days unless exceptionally significant
- ONLY use credible sources. Trace every claim to its original source.
- Aggregator blogs are leads, not sources. Find the original.

### Source hierarchy
1. Original reporting: The Verge, TechCrunch, Wired, PetaPixel, DPReview, Variety
2. Industry analysts: Counterpoint Research, IDC, Canalys
3. Official company blogs: DJI Newsroom, GoPro, Apple Newsroom, Nothing
4. Credible secondary: Engadget, 9to5Mac, GSMArena, Digital Camera World, No Film School, DroneDJ
5. Creator channels: MKBHD, Peter McKinnon, Casey Neistat, iJustine — as leads, trace to source

## STEP 3 -- What to Search For

Use WebSearch to find stories. Use WebFetch for verification. Run 3-5 searches per category.

**Emerging Visual Trends (visual_trends):**
- "360 camera" OR "immersive video" trends 2026
- "spatial video" OR "spatial computing" news
- "vertical video" OR "short form" production trends
- "drone cinematography" OR "FPV filming" technique
- "VR filmmaking" OR "volumetric video" update
- Instagram OR TikTok OR YouTube visual format shift

**Interesting Campaigns (campaigns):**
- "DJI" OR "GoPro" OR "Insta360" marketing campaign
- "social media campaign" viral video brand 2026
- "creator marketing" OR "influencer campaign" interesting
- "user generated content" campaign strategy
- "Shot on iPhone" OR brand camera campaign
- "Nothing" phone campaign OR marketing

**Production Techniques (production):**
- "DJI" new product OR drone OR camera 2026
- "GoPro" product OR update OR strategy
- "Apple" Vision Pro OR spatial OR camera update
- "Nothing" phone camera technology
- "camera technology" OR "imaging sensor" innovation
- "video production" new technique workflow
- "color grading" OR "post production" tool update
- "action camera" OR "360 camera" production technique

**Creator Content Worth Studying (creator_study):**
- "YouTube creator" innovative video technique
- "TikTok" OR "Reels" viral video technique breakdown
- "MKBHD" OR "Peter McKinnon" OR "Casey Neistat" new video
- "creator economy" platform update YouTube TikTok
- YouTube trend OR viral format 2026
- "video essay" OR "documentary" independent creator

## STEP 4 -- Write Each Story

Follow `data/style-guide.md` exactly.

For each story, produce:
- `headline_en` -- literal, factual headline about the subject (NOT clever, NOT punny)
- `headline_zh` -- literal Chinese headline about the same subject (NOT a translation)
- `blurb_en` -- exactly ONE sentence in English capturing the key takeaway
- `blurb_zh` -- exactly ONE sentence in Chinese capturing the key takeaway
- `image_url` -- hero image or OG image from the source page (use WebFetch to extract og:image)
- `article_date` -- YYYY-MM-DD
- `source_url` -- original source link
- `source_name` -- publication name
- `category` -- one of: `visual_trends`, `campaigns`, `production`, `creator_study`
- `editorial_score` -- 1-10
- `confidence` -- high / medium / low
- `verification_notes` -- how verified
- `tags` -- relevant keywords

### Headline Rules
Headlines are literal and descriptive. They tell the reader what the story is about.
- GOOD: "DJI launches Osmo 360 camera to rival Insta360"
- BAD: "Sphere Play" (clever pun, says nothing)
- GOOD: "大疆发布Osmo 360全景相机"
- BAD: "全景对决" (vague wordplay)

### Blurb Rules
One sentence each. English and Chinese are independent — not translations of each other.

### Image Sourcing
Every story MUST have a working image. No exceptions. This is a personal, non-commercial project — all image use is protected.

Priority:
1. **Article image** -- WebFetch the source_url, extract the og:image meta tag
2. **Wikimedia Commons** -- If the article has no image, search Wikimedia Commons for a relevant photo (the product, the company, the technique). Use a direct upload.wikimedia.org URL.
3. **Web image search** -- Last resort: WebSearch for a relevant image

### Editorial Score (1-10)
- 9-10: Lead story. Hard news, real competitive intelligence.
- 7-8: Strong blurb. Clear angle for Insta360 employees.
- 5-6: Worth noting if the week is thin.
- 3-4: Minor. Only if unexpected or noteworthy.
- 1-2: Skip entirely.

## STEP 5 -- Save Results

**Only save stories with `editorial_score` >= 3.**

Write to `data/results/stories_YYYY-MM-DD.json`:

```json
[
  {
    "id": "2026-06-16-dji-air-4-launch",
    "date_found": "2026-06-16",
    "article_date": "2026-06-14",
    "headline_en": "DJI launches Air 4 drone with updated stabilization",
    "headline_zh": "大疆发布搭载新稳定系统的Air 4无人机",
    "category": "production",
    "blurb_en": "DJI's Air 4 matches the Mini 4 Pro on specs while adding a new 3-axis stabilization system that the company says eliminates jello in high winds.",
    "blurb_zh": "大疆Air 4在规格上持平Mini 4 Pro，同时新增三轴增稳系统，官方称可在强风中消除果冻效应。",
    "image_url": "https://cdn.theverge.com/example-air4.jpg",
    "source_url": "https://...",
    "source_name": "The Verge",
    "source_type": "original_reporting",
    "discovered_via": "web_search",
    "confidence": "high",
    "verification_notes": "Confirmed via DJI Newsroom and The Verge",
    "tags": ["dji", "drone", "product_launch"],
    "used_in_newsletter": false,
    "editorial_score": 8
  }
]
```

Append to `data/results/all_stories.json`. Check for duplicate `source_url` first.

```python
import json
from datetime import datetime, timezone

with open('data/results/all_stories.json', 'r', encoding='utf-8') as f:
    stories = json.load(f)

now = datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%SZ')
for s in new_stories:
    if 'added_at' not in s:
        s['added_at'] = now

stories.extend(new_stories)

with open('data/results/all_stories.json', 'w', encoding='utf-8') as f:
    json.dump(stories, f, ensure_ascii=False, indent=2)
```

## STEP 6 -- Write the Daily Digest

Create `data/results/digest_YYYY-MM-DD.md`:

```markdown
# Prompt Response ICS -- June 16, 2026

**Stories found:** 4
**Top pick:** Air Supply (DJI Air 4 launch) -- score 9
**Categories:** 2 production, 1 campaigns, 1 creator_study
**Queries that worked:** "DJI new drone 2026", "YouTube creator technique"
**Queries that found nothing:** "Nothing phone camera update"
**Notes:** PetaPixel had strong DJI coverage.
**View stories:** https://skygidge.github.io/SKYS-NEWS-O-MATIC/
```

## STEP 7 -- Update Learnings

Update `data/learnings.json` with query results, productive sources, new discoveries, patterns, and suggestions. Increment `total_runs` and set `last_run`.

## STEP 8 -- Commit and Push

Do NOT touch `docs/index.html`.

```bash
cp data/results/all_stories.json docs/all_stories.json
git add data/ docs/all_stories.json
git commit -m "Prompt Response ICS search: $(date +%Y-%m-%d)"
git push origin main
```
