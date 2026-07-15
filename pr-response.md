# PR Response Doc — CineLog Watchlist Feature



# AI Usage

<!-- fill in at the end -->



# Comment 1 — Rename

**What I did:** Renamed `save\_to\_watchlist()` to `add\_to\_watchlist()` in `services/watchlist\_service.py` to match the naming convention used by `add\_to\_collection()` in the collection service. Updated the import and call site in `routes/watchlist/watchlist.py` (inside `add\_film()`).

**How I verified:** Ran a search across the whole repo for `save\_to\_watchlist` and confirmed zero remaining references. Ran `pytest tests/ -v` to confirm all existing tests still pass after the rename.



# Comment 2 — Deduplication

**What I did:** Added an `AlreadyInWatchlistError` exception and a duplicate check in `add\_to\_watchlist()`, querying `WatchlistEntry` by `user\_id` and `film\_id` before creating a new entry — mirroring the pattern used in `add\_to\_collection()` against `CollectionEntry`.

**How I verified:** Ran `pytest tests/ -v` to confirm existing tests still pass.



## Comment 3 — Missing test
**What I did:** Created `tests/test_watchlist.py` following the fixture and structure of `tests/test_collection.py`. Added `test_add_to_watchlist_nonexistent_film_raises`, modeled directly on `test_add_to_collection_nonexistent_film_raises`, to confirm `FilmNotFoundError` is raised for a fake film_id instead of a database error. Also added `test_add_to_watchlist_creates_entry` and `test_add_to_watchlist_duplicate_raises` to cover the basic add and the Comment 2 dedup behavior.
**How I verified:** Ran `pytest tests/test_watchlist.py -v` — all three tests pass. Ran the full suite with `pytest tests/ -v` to confirm nothing else broke.


## Comment 4 — Default visibility
**My position:** I'm changing the default to `public=False` in `WatchlistEntry`.
**Reasoning:** A watchlist is fundamentally different from a collection. A collection ("films I've watched") is a completed, curated statement a user is choosing to share. A watchlist ("films I want to watch") is more exploratory — it can reveal what someone is currently curious about (a specific genre, director, or sensitive topic) before they've decided anything publicly. Defaulting to private treats the watchlist as closer to a personal to-do list than a public statement, and users can opt in to visibility once they're comfortable, rather than having to opt out after the fact.
**Tradeoff acknowledged:** Defaulting to private reduces the data available for CineLog's social/discovery features (friend activity, "others are watching") right out of the gate, since most users never change default settings. This works against the "community" framing of the app. If the product later leans harder into social discovery, this default may need revisiting.



## Comment 5 — Sort order
**My position:** I'm keeping `get_watchlist()` sorted alphabetically by `Film.title`, rather than switching to `date_added` descending.
**Reasoning:** A watchlist and a collection serve different mental models even though they're structurally similar. The collection feature is a log of what a user has already done — chronological order makes sense there because it reads like an activity history ("what did I watch recently"). A watchlist, by contrast, is a lookup list a user browses in the moment they're deciding what to watch next ("what's on my list?"). As the list grows, alphabetical order makes a specific title fast to find, while chronological order becomes noise once the user no longer remembers what order they added things in.
**Engagement with reviewer's point:** I understand the argument for consistency — using the same sort convention as `get_collection()` makes the codebase more predictable for future contributors, and that's a real cost of my choice. But I think matching an existing pattern for its own sake isn't a strong enough reason when the two features have different use cases. If a user's watchlist is small (a handful of titles), the ordering barely matters either way — but once it's larger, alphabetical is more usable for a browsing task than a chronological one. I'd be open to revisiting this if user data later shows people mostly interact with recently-added titles rather than searching their full list.



## Comment 6 — Rebase
**What conflicted:** `models.py` — `WatchlistEntry.film_id` was still defined as `db.Integer`, referencing the pre-refactor Film ID type, while `main` had migrated `Film.id` and related foreign keys to `db.String(36)` (UUID) in the "refactor: migrate film IDs from integer to UUID" commit.
**How I resolved it:** Updated `WatchlistEntry.film_id` to `db.Column(db.String(36), db.ForeignKey("film.id"), nullable=False)` to match the UUID type used elsewhere in the schema. Also updated the docstring in `add_to_watchlist()` and the fake film_id in `test_add_to_watchlist_nonexistent_film_raises` from an integer to a UUID string. During conflict resolution I initially lost the `User`, `Film`, and `CollectionEntry` classes from `models.py` (an editor issue during the rebase caused the file to be overwritten instead of merged) — caught this because `pytest` failed with `NameError: name 'db' is not defined`, and restored the missing classes in a follow-up commit.
**How I verified no conflict remains:** `git rebase --continue` completed with "Successfully rebased and updated refs/heads/feature/watchlist" and no remaining conflict markers. Ran `pytest tests/ -v` — all 7 tests pass.


### Commit history
![commit history](commit-history.png)

## PR Description

**What this feature does**
Adds a watchlist feature to CineLog, parallel to the existing collection feature. Users can add films they intend to watch, view their watchlist, and each entry is added via `POST /watchlist/<user_id>/add` and viewed via `GET /watchlist/<user_id>`.

**Design decisions**
- **Default visibility:** [state whichever you finalized — e.g. "Watchlist entries default to `public=False`. A watchlist reveals what a user is currently curious about, which is more exploratory and less curated than a completed collection, so I chose to default it private and let users opt in to sharing. Full reasoning in Comment 4 above."]
- **Sort order:** [state whichever you finalized — e.g. "The watchlist is sorted alphabetically by film title rather than by date added, since it's meant to be browsed as a lookup list rather than read as a chronological activity log. Full reasoning in Comment 5 above."]

**How to manually test**
1. Start the app: `python app.py`
2. Seed a user and film via the Flask shell or a quick script (no creation endpoint exists yet):
```python
   from app import create_app, db
   from models import User, Film
   app = create_app()
   with app.app_context():
       user = User(username="tester", email="tester@example.com")
       film = Film(title="Arrival", year=2016, genre="Sci-Fi")
       db.session.add_all([user, film])
       db.session.commit()
       print(user.id, film.id)
```
3. Add a film to the watchlist:
