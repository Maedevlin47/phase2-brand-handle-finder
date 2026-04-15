# Brand Handle Finder

A Claude Code-native tool that finds and verifies the official Instagram and X (Twitter) handles for a list of brands — no API key required.

---

## Overview

Brand Handle Finder is a research tool that automates the tedious work of tracking down official social media handles for brands. You give it a list of brand names, and it searches the web, cross-references sources, and returns verified handles with a confidence score.

All research is performed by Claude Code's built-in web search — no external APIs, no scraping scripts, no infrastructure to manage.

---

## Features

- Finds official Instagram and X (Twitter) handles for any brand
- Scores confidence from 0–100 based on verification signals
- Checks brand websites, follower counts, verified badges, and cross-source confirmation
- Flags results for manual review when confidence is low
- Processes brands in bulk from a CSV file
- Resumes automatically if interrupted — no work is lost
- Single-brand interactive mode for quick lookups

---

## Installation / Setup

**Prerequisites:**

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Python 3.10 or higher

**Steps:**

```bash
# 1. Clone the repo
git clone <repo-url>
cd brand_social_finder

# 2. Install dependencies
pip install -r requirements.txt
```

That's it. No API keys or environment variables needed.

---

## Usage

All commands are run inside Claude Code.

### Batch mode (recommended for lists)

**1. Create your input file** — save it as `to-process.csv` in the project root:

```csv
Name,List,Location
Biti's,Vietnamese Footwear Brands,Vietnam
Bata,Global Footwear,Czechia
Muji,Japanese Retail Brands,Japan
```

| Column | Required | Description |
|---|---|---|
| `Name` | Yes | Brand name to look up |
| `List` / `Source` / `Source List` | No | Name of the source list (header is case-insensitive) |
| `Location` | Yes | Country or region |

**2. Run the batch command:**

```
/process-brands-csv
```

Progress is logged to the terminal as each brand is processed. Results are written to `results.csv`.

If the run is interrupted, re-run `/process-brands-csv` — it will pick up where it left off.

---

### Single brand lookup

```
/find-brand-handle Nike
/find-brand-handle "Biti's"
```

Returns one result row with handles, confidence score, and notes.

---

## Example Output

Results are written to `results.csv`:

| brand_name | source_name | country | website | instagram_handle | x_handle | manual_review | confidence_score | confidence_signals | notes |
|---|---|---|---|---|---|---|---|---|---|
| Nike | Global Footwear | USA | nike.com | nike | Nike | false | 95 | verified_badge; website_link; multiple_sources | Instagram and X handles found in website footer. Verified badge confirmed on both accounts. |
| Biti's | Vietnamese Footwear | Vietnam | bitis.com.vn | bitisshoes | - | false | 55 | handle_matches_brand; active_account | Instagram handle found via web search with matching brand name. No X account identified. |
| Bata | Global Footwear | Czechia | bata.com | bata | - | true | 45 | handle_matches_brand | Multiple regional Instagram accounts found. Global handle selected but manual review recommended. |

**Terminal output during batch processing:**

```
Processing 1/3: Biti's (Vietnam)…
✓ 1/3: Biti's — instagram: bitisshoes, x: -, confidence: 55
Processing 2/3: Bata (Czechia)…
✓ 2/3: Bata — instagram: bata, x: -, confidence: 45
Processing 3/3: Muji (Japan)…
✓ 3/3: Muji — instagram: muji_global, x: muji, confidence: 80
Done. 3 brand(s) processed. Results written to results.csv.
```

---

## How It Works

For each brand, the tool follows a 6-step research process:

1. **Find the official website** — searches for the brand's site and checks its footer and about/contact pages for linked social accounts (highest-confidence signal)
2. **Find Instagram candidates** — searches the web and `site:instagram.com` for matching accounts
3. **Find X candidates** — searches the web and `site:x.com` for matching accounts
4. **Select the best handle** — prioritizes: website link > global account when website-linked AND higher follower count > highest follower count > other signals (bio link, cross-source, name match, verified badge)
5. **Score confidence** — starts at 0, adds points for positive signals, deducts for ambiguity
6. **Write notes** — records where each handle was found and any caveats

**Confidence scoring:**

| Score | Meaning |
|---|---|
| 80–100 | High confidence — safe to use |
| 50–79 | Moderate — spot-check recommended |
| 0–49 | Low — manually verify before using |

| Signal | Points |
|---|---|
| Handle linked from official website | +50 |
| Profile bio links back to official website | +30 |
| Confirmed across multiple sources | +25 |
| Selected account has the highest follower count among multiple candidates | +20 |
| Handle name is exact or near-exact match to brand | +18 |
| Verified badge confirmed | +15 |
| Multiple competing accounts, no clear winner | −15 |
| Handle found via search only (not on website) | −10 |
| Follower count could not be verified | −10 |
| Selected account is a regional variant | −5 |

---

## Project Structure

```
brand_social_finder/
├── .claude/
│   └── skills/
│       └── find-brand-handle/     # Shared research logic
├── main.py                        # CSV utility helpers (read/write)
├── formatter.py                   # Output formatting helpers
├── to-process.csv                 # Input: brands to look up
├── results.csv                    # Output: results are written here
├── requirements.txt
├── CLAUDE.md                      # Project instructions for Claude
└── README.md
```

---

## Configuration

No configuration is required. The tool works out of the box.

The only customizable input is `to-process.csv` — you can name it differently and pass the filename as an argument:

```
/process-brands-csv my-brands.csv
```

---

## Limitations

- Results depend on publicly available information — private or obscure brands may return low-confidence results
- Follower counts are not always visible in search results, which reduces confidence scoring accuracy
- Regional brand variants (e.g. `nike_uk` vs `nike`) require manual review to confirm the right global account
- The tool does not support LinkedIn, TikTok, or other platforms — Instagram and X only
- Accuracy is not guaranteed; always manually verify results flagged with `manual_review: true`
