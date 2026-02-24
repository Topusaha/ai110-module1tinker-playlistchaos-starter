# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Playlist Chaos** is a Streamlit-based smart playlist generator for an AI110 course module. The app is intentionally imperfect and contains bugs for students to discover and fix as part of a debugging exercise.

## Running the Application

```bash
# Install dependencies
pip install -r requirements.txt

# Run the Streamlit app
streamlit run app.py
```

The app will open in your browser at `http://localhost:8501`.

## Code Architecture

### Two-Layer Architecture

The codebase is split into UI and logic layers:

**`app.py`** - Streamlit UI layer
- Manages session state (songs, profile, history)
- Renders UI components: playlists, search, lucky pick, stats, history
- Handles user interactions: adding songs, changing profile, clicking buttons
- Calls functions from `playlist_logic.py` for all business logic

**`playlist_logic.py`** - Business logic layer
- Song normalization and classification
- Playlist building and merging
- Search functionality
- Statistics computation
- Random song selection (lucky pick)
- History tracking

### Key Data Flow

1. **Songs → Normalization → Classification → Playlists**
   - Raw song dicts are normalized (titles stripped, artists/genres lowercased)
   - Each song is classified as "Hype", "Chill", or "Mixed" based on energy and profile
   - Songs are grouped into playlist dictionaries by mood

2. **User Profile Drives Classification**
   - `hype_min_energy`: Threshold for high-energy songs (default: 7)
   - `chill_max_energy`: Threshold for low-energy songs (default: 3)
   - `favorite_genre`: Genre that automatically qualifies as "Hype"
   - Songs between thresholds fall into "Mixed"

3. **Search Uses Field Matching**
   - Currently hardcoded to search by "artist" field only
   - Performs lowercase, trimmed substring matching
   - Empty query returns all songs

## Intended Behavior (from memory/intended_behavior.md)

### Song Classification
- **Hype**: energy >= hype_min_energy OR genre == favorite_genre OR genre contains ["rock", "punk", "party"]
- **Chill**: energy <= chill_max_energy OR title contains ["lofi", "ambient", "sleep"]
- **Mixed**: All other songs

### Search
- Case-insensitive partial match
- Query substring should be found within the song's field value
- Example: "AC" should match "AC/DC"

### Statistics
- **Total Songs**: Count of all unique songs across categories
- **Average Energy**: Mean energy of ALL songs (not just one category)
- **Hype Ratio**: Percentage of Hype songs / total songs

### Lucky Pick
- "hype" mode: Pick from Hype playlist only
- "chill" mode: Pick from Chill playlist only
- "any" mode: Pick from combined Hype + Chill (should include Mixed)

### Normalization
- Titles: Stripped of whitespace
- Artists: Lowercased and stripped
- Genres: Lowercased and stripped

## Known Bugs (Part of Learning Exercise)

The code intentionally contains bugs for students to discover. Common issues include:

1. **Search logic** - Containment check may be backwards (checking `value in query` vs `query in value`)
2. **Statistics** - May calculate averages or ratios using wrong subsets
3. **Classification** - Logic may misclassify songs at threshold boundaries
4. **Lucky pick** - "any" mode may not include all expected playlists
5. **Normalization** - May not be consistently applied

When fixing bugs, students should add comments explaining the fix.

## Session State Management

Streamlit session state stores:
- `st.session_state.songs` - List of all song dictionaries
- `st.session_state.profile` - User profile with thresholds and preferences
- `st.session_state.history` - List of songs picked via "Lucky Pick"

State is initialized once on first load and persists across reruns.

## Testing Changes

After making fixes:
1. Run the app with `streamlit run app.py`
2. Add songs with various energy levels (1-10)
3. Change profile settings and observe playlist changes
4. Test search with partial matches and different cases
5. Use lucky pick multiple times and check history
6. Verify statistics update correctly
