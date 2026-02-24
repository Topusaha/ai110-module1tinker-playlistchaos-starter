Song Classification (The "Mood" Engine):
Hype: A song should be classified as "Hype" if its energy is greater than or equal to the hype_min_energy (default 7), if its genre matches the user's favorite_genre, or if the genre contains "hype" keywords (rock, punk, party).
Chill: A song should be classified as "Chill" if its energy is less than or equal to the chill_max_energy (default 3) or if the title contains "chill" keywords (lofi, ambient, sleep).
Mixed: Any song that does not strictly meet the "Hype" or "Chill" criteria should fall into the "Mixed" playlist.
Search Functionality:
The search bar should be case-insensitive and perform a partial match against the selected field (e.g., searching "AC" should find "AC/DC").
It should return all songs where the query string is contained within the song's attribute.
Playlist Statistics:
Total Songs: Should represent the unique count of all songs across all categories.
Average Energy: Should be the mathematical average of the energy levels of all songs in the system.
Hype Ratio: Should represent the percentage of "Hype" songs relative to the total number of songs.
Lucky Pick:
When a user selects "Hype" or "Chill," the app must only pick a song from that specific playlist.
Selecting "Any" should pull a random song from the combined pool of Hype and Chill (and ideally Mixed).
Data Normalization:
All user input (titles, artists, genres) should be trimmed of leading/trailing whitespace and handled consistently (e.g., artists and genres converted to lowercase for comparison) to prevent duplicate or mismatched entries.