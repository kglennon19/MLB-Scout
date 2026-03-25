# MLB Line Scout v2

Single-file browser-based MLB betting model with platoon splits, player-level data, and weather-adjusted park factors. No server, no build step, no npm. Deploys to GitHub Pages.

## What's New in v2

- **Platoon splits** (18% weight) — team batting vs LHP and vs RHP from Baseball Reference, matched to today's opposing starter's handedness
- **ESPN roster fetch** — automatically captures bats R/L/S and throws R/L for every player on today's teams
- **5 data scrapers** — team stats, vs-LHP splits, vs-RHP splits, individual player batting, BallparkPal park factors
- **Player-level props** — pitcher K's, batter HR, total bases using individual stats when synced
- **REST component removed** — irrelevant in regular season per analysis
- **BR-to-ESPN abbreviation mapping** — handles ATH/OAK, KCR/KC, SDP/SD, SFG/SF, TBR/TB, WSN/WSH

## Data Architecture

### Tier 1: ESPN Auto-Fetch (zero user action)
| Data | Endpoint | What It Gets |
|------|----------|-------------|
| Game slate + odds | Scoreboard API | Games, run lines, totals, moneylines, venue |
| Starting pitchers | probables[0] | Pitcher name, ID, confirmation status |
| Individual pitcher stats | Athlete endpoint | ERA, FIP, WHIP, K/9, BB/9, IP (106 stats) |
| Team rosters + handedness | Roster endpoint | Every player bats R/L/S, throws R/L |
| Team stats | Statistics endpoint | ERA, OPS, R/G, HR for all 30 teams |

### Tier 2: Baseball Reference (3-4 copy+paste steps, daily)
| Step | Page | Scraper | Data |
|------|------|---------|------|
| 1 | leagues/majors/2025.shtml | Team Stats | ERA, FIP, OPS, R/G for all 30 teams |
| 2 | BR splits tool (vs LHP) | Splits LHP | Team OPS/SLG/HR vs left-handed pitchers |
| 3 | BR splits tool (vs RHP) | Splits RHP | Team OPS/SLG/HR vs right-handed pitchers |
| 4 | leagues/majors/2025-standard-batting.shtml | Player Batting | Individual OPS, HR, SLG for 1000+ players |

### Tier 3: BallparkPal (optional, $10/mo)
| Step | Page | Data |
|------|------|------|
| 5 | ballparkpal.com/Park-Factors.php | Weather-adjusted, batter-level daily park factors |

## Formula Weights (v2)

| Component | Weight | Source |
|-----------|--------|--------|
| Starter | 25% | ESPN athlete stats (ERA/FIP x projected IP) |
| Base | 18% | Pythagorean W% from R/G and RA/G |
| Platoon | 18% | BR splits: team OPS vs opposing starter hand |
| Bullpen | 12% | Team bullpen ERA differential |
| Lineup | 8% | Today's actual lineup quality |
| Park | 8% | Park factor x matchup interaction |
| Recency | 6% | Last-5 game scoring trend |
| Context | 5% | Day/night, interleague, divisional |

## How Platoon Works

Game: NYY @ SF with Max Fried (LHP) vs Logan Webb (RHP)

1. ESPN tells us Fried throws LEFT, Webb throws RIGHT
2. NYY batting vs LHP = .797 OPS (from BR splits scraper)
3. SF batting vs RHP = .783 OPS (from BR splits scraper)
4. Formula uses THESE split-specific numbers, not season averages
5. Park factor (Oracle Park, 0.92) suppresses HR projections
6. Result: platoon-adjusted edge that accounts for matchup context

## Deploy

1. Upload index.html to kglennon19/MLB_Agent repo root
2. Enable GitHub Pages (main branch, root)
3. Add .nojekyll file
4. Visit kglennon19.github.io/MLB_Agent/

## Verified Table IDs (from live Baseball Reference, March 2026)

| Page | Table ID | Rows | Key Columns |
|------|----------|------|-------------|
| Main stats | teams_standard_batting | 30 | runs_per_game, onbase_plus_slugging, HR, PA |
| Main stats | teams_standard_pitching | 30 | earned_run_avg, fip, whip, strikeouts_per_nine |
| Splits | split1 | 30 | team, batting_avg, slugging_perc, onbase_plus_slugging, HR |
| Player batting | players_standard_batting | 1000+ | b_pa, b_hr, b_batting_avg, b_slugging_perc |

*MLB Line Scout v2 — March 2026*
