# MTA CarBall Rating System - Complete Scoring Engine

## About This Document

This document provides a **complete and transparent explanation** of the CarBall player rating system.

**Purpose:** A full explanation of how player ratings are calculated.

> **Note:** This is not an executable software project, but rather documentation for the computing engine. The logic is explained in detail so that anyone can verify and understand the system.

---

## SYSTEM OVERVIEW

### The Core Formula

```
FINAL RATING = RAW SKILL × EXPERIENCE MULTIPLIER
```

- **Raw Skill:** 1.0 - 10.0 scale (player's pure performance)
- **Experience Multiplier:** 0.0 - 1.15 scale (rewards experience)
- **Final Rating:** 1.0 - 10.0 scale (capped at 10.0 maximum)

### Why This System?

1. **Fair:** Every player starts from the same baseline
2. **Transparent:** All calculations are publicly documented
3. **Balanced:** Rewards both skill AND experience
4. **Role-Aware:** Different roles are evaluated differently
5. **Capped:** No infinite scaling, 10.0 is the maximum

---

## RAW SKILL CALCULATION (1.0 - 10.0)

Raw Skill measures pure performance through **6 weighted criteria**. Each criterion has a maximum contribution, and all are summed together.

### The 6 Criteria

| # | Criterion | Max Points | Formula | Cap |
|---|-----------|------------|---------|-----|
| 1 | 🏆 Win Rate | 3.5 | (Win Rate / 100) × 3.5 | None |
| 2 | 👑 MVP per Win | 2.5 | (MVP per Win / 100) × 2.5 | None |
| 3 | ⚽ Goals per Game | 1.5 | min(Goals/Game, 1.5) × 1.5 | 1.5 |
| 4 | 🧤 Saves per Game | 2.0 | min(Saves/Game, 3.0) × 2.0 | 3.0 |
| 5 | 🎯 Assists per Game | 1.5 | min(Assists/Game, 1.0) × 1.5 | 1.0 |
| 6 | 💫 MVP Efficiency | 0.5 | (MVP Efficiency / 10) × 0.5 | None |

### Penalty System

| Penalty | Condition | Deduction |
|---------|-----------|-----------|
| ❌ Own Goal Penalty | Own Goals/Game ≥ 0.55 | -0.5 points |

### Complete Formula

```python
raw_skill = (
    # Component 1: Win Rate (max 3.5)
    (win_rate / 100 * 3.5) +
    
    # Component 2: MVP per Win (max 2.5)
    (mvp_per_win / 100 * 2.5) +
    
    # Component 3: Goals per Game (max 1.5, capped at 1.5)
    (min(goals_per_game, 1.5) * 1.5) +
    
    # Component 4: Saves per Game (max 2.0, capped at 3.0)
    (min(saves_per_game, 3.0) * 2.0) +
    
    # Component 5: Assists per Game (max 1.5, capped at 1.0)
    (min(assists_per_game, 1.0) * 1.5) +
    
    # Component 6: MVP Efficiency (max 0.5)
    (mvp_efficiency / 10.0 * 0.5)
)

# Apply penalty for excessive own goals
if own_goals_per_game >= 0.55:
    raw_skill -= 0.5

# Cap raw skill between 1.0 and 10.0
final_skill = min(max(raw_skill, 1.0), 10.0)
```

### Understanding Each Component

#### 1. Win Rate (🏆)

- **Why:** Winning is the ultimate goal
- **How:** 100% win rate = 3.5 points, 0% = 0 points
- **Example:** 67% win rate = (67/100) × 3.5 = 2.345 points

#### 2. MVP per Win (👑)

- **Why:** MVPs indicate game-changing impact
- **How:** 100% MVP per win = 2.5 points, 0% = 0 points
- **Example:** 46% MVP per win = (46/100) × 2.5 = 1.15 points

#### 3. Goals per Game (⚽)

- **Why:** Goals win matches
- **How:** 1.5+ goals/game = 1.5 points (capped)
- **Example:** 0.59 goals/game = 0.59 × 1.5 = 0.885 points

#### 4. Saves per Game (🧤)

- **Why:** Defense wins championships
- **How:** 3.0+ saves/game = 2.0 points (capped)
- **Example:** 1.92 saves/game = 1.92 × 2.0 = 3.84 points

#### 5. Assists per Game (🎯)

- **Why:** Playmaking creates opportunities
- **How:** 1.0+ assists/game = 1.5 points (capped)
- **Example:** 0.33 assists/game = 0.33 × 1.5 = 0.495 points

#### 6. MVP Efficiency (💫)

- **Why:** Impact per action matters
- **How:** 10%+ efficiency = 0.5 points
- **Example:** 5.01% efficiency = (5.01/10) × 0.5 = 0.2505 points

---

## EXPERIENCE MULTIPLIER (0.0 - 1.15)

Experience multiplier rewards players who have played more matches. It ensures that experience matters while still allowing new players to compete.

### Tier System

| Tier | Matches | Multiplier Range | Formula |
|------|---------|------------------|---------|
| Veteran | 10,000+ | 1.00 - 1.15 | 1.00 + ((games-10000)/1500) × 0.015 (capped at 1.15) |
| Experienced | 5,000 - 9,999 | 0.85 - 0.99 | 0.85 + ((games-5000)/5000) × 0.14 |
| Established | 2,500 - 4,999 | 0.70 - 0.84 | 0.70 + ((games-2500)/2500) × 0.15 |
| Developing | 500 - 2,499 | 0.40 - 0.69 | 0.40 + ((games-500)/2000) × 0.30 |
| New | 0 - 499 | 0.00 - 0.39 | (games/500) × 0.40 |

### Complete Formula

```python
def calculate_experience_multiplier(total_games):
    # No matches = no multiplier
    if total_games == 0:
        return 0.0
    
    # Veteran Tier: 10,000+ matches
    # Each 1500 matches above 10k gives +0.015 bonus
    # Max cap is 1.15
    if total_games >= 10000:
        extra_games = total_games - 10000
        bonus = (extra_games / 1500) * 0.015
        return min(1.00 + bonus, 1.15)
    
    # Experienced Tier: 5,000-9,999 matches
    # Linear scaling from 0.85 at 5k to 0.99 at 10k
    elif total_games >= 5000:
        return 0.85 + ((total_games - 5000) / 5000) * 0.14
    
    # Established Tier: 2,500-4,999 matches
    # Linear scaling from 0.70 at 2.5k to 0.84 at 5k
    elif total_games >= 2500:
        return 0.70 + ((total_games - 2500) / 2500) * 0.15
    
    # Developing Tier: 500-2,499 matches
    # Linear scaling from 0.40 at 500 to 0.69 at 2.5k
    elif total_games >= 500:
        return 0.40 + ((total_games - 500) / 2000) * 0.30
    
    # New Tier: 0-499 matches
    # Linear scaling from 0.0 at 0 to 0.40 at 500
    else:
        return (total_games / 500) * 0.40
```

### Why This Multiplier?

- **Rewards Longevity:** Veterans get up to 15% bonus
- **Encourages New Players:** New players aren't punished too harshly
- **Gradual Progression:** Smooth scaling between tiers
- **Fair Cap:** No one can exceed 1.15x multiplier

---

## TACTICAL ROLE DETERMINATION

Players are classified into 3 main roles with sub-categories based on their statistics. This ensures that players are evaluated within their role context.

### Classification Flowchart

```
                    START
                      │
                      ▼
         ┌─────────────────────┐
         │ Goalkeeping Index   │
         │ > 55%?              │
         └─────────────────────┘
              │            │
             YES           NO
              │            │
              ▼            ▼
     ┌─────────────┐  ┌─────────────┐
     │ GOALKEEPER  │  │ Goal/Assist │
     │  Category   │  │ Ratio > 1.5 │
     └─────────────┘  │ AND Goals/G │
              │        │ >= 1.2?    │
              │        └─────────────┘
              │              │            │
              │             YES           NO
              │              │            │
              │              ▼            ▼
              │     ┌─────────────┐  ┌─────────────┐
              │     │  STRIKER    │  │ PLAYMAKER   │
              │     │  Category   │  │  Category   │
              │     └─────────────┘  └─────────────┘
              │
              ▼
         ┌─────────────┐
         │ Apply       │
         │ Veteran     │
         │ Label if    │
         │ 10K+ Matches│
         └─────────────┘
```

### 1. 🧤 GOALKEEPER (Saves > 55% of actions)

**Sub-Categories:**

| Category | Requirements |
|----------|--------------|
| Elite Goalkeeper | Defense Reliability ≥ 8.0 AND Win Rate ≥ 45% |
| Liability Goalkeeper | Own Goals/Game ≥ 0.55 |
| Passive Goalkeeper | All other goalkeepers |

**Logic:**

```python
if goalkeeping_index > 55:
    if defense_reliability >= 8.0 and win_rate >= 45:
        role = "Elite Goalkeeper"
    elif own_goals_per_game >= 0.55:
        role = "Liability Goalkeeper"
    else:
        role = "Passive Goalkeeper"
```

### 2. ⚽ STRIKER (Goal/Assist > 1.5 AND Goals/Game ≥ 1.2)

**Sub-Categories:**

| Category | Requirements |
|----------|--------------|
| Elite Striker | Win Rate ≥ 48% AND MVP per Win ≥ 33.33% |
| Ball-Chasing Striker | Win Rate < 43% OR MVP per Win < 28% OR Own Goals/Game ≥ 0.55 |
| Opportunist Striker | All other strikers |

**Logic:**

```python
elif goal_assist_ratio > 1.5 and goals_per_game >= 1.2:
    if win_rate >= 48 and mvp_per_win >= 33.33:
        role = "Elite Striker"
    elif win_rate < 43 or mvp_per_win < 28 or own_goals_per_game >= 0.55:
        role = "Ball-Chasing Striker"
    else:
        role = "Opportunist Striker"
```

### 3. 🎯 PLAYMAKER (All other players)

**Sub-Categories:**

| Category | Requirements |
|----------|--------------|
| Elite Playmaker | Assists/Game ≥ 0.75 AND Win Rate ≥ 48% |
| Efficient MVP | MVP Efficiency ≥ 10.0% AND Win Rate ≥ 45% |
| Standard Playmaker | Win Rate ≥ 48% AND MVP per Win ≥ 35% |
| Inconsistent Midfielder | Win Rate < 43% OR Own Goals/Game ≥ 0.55 |
| All-Round Midfielder | All other playmakers |

**Logic:**

```python
else:
    if assists_per_game >= 0.75 and win_rate >= 48:
        role = "Elite Playmaker"
    elif mvp_efficiency >= 10.0 and win_rate >= 45:
        role = "Efficient MVP"
    elif win_rate >= 48 and mvp_per_win >= 35:
        role = "Standard Playmaker"
    elif win_rate < 43 or own_goals_per_game >= 0.55:
        role = "Inconsistent Midfielder"
    else:
        role = "All-Round Midfielder"
```

### Veteran Label

Players with 10,000+ matches get the "☣️ 10K+ Match" label added to their role:

```python
if total_games >= 10000:
    return f"☣️ 10K+ Match | {role}"
```

---

## TIER SYSTEM

Final rating is converted to a tier for easy visual identification:

| Rating Range | Tier | Emoji | Meaning |
|--------------|------|-------|---------|
| 9.0 - 10.0 | S-Tier | ☣ | Best of the best |
| 7.5 - 8.9 | A-Tier | 🥇 | Elite players |
| 6.0 - 7.4 | B-Tier | 🥈 | Very good players |
| 4.5 - 5.9 | C-Tier | 🥉 | Above average |
| 3.0 - 4.4 | D-Tier | 📈 | Average players |
| 0.0 - 2.9 | E-Tier | 🆕 | New or developing |

### Complete Function

```python
def calculate_tier_from_rating(rating):
    if rating >= 9.0:
        return "☣ S-Tier"
    elif rating >= 7.5:
        return "🥇 A-Tier"
    elif rating >= 6.0:
        return "🥈 B-Tier"
    elif rating >= 4.5:
        return "🥉 C-Tier"
    elif rating >= 3.0:
        return "📈 D-Tier"
    else:
        return "🆕 E-Tier"
```

---

## COMPLETE EXAMPLE: "keLLog"

Let's walk through the complete calculation for a real player.

### Step 1: Raw Data Input

| Stat | Value |
|------|-------|
| Won | 4,401 |
| Lost | 5,854 |
| MVP | 1,481 |
| Goals | 6,528 |
| Assists | 3,398 |
| Saves | 19,651 |
| Own Goals | 1,623 |

### Step 2: Calculate Per-Game Metrics

```python
total_games = 4,401 + 5,854 = 10,255

win_rate = (4,401 / 10,255) × 100 = 42.91%
goals_per_game = 6,528 / 10,255 = 0.6365
assists_per_game = 3,398 / 10,255 = 0.3313
saves_per_game = 19,651 / 10,255 = 1.9162
own_goals_per_game = 1,623 / 10,255 = 0.1583

mvp_per_win = (1,481 / 4,401) × 100 = 33.65%
```

### Step 3: Calculate Advanced Metrics

```python
total_actions = 6,528 + 3,398 + 19,651 = 29,577

goalkeeping_index = (19,651 / 29,577) × 100 = 66.44%
defense_reliability = 19,651 / 1,623 = 12.11
goal_to_assist_ratio = 6,528 / 3,398 = 1.92

mvp_efficiency = (1,481 / 29,577) × 100 = 5.01%
```

### Step 4: Determine Tactical Role

```python
# Goalkeeper Check
goalkeeping_index = 66.44% > 55% → GOALKEEPER!

# Sub-category check
defense_reliability = 12.11 ≥ 8.0 ✓
win_rate = 42.91% ≥ 45% → FALSE

# Not Elite Goalkeeper
own_goals_per_game = 0.1583 ≥ 0.55 → FALSE

# → Passive Goalkeeper

# Veteran Check
total_games = 10,255 ≥ 10,000 → ✓

# FINAL ROLE:
"☣️ 10K+ Match | Passive Goalkeeper"
```

### Step 5: Calculate Raw Skill

```python
# Component 1: Win Rate
(42.91 / 100) × 3.5 = 1.5019

# Component 2: MVP per Win
(33.65 / 100) × 2.5 = 0.8413

# Component 3: Goals per Game (capped at 1.5)
min(0.6365, 1.5) × 1.5 = 0.9548

# Component 4: Saves per Game (capped at 3.0)
min(1.9162, 3.0) × 2.0 = 3.8324

# Component 5: Assists per Game (capped at 1.0)
min(0.3313, 1.0) × 1.5 = 0.4970

# Component 6: MVP Efficiency
(5.01 / 10) × 0.5 = 0.2505

# Sum all components
raw_skill = 1.5019 + 0.8413 + 0.9548 + 3.8324 + 0.4970 + 0.2505
raw_skill = 7.8779

# Own Goal Penalty Check
own_goals_per_game = 0.1583 < 0.55 → No penalty

# Cap between 1.0 and 10.0
final_skill = min(max(7.8779, 1.0), 10.0) = 7.8779
```

### Step 6: Calculate Experience Multiplier

```python
total_games = 10,255 ≥ 10,000 → Veteran Tier

extra_games = 10,255 - 10,000 = 255
bonus = (255 / 1500) × 0.015 = 0.0026
multiplier = min(1.00 + 0.0026, 1.15) = 1.0026

# Round to 3 decimals
multiplier = 1.003
```

### Step 7: Calculate Final Rating

```python
final_rating = final_skill × multiplier
final_rating = 7.8779 × 1.0026 = 7.8984

# Cap at 10.0
final_rating = min(7.8984, 10.0) = 7.9

# Round to 1 decimal
final_rating = 7.9
```

### Step 8: Calculate Tier

```python
rating = 7.9
7.5 ≤ 7.9 < 9.0 → "🥇 A-Tier"
```

### Final Result

```
@keLLog | ☣️ 10K+ Match | 🧤 Passive Goalkeeper | 🥇 A-Tier
Rating: 7.9/10 (Raw: 7.9)
Win: %42.9 4401W/5854L (0.75)
MVP: 1481 (34%)
Stats: Saves: 19651 - Goals: 6528 - Assists: 3398 - Own Goals: 1623
```

---

## BREAKDOWN OF RESULTS

### Why 7.9 Rating?

| Component | Score | Max | Percentage |
|-----------|-------|-----|------------|
| Win Rate | 1.50 | 3.5 | 43% |
| MVP per Win | 0.84 | 2.5 | 34% |
| Goals/Game | 0.95 | 1.5 | 64% |
| Saves/Game | 3.83 | 2.0 | 100%+ |
| Assists/Game | 0.50 | 1.5 | 33% |
| MVP Efficiency | 0.25 | 0.5 | 50% |
| **TOTAL** | **7.88** | **10.0** | **79%** |

**Key Insights:**

- Saves/Game is the strongest component - 3.83 points out of 2.0 max (capped)
- Win Rate is the weakest - Only 43% of max
- MVP per Win is moderate - 34% of max
- No Own Goal penalty - 0.158 < 0.55 threshold

### Why Passive Goalkeeper?

- Goalkeeping Index: 66.44% > 55% → Definitely a goalkeeper
- Defense Reliability: 12.11 ≥ 8.0 ✓ (Elite requirement)
- Win Rate: 42.91% < 45% ✗ (Fails Elite requirement)
- Own Goals: 0.158 < 0.55 ✓ (Not Liability)
- **Result:** Passive Goalkeeper

### Why 1.003 Multiplier?

- 10,255 matches is just above the 10,000 threshold
- 255 games above threshold = minimal bonus
- Multiplier: 1.003 (only 0.3% bonus)
- More matches would increase multiplier

---

## COMPLETE PROCESS FLOW

```
1. INPUT: Raw Statistics
   ↓
2. VALIDATE: Check for zero games
   ↓
3. CALCULATE: Per-game metrics
   - Win Rate
   - Goals/Game
   - Assists/Game
   - Saves/Game
   - Own Goals/Game
   ↓
4. CALCULATE: Advanced metrics
   - Total Actions
   - Goalkeeping Index
   - Defense Reliability
   - Goal/Assist Ratio
   - MVP Efficiency
   ↓
5. DETERMINE: Tactical Role
   - Goalkeeper / Striker / Playmaker
   - Sub-category
   - Veteran label (if 10K+ matches)
   ↓
6. CALCULATE: Raw Skill (6 components)
   ↓
7. APPLY: Own Goal Penalty (if applicable)
   ↓
8. CAP: Raw Skill between 1.0 - 10.0
   ↓
9. CALCULATE: Experience Multiplier
   ↓
10. CALCULATE: Final Rating = Skill × Multiplier
   ↓
11. CAP: Final Rating ≤ 10.0
   ↓
12. DETERMINE: Tier (S to E)
   ↓
13. OUTPUT: Complete player profile
```

---

## WHY THIS SYSTEM WORKS

### 1. Balanced Criteria

- Attack (Goals, Assists)
- Defense (Saves)
- Team Impact (Win Rate, MVP)
- No single stat dominates

### 2. Role Recognition

- Goalkeepers aren't compared to Strikers
- Each role has its own evaluation path
- Sub-categories add nuance

### 3. Experience Matters

- Veterans get recognition (10K+ label)
- Multiplier rewards longevity
- New players aren't punished too harshly

### 4. Fair Caps

- No stat can exceed reasonable limits
- Raw Skill capped at 10.0
- Final Rating capped at 10.0

### 5. Penalty System

- Excessive own goals are penalized
- Prevents negative playstyles
- Encourages team play

### 6. Transparency

- All formulas are public
- Anyone can verify calculations
- No hidden adjustments

---

## GLOSSARY

| Term | Definition |
|------|------------|
| Raw Skill | Pure performance score (1.0-10.0) |
| Experience Multiplier | Bonus based on matches played (0.0-1.15) |
| Final Rating | Raw Skill × Multiplier (1.0-10.0) |
| MVP Efficiency | MVPs per total action × 100 |
| Goalkeeping Index | Percentage of actions that are saves |
| Defense Reliability | Saves per own goal |
| Goal/Assist Ratio | Goals divided by assists |
| Veteran | Player with 10,000+ matches |
| Tier | Letter grade (S to E) based on rating |

---

## DESIGN PHILOSOPHY

### Fairness

Every player starts from the same baseline. The system doesn't favor any specific playstyle.

### Transparency

All calculations are documented and verifiable. No hidden formulas or adjustments.

### Balance

Attack, defense, and team impact are all weighted appropriately. No single stat dominates.

### Recognition

Good performance is rewarded. Experience is valued. Role specialization is recognized.

### Simplicity

The core formula is simple enough to understand, yet comprehensive enough to be accurate.

---

## IMPLEMENTATION NOTES

### Data Requirements

The system requires these raw statistics per player:

- Won / Lost (Matches)
- MVP (Total awards)
- Goals / Assists / Saves (Total counts)
- Own Goals (Total count)

### Calculation Order

1. Extract raw data
2. Calculate per-game metrics
3. Calculate advanced metrics
4. Determine role
5. Calculate raw skill
6. Apply penalties
7. Apply multiplier
8. Calculate final rating
9. Determine tier

### Validation

- Zero games = default values
- Missing data = defaults to 0
- All calculations are protected against division by zero

---

## SYSTEM LIMITATIONS

- **Static Weights:** Criteria weights are fixed and don't adapt
- **No Player Interaction:** Doesn't account for teammates/opponents
- **Pure Statistics:** Doesn't capture tactical decisions
- **Match Context:** Doesn't consider match importance
- **Time Decay:** Older matches count equally to recent ones

### Future Improvements

- Dynamic weights based on meta
- Performance decay over time
- Team chemistry factors
- Opponent strength adjustment

---

## SYSTEM ADVANTAGES

- **Comprehensive:** Covers attack, defense, and impact
- **Role-Aware:** Different roles evaluated differently
- **Experience-Based:** Rewards veterans appropriately
- **Fair:** Equal opportunity for all players
- **Transparent:** All calculations are public
- **Capped:** No infinite scaling
- **Balanced:** No single stat dominates

---

This document provides the complete, detailed explanation of the CarBall rating system. All calculations are open for verification and improvement.

**Last Updated:** August 2026
