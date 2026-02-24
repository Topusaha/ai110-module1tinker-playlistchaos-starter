# Test Results for Playlist Chaos

Tests run by tracing through `playlist_logic.py` with default profile (hype_min_energy: 7, chill_max_energy: 3, favorite_genre: rock).

---

## Group 1: Energy-Based Classification

| # | Test Name | Expected | Actual | Pass? |
|---|-----------|----------|--------|-------|
| 1 | Test High Energy (energy 10, electronic) | Hype | Hype | ✅ PASS |
| 2 | Test Low Energy (energy 1, electronic) | Chill | Chill | ✅ PASS |
| 3 | Test Mid Energy (energy 5, electronic) | Mixed | Mixed | ✅ PASS |
| 4 | Boundary High (energy 7, electronic) | Hype | Hype | ✅ PASS |
| 5 | Boundary Low (energy 3, electronic) | Chill | Chill | ✅ PASS |

---

## Group 2: Favorite Genre Conflict

| # | Test Name | Expected | Actual | Pass? |
|---|-----------|----------|--------|-------|
| 6 | Low Energy Rock (energy 1, rock) | Chill | Hype | ❌ FAIL |
| 7 | Mid Energy Rock (energy 5, rock) | Hype | Hype | ✅ PASS |
| 8 | Quiet Rock Ballad (energy 2, rock) | Chill | Hype | ❌ FAIL |

**Bug:** `genre == favorite_genre` is checked before energy, so any rock song (favorite genre) always goes to Hype regardless of its energy level.

---

## Group 3: Hype Keyword Conflict

| # | Test Name | Expected | Actual | Pass? |
|---|-----------|----------|--------|-------|
| 9 | Quiet Punk Acoustic (energy 2, punk) | Chill | Hype | ❌ FAIL |
| 10 | Soft Party Music (energy 1, party) | Chill | Hype | ❌ FAIL |
| 11 | Hard Rock Energy (energy 9, rock) | Hype | Hype | ✅ PASS |

**Bug:** `is_hype_keyword` (checks if genre contains "rock", "punk", or "party") is evaluated before energy thresholds, so low-energy punk/party songs get forced into Hype.

---

## Group 4: Chill Keyword Behavior

| # | Test Name | Expected | Actual | Pass? |
|---|-----------|----------|--------|-------|
| 12 | "lofi beats" (energy 5, genre: other) | Chill | Chill | ✅ PASS |
| 13 | "High Energy" (energy 5, genre: lofi) | Mixed | Mixed | ✅ PASS |
| 14 | "ambient soundscape" (energy 8, electronic) | Chill (title keyword) or Hype? | Hype | ❌ FAIL (Chill keyword ignored — energy wins) |
| 15 | "sleep music" (energy 1, other) | Chill | Chill | ✅ PASS |

**Note on #14:** The code checks Hype conditions first. Energy 8 >= 7 triggers Hype before the chill title keyword ("ambient") is ever evaluated.

**Note on #13:** The genre "lofi" does NOT trigger chill — chill keywords only check the song title, not the genre. This asymmetry is a design bug (hype keywords check genre, chill keywords check title).

---

## Group 5: Threshold Boundary Tests

| # | Test Name | Expected | Actual | Pass? |
|---|-----------|----------|--------|-------|
| 16 | Boundary Test A (energy 3, jazz) | Chill | Chill | ✅ PASS |
| 17 | Boundary Test B (energy 4, jazz) | Mixed | Mixed | ✅ PASS |
| 18 | Boundary Test C (energy 7, jazz) | Hype | Hype | ✅ PASS |

---

## Group 6: Profile Change Test

| # | Test Name | Expected (default) | Actual (default) | Pass? |
|---|-----------|---------------------|------------------|-------|
| 19 | Profile Test 1 (energy 5, pop) | Mixed | Mixed | ✅ PASS |
| 20 | Profile Test 2 (energy 6, pop) | Mixed | Mixed | ✅ PASS |

After changing profile to hype_min_energy: 5, chill_max_energy: 6, favorite_genre: pop:

| # | Test Name | Expected (new profile) | Actual (new profile) | Pass? |
|---|-----------|------------------------|----------------------|-------|
| 19 | Profile Test 1 (energy 5, pop) | Hype | Hype | ✅ PASS |
| 20 | Profile Test 2 (energy 6, pop) | Hype | Hype | ✅ PASS |

---

## Summary

| Result | Count |
|--------|-------|
| ✅ PASS | 15 |
| ❌ FAIL | 5 |

**Tests that failed:** #6, #8 (favorite genre overrides low energy), #9, #10 (hype genre keywords override low energy), #14 (hype energy check overrides chill title keyword).

---

## Additional Bugs Found (Beyond Test Cases)

**Lucky Pick "any" mode** — Only picks from Hype + Chill. Mixed songs are excluded, even though the intended behavior says "any" should include all songs.

**Empty playlist crash** — `random_choice_or_none()` calls `random.choice()` on an empty list, which raises an `IndexError` instead of returning `None` as the function name implies. If Hype or Chill is empty and lucky pick is used in that mode, the app will crash.

**Title normalization** — `normalize_title()` only strips whitespace but does NOT lowercase. Chill keyword matching on titles is therefore case-sensitive: a song titled "Lofi Beats" will NOT be classified as Chill, but "lofi beats" will.

**Search is fixed** — The search containment bug (`value in query` vs `query in value`) appears to have already been corrected in the current code. Search works as expected.
