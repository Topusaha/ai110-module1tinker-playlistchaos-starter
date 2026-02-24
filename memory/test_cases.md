# Test Cases for Playlist Classification

Use these test cases to verify song classification behavior. Add each song through the app's "Add a song" sidebar.

## Test Setup

**Default Profile Settings:**
- Hype min energy: 7
- Chill max energy: 3
- Favorite genre: rock

---

## Group 1: Energy-Based Classification (No Keyword Conflicts)

These test basic energy thresholds without genre/keyword interference.

| # | Title | Artist | Genre | Energy | Tags | Expected Result | Why |
|---|-------|--------|-------|--------|------|-----------------|-----|
| 1 | "Test High Energy" | "Test Artist" | electronic | 10 | none | **Hype** | Energy 10 >= 7 |
| 2 | "Test Low Energy" | "Test Artist" | electronic | 1 | none | **Chill** | Energy 1 <= 3 |
| 3 | "Test Mid Energy" | "Test Artist" | electronic | 5 | none | **Mixed** | Energy 5 is between 3 and 7 |
| 4 | "Boundary High" | "Test Artist" | electronic | 7 | none | **Hype** | Energy exactly at hype threshold |
| 5 | "Boundary Low" | "Test Artist" | electronic | 3 | none | **Chill** | Energy exactly at chill threshold |

---

## Group 2: Favorite Genre Conflict

Tests what happens when a song's genre matches favorite_genre but has low energy.

| # | Title | Artist | Genre | Energy | Tags | Expected Result | Actual (Buggy) Result |
|---|-------|--------|-------|--------|------|-----------------|----------------------|
| 6 | "Low Energy Rock" | "Test Artist" | rock | 1 | none | **Chill** (energy 1 <= 3) | **Hype** (genre = favorite) |
| 7 | "Mid Energy Rock" | "Test Artist" | rock | 5 | none | **Hype** (genre = favorite) | **Hype** ✓ |
| 8 | "Quiet Rock Ballad" | "Test Artist" | rock | 2 | ballad | **Chill** (energy 2 <= 3) | **Hype** (genre = favorite) |

**Bug Revealed:** Songs with favorite_genre ALWAYS go to Hype, even with chill-level energy.

---

## Group 3: Hype Keyword Conflict

Tests genre keywords (rock, punk, party) that trigger hype classification.

| # | Title | Artist | Genre | Energy | Tags | Expected Result | Actual (Buggy) Result |
|---|-------|--------|-------|--------|------|-----------------|----------------------|
| 9 | "Quiet Punk Acoustic" | "Test Artist" | punk | 2 | acoustic | **Chill** (energy 2 <= 3) | **Hype** (punk keyword) |
| 10 | "Soft Party Music" | "Test Artist" | party | 1 | soft | **Chill** (energy 1 <= 3) | **Hype** (party keyword) |
| 11 | "Hard Rock Energy" | "Test Artist" | rock | 9 | loud | **Hype** (energy 9 >= 7) | **Hype** ✓ |

**Bug Revealed:** Genre keywords override energy-based classification.

---

## Group 4: Chill Keyword Behavior

Tests if chill keywords (lofi, ambient, sleep) work correctly. Note: These check the TITLE, not genre.

| # | Title | Artist | Genre | Energy | Tags | Expected Result | Notes |
|---|-------|--------|-------|--------|------|-----------------|-------|
| 12 | "lofi beats" | "Test Artist" | other | 5 | none | **Chill** (title has "lofi") | Title keyword should trigger |
| 13 | "High Energy" | "Test Artist" | lofi | 5 | none | **Mixed** (energy 5 is mid) | Genre "lofi" won't help (checks title!) |
| 14 | "ambient soundscape" | "Test Artist" | electronic | 8 | none | **Chill** (title has "ambient") or **Hype**? | Conflict: title vs energy |
| 15 | "sleep music" | "Test Artist" | other | 1 | relax | **Chill** (both title and energy) | Should definitely be Chill |

**Bug Revealed:** Chill keywords check TITLE (line 74), while hype keywords check GENRE (line 73). This asymmetry is confusing.

---

## Group 5: Threshold Boundary Tests

Tests songs exactly at the thresholds.

| # | Title | Artist | Genre | Energy | Tags | Expected Result | Why |
|---|-------|--------|-------|--------|------|-----------------|-----|
| 16 | "Boundary Test A" | "Test Artist" | jazz | 3 | none | **Chill** | energy <= 3 |
| 17 | "Boundary Test B" | "Test Artist" | jazz | 4 | none | **Mixed** | 3 < energy < 7 |
| 18 | "Boundary Test C" | "Test Artist" | jazz | 7 | none | **Hype** | energy >= 7 |

---

## Group 6: Change Profile Settings Test

**Instructions:** After adding these songs, change the profile settings and observe reclassification.

Add these with default settings:
| # | Title | Artist | Genre | Energy | Tags | Expected (Default) |
|---|-------|--------|-------|--------|------|--------------------|
| 19 | "Profile Test 1" | "Test Artist" | pop | 5 | none | **Mixed** |
| 20 | "Profile Test 2" | "Test Artist" | pop | 6 | none | **Mixed** |

**Then change settings to:**
- Hype min energy: 5
- Chill max energy: 6
- Favorite genre: pop

**Refresh or re-render playlists (toggle a setting) and observe:**
- Song #19 should move to **Hype** (genre = favorite)
- Song #20 should move to **Hype** (genre = favorite)

---

## Expected Bug Summary

Based on the current code logic, you should see:

1. **Any song with genre="rock" goes to Hype** (due to favorite_genre default)
2. **Any song with genre containing "rock", "punk", or "party" goes to Hype** (regardless of energy)
3. **Chill keywords only work if they're in the TITLE, not genre**
4. **The order matters:** Hype criteria are checked first, so songs that qualify for both Hype and Chill will always go to Hype

---

## Quick Test Checklist

Add these three songs to quickly see the main bug:

1. **Title:** "Quiet Rock Song" | **Artist:** "Test" | **Genre:** rock | **Energy:** 1
   - Should be: Chill (energy 1 <= 3)
   - Actually is: Hype (genre = favorite_genre)

2. **Title:** "Peaceful Music" | **Artist:** "Test" | **Genre:** ambient | **Energy:** 2
   - Should be: Chill (energy 2 <= 3)
   - Actually is: Chill ✓ (works because genre isn't a hype keyword)

3. **Title:** "lofi study beats" | **Artist:** "Test" | **Genre:** electronic | **Energy:** 8
   - Should be: Hype (energy 8 >= 7) or Chill (title has "lofi")?
   - Intended per spec: Chill (title keyword)
   - Actually is: Hype (energy checked first)
