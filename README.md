# React Movie Search… with Vite

<div style="display: flex; justify-content: space-between;">

<p><img width="862" height="342" alt="Untitled Diagram drawio" src="https://github.com/user-attachments/assets/a3d67539-f43f-4923-a6f5-7dfce418b427" /></p>
<p><img src="https://i.imgur.com/TGZKfoI.png" alt="components"></p>

</div>

### Lesson to be Learned

* Making AJAX requests in a React app
* Lifting state that is shared by components
* Using the `useEffect` hook

---

# React Movie Search… with Vite

---

## Lesson Outcomes

* Fetch data from an external API
* Manage state across components
* Design a clean component hierarchy
* Pass props down… lift shared state up
* Use `useEffect` to model lifecycle
* Understand how `useEffect` changes state and triggers renders

---

## How SPAs and React work… using the diagram

**Flow.** The visitor loads one HTML shell. React mounts in the browser. Components render UI. React calls APIs with `fetch`… no full-page reload.
Node and Express answer those API calls. Express talks to a database like MongoDB. JSON comes back. React updates the DOM. It feels instant.

**Mental model.** React is the front desk. Express is the back office. MongoDB is the file room. `fetch` is the courier moving messages both ways.

---

## Why Vite for this lab

It starts instantly. It rebuilds fast. It keeps focus on React, not tooling.
Vite exposes env vars that start with `VITE_`… perfect for our API key.

---

# Part 0… Setup

### 0.1 Create the project

```bash
npm create vite@latest react-movies -- --template react
cd react-movies
npm install
npm run dev
```

### 0.2 Add the API key

Create `.env` in the project root.

```env
VITE_OMDB_KEY=YOUR_KEY_HERE
```

### 0.3 Folder plan

```
react-movies/
  index.html
  src/
    main.jsx
    App.jsx
    components/
      Form.jsx
      MovieDisplay.jsx
    App.css
    index.css
  .env
```

---

# Part 1… Starter to First Render

### 1.1 Clean `main.jsx`

```jsx
// src/main.jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### 1.2 Start with a tiny `App`

```jsx
// src/App.jsx
import "./App.css";

export default function App() {
  return (
    <div className="container">
      <h1>React Movie Search</h1>
      <p className="muted">We will add features step by step…</p>
    </div>
  );
}
```

**Checkpoint.** App renders. No components yet.

---

# Part 2… Component Hierarchy

**Goal.** Design the tree before writing logic.

```
App
 ├─ Form                // user types a title… submits
 └─ MovieDisplay        // shows loading… error… or the movie
```

Create empty components.

```jsx
// src/components/Form.jsx
export default function Form() {
  return <form className="search">Form goes here…</form>;
}
```

```jsx
// src/components/MovieDisplay.jsx
export default function MovieDisplay() {
  return <p className="muted">Results render here…</p>;
}
```

Wire them in.

```jsx
// src/App.jsx
import "./App.css";
import Form from "./components/Form.jsx";
import MovieDisplay from "./components/MovieDisplay.jsx";

export default function App() {
  return (
    <div className="container">
      <h1>React Movie Search</h1>
      <Form />
      <MovieDisplay />
    </div>
  );
}
```

---

# Part 3… State Management in the Parent

**Why here.** Both `Form` and `MovieDisplay` care about the same data.
Keep shared state at the nearest common parent.

Add parent state and a search function.

```jsx
// src/App.jsx
import { useState, useEffect } from "react";
import "./App.css";
import Form from "./components/Form.jsx";
import MovieDisplay from "./components/MovieDisplay.jsx";

export default function App() {
  const [movie, setMovie] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  const apiKey = import.meta.env.VITE_OMDB_KEY;

  async function getMovie(searchTerm) {
    if (!searchTerm) return;
    setLoading(true);
    setError("");
    try {
      const url = `https://www.omdbapi.com/?apikey=${apiKey}&t=${encodeURIComponent(searchTerm)}`;
      const res = await fetch(url);
      const data = await res.json();

      if (data.Response === "True") {
        setMovie(data);
      } else {
        setMovie(null);
        setError(data.Error || "Movie not found.");
      }
    } catch {
      setMovie(null);
      setError("Network error. Try again.");
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="container">
      <h1>React Movie Search</h1>
      <Form moviesearch={getMovie} />
      <MovieDisplay movie={movie} loading={loading} error={error} />
    </div>
  );
}
```

---

# Part 4… Passing Props into `Form` and Controlling Input

```jsx
// src/components/Form.jsx
import { useState } from "react";

export default function Form({ moviesearch }) {
  const [formData, setFormData] = useState({ searchterm: "" });

  function handleChange(e) {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  }

  function handleSubmit(e) {
    e.preventDefault();
    moviesearch(formData.searchterm.trim());
  }

  return (
    <form onSubmit={handleSubmit} className="search">
      <label htmlFor="searchterm" className="sr-only">Search</label>
      <input
        id="searchterm"
        name="searchterm"
        type="text"
        placeholder="Search by title…"
        value={formData.searchterm}
        onChange={handleChange}
        aria-label="Movie title"
      />
      <button type="submit">Search</button>
    </form>
  );
}
```

---

# Part 5… Display States with `MovieDisplay`

```jsx
// src/components/MovieDisplay.jsx
export default function MovieDisplay({ movie, loading, error }) {
  if (loading) return <p className="muted">Loading…</p>;
  if (error) return <p className="error">{error}</p>;
  if (!movie) return <p className="muted">Search for a movie to begin.</p>;

  return (
    <article className="card">
      <img src={movie.Poster} alt={movie.Title} />
      <div className="card-body">
        <h2>{movie.Title}</h2>
        <p>{movie.Genre} • {movie.Year}</p>
        <p><strong>Rated:</strong> {movie.Rated} • <strong>Runtime:</strong> {movie.Runtime}</p>
        <p className="plot">{movie.Plot}</p>
      </div>
    </article>
  );
}
```

---

# Part 6… `useEffect` as Lifecycle

```jsx
import { useState, useEffect } from "react";

// inside App
useEffect(() => {
  getMovie("Clueless");
}, []);
```

* Mount → runs once after first render
* Update → runs when dependencies change
* Unmount → cleanup runs

---

# Part 7… `useEffect` impact on State

* Calling `setState` in `useEffect` triggers re-render.
* No dependency array = runs after every render.
* Add dependencies to control when it runs.

---

# Part 8… Lifting State vs Local State

* Input text stays local in `Form`
* Movie data lives in `App` for sharing

<img width="1024" height="1024" alt="movie display" src="https://github.com/user-attachments/assets/9752a549-ff92-4ac5-a12a-ac212b76701b" />

* **App** as the root owning state: `movie`, `loading`, `error`, and `getMovie(searchTerm)`.
* **Form** and **MovieDisplay** as **siblings** beneath **App**.
* **Props down / events up**:

  * `Form` receives `{ moviesearch: getMovie }`. On submit it calls the parent.
  * `MovieDisplay` receives `{ movie, loading, error }`.
* **Fetch & lifecycle**:

  * `useEffect` in **App** optionally calls `getMovie('Clueless')` on mount.
  * `getMovie` sets `loading → true`, fetches OMDb, sets `movie` or `error`, then `loading → false`.
* **Re-render rule**:

  * When **Form** triggers `getMovie` and **App** updates state, **App** re-renders and any child that consumes changed props re-renders… so **MovieDisplay** updates automatically.

---

**Goal**
In teams, extend the Movie Search app into a **Mini Movie Explorer** using the OMDb API. Add whatever features you want using the OMDB API Docs below are SOME IDEAS, and Acceptance Criteria, you do't time to do all of them, but try it out.

**Features to build**

1. **Search Results Grid**

   * Use the `s=` endpoint to fetch multiple results.
   * Display posters and titles in a responsive grid.
   * Support pagination with `page=` parameter.

2. **Details View**

   * Click a grid card to fetch full details via `i=` (imdbID).
   * Show details in a **modal** (title, plot, ratings, cast).

3. **Watchlist**

   * Add/remove movies to a watchlist.
   * Store it in `localStorage` so it survives refresh.

4. **Styling Polish**

   * Responsive design for mobile and desktop.
   * Hover states and clean UI.
   * Backdrop blur for modal.

**Roles** (optional suggestion)

* **PM** – Runs the repo, coordinates tasks.
* **API Lead** – Writes fetch helpers for search & details.
* **State Lead** – Manages watchlist and global state.
* **UI Lead** – Builds grid, modal, styling.
* **QA Lead** – Tests edge cases and empty states.

**Timebox**

* 45 min build
* 5 min finalize
* 10 min presentation

**Acceptance Criteria**

* Typing in search updates the results grid.
* Clicking a card opens details modal.
* Watchlist works and persists.
* Layout looks good on both mobile and desktop.


# Helper 1: OMDb API functions

`src/lib/omdb.js`

```jsx
const API = import.meta.env.VITE_OMDB_KEY;

export async function searchMovies(term, page = 1) {
  if (!term) return { items: [], total: 0 };
  const url = `https://www.omdbapi.com/?apikey=${API}&s=${encodeURIComponent(term)}&page=${page}`;
  const res = await fetch(url);
  const data = await res.json();
  if (data.Response === "True") {
    return { items: data.Search, total: Number(data.totalResults) };
  }
  return { items: [], total: 0, error: data.Error || "No results" };
}

export async function getMovieById(id) {
  const url = `https://www.omdbapi.com/?apikey=${API}&i=${id}&plot=full`;
  const res = await fetch(url);
  return await res.json();
}
```

---

# Helper 2: Watchlist storage

`src/lib/watchlist.js`

```jsx
const KEY = "watchlist";

export function loadWatchlist() {
  try { return JSON.parse(localStorage.getItem(KEY)) ?? []; }
  catch { return []; }
}

export function saveWatchlist(list) {
  localStorage.setItem(KEY, JSON.stringify(list));
}

export function toggleWatch(list, movie) {
  const exists = list.some(m => m.imdbID === movie.imdbID);
  return exists ? list.filter(m => m.imdbID !== movie.imdbID) : [...list, movie];
}
```

---

# Helper 3: Modal

`src/components/Modal.jsx`

```jsx
export default function Modal({ open, onClose, children }) {
  if (!open) return null;
  return (
    <div className="backdrop" onClick={onClose}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        <button className="close" onClick={onClose}>×</button>
        {children}
      </div>
    </div>
  );
}
```

---

# Helper 4: Results Grid

`src/components/ResultsGrid.jsx`

```jsx
export default function ResultsGrid({ items, onSelect, onToggleWatch, watchlist }) {
  return (
    <div className="grid">
      {items.map(m => {
        const inList = watchlist.some(x => x.imdbID === m.imdbID);
        return (
          <article key={m.imdbID} className="card-sm">
            <img src={m.Poster !== "N/A" ? m.Poster : "/placeholder.png"} alt={m.Title} />
            <div style={{ padding: ".5rem .75rem" }}>
              <strong>{m.Title}</strong>
              <div style={{ display: "flex", gap: ".5rem", marginTop: ".5rem" }}>
                <button onClick={() => onSelect(m.imdbID)}>Details</button>
                <button onClick={() => onToggleWatch(m)}>{inList ? "Remove" : "Watch"}</button>
              </div>
            </div>
          </article>
        );
      })}
    </div>
  );
}
```

---

# Helper 5: Pager

`src/components/Pager.jsx`

```jsx
export default function Pager({ page, pages, onPage }) {
  if (pages <= 1) return null;
  return (
    <div className="pager">
      <button disabled={page <= 1} onClick={() => onPage(page - 1)}>Prev</button>
      <span>{page} / {pages}</span>
      <button disabled={page >= pages} onClick={() => onPage(page + 1)}>Next</button>
    </div>
  );
}
```

---

# Minimal activity wiring

Drop this inside your `App.jsx` version for the activity section.

```jsx
import { useState, useEffect } from "react";
import { searchMovies, getMovieById } from "./lib/omdb.js";
import { loadWatchlist, saveWatchlist, toggleWatch } from "./lib/watchlist.js";
import ResultsGrid from "./components/ResultsGrid.jsx";
import Pager from "./components/Pager.jsx";
import Modal from "./components/Modal.jsx";

export default function App() {
  const [query, setQuery] = useState("");
  const [page, setPage] = useState(1);
  const [items, setItems] = useState([]);
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");
  const [selected, setSelected] = useState(null);
  const [details, setDetails] = useState(null);
  const [watch, setWatch] = useState(loadWatchlist());

  useEffect(() => { saveWatchlist(watch); }, [watch]);

  useEffect(() => {
    let ignore = false;
    async function run() {
      if (!query) return;
      setLoading(true);
      setError("");
      const { items: result, total: t, error: err } = await searchMovies(query, page);
      if (!ignore) {
        setItems(result);
        setTotal(t);
        if (err) setError(err);
        setLoading(false);
      }
    }
    run();
    return () => { ignore = true; };
  }, [query, page]);

  useEffect(() => {
    let ignore = false;
    async function run() {
      if (!selected) return;
      const data = await getMovieById(selected);
      if (!ignore) setDetails(data);
    }
    run();
    return () => { ignore = true; };
  }, [selected]);

  const pages = Math.max(1, Math.ceil(total / 10));

  return (
    <div className="container">
      <h1>Mini Movie Explorer</h1>

      <form
        onSubmit={(e) => { e.preventDefault(); setPage(1); setQuery(e.currentTarget.term.value.trim()); }}
        className="search"
      >
        <label htmlFor="term" className="sr-only">Search</label>
        <input id="term" name="term" placeholder="Search movies…" />
        <button type="submit">Search</button>
      </form>

      {loading && <p className="muted">Loading…</p>}
      {error && <p className="error">{error}</p>}

      {!loading && items.length > 0 && (
        <>
          <ResultsGrid
            items={items}
            onSelect={setSelected}
            onToggleWatch={(m) => setWatch(prev => toggleWatch(prev, m))}
            watchlist={watch}
          />
          <Pager page={page} pages={pages} onPage={setPage} />
        </>
      )}

      <h2>Watchlist</h2>
      <ul>
        {watch.map(w => (
          <li key={w.imdbID}>
            {w.Title} <button onClick={() => setWatch(prev => toggleWatch(prev, w))}>Remove</button>
          </li>
        ))}
      </ul>

      <Modal open={Boolean(selected)} onClose={() => { setSelected(null); setDetails(null); }}>
        {!details ? <p className="muted">Loading details…</p> : (
          <>
            <h3>{details.Title} ({details.Year})</h3>
            <p>{details.Plot}</p>
            <p><strong>Actors:</strong> {details.Actors}</p>
            <p><strong>Ratings:</strong> {details.Ratings?.map(r => `${r.Source} ${r.Value}`).join(" · ")}</p>
          </>
        )}
      </Modal>
    </div>
  );
}
```

---

# CSS helpers

```css
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px,1fr)); gap: 1rem; }
.card-sm { background: var(--panel); border: 1px solid #1f2937; border-radius: 10px; overflow: hidden; }
.card-sm img { width: 100%; aspect-ratio: 2/3; object-fit: cover; }
.backdrop { position: fixed; inset: 0; background: rgba(0,0,0,.6); display: grid; place-items: center; padding: 1rem; }
.modal { background: #0b1324; border: 1px solid #1f2937; border-radius: 12px; max-width: 720px; width: 100%; padding: 1rem 1.25rem; }
.close { float: right; font-size: 1.5rem; background: transparent; border: 0; color: var(--text); cursor: pointer; }
.pager { display: flex; gap: .5rem; justify-content: center; margin: 1rem 0; }
```

---


