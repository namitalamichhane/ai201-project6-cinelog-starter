\# PR Response Doc — CineLog Watchlist Feature



\## AI Usage

<!-- fill in at the end -->



\## Comment 1 — Rename

\*\*What I did:\*\* Renamed `save\_to\_watchlist()` to `add\_to\_watchlist()` in `services/watchlist\_service.py` to match the naming convention used by `add\_to\_collection()` in the collection service. Updated the import and call site in `routes/watchlist/watchlist.py` (inside `add\_film()`).

\*\*How I verified:\*\* Ran a search across the whole repo for `save\_to\_watchlist` and confirmed zero remaining references. Ran `pytest tests/ -v` to confirm all existing tests still pass after the rename.



\## Comment 2 — Deduplication

\*\*What I did:\*\* Added an `AlreadyInWatchlistError` exception and a duplicate check in `add\_to\_watchlist()`, querying `WatchlistEntry` by `user\_id` and `film\_id` before creating a new entry — mirroring the pattern used in `add\_to\_collection()` against `CollectionEntry`.

\*\*How I verified:\*\* Ran `pytest tests/ -v` to confirm existing tests still pass.



## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` following the fixture and structure of `tests/test_collection.py`. Added `test_add_to_watchlist_nonexistent_film_raises`, modeled directly on `test_add_to_collection_nonexistent_film_raises`, to confirm `FilmNotFoundError` is raised for a fake film_id instead of a database error. Also added `test_add_to_watchlist_creates_entry` and `test_add_to_watchlist_duplicate_raises` to cover the basic add and the Comment 2 dedup behavior.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — all three tests pass. Ran the full suite with `pytest tests/ -v` to confirm nothing else broke.


\## Comment 4 — Default visibility

\*\*My position:\*\*

\*\*Reasoning:\*\*

\*\*Tradeoff acknowledged:\*\*



\## Comment 5 — Sort order

\*\*My position:\*\*

\*\*Reasoning:\*\*

\*\*Engagement with reviewer's point:\*\*



\## Comment 6 — Rebase

\*\*What conflicted:\*\*

\*\*How I resolved it:\*\*

\*\*How I verified no conflict remains:\*\*



\## PR Description

