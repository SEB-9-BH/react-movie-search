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

## Why Vite 

Vite is fast in dev. Hot reload is instant.
Builds are smaller and quicker.

Cons exist. Fewer batteries included than something like Create React App.
Some plugins may need light config.

Think of Vite as a sports bike… CRA is a minivan.
Both get you there. One accelerates harder.

---

## Create the Project with Vite

```bash
# 1) Scaffold
npm create vite@latest react-movies -- --template react

# 2) Install deps
cd react-movies
npm install

# 3) Run dev server
npm run dev
```

You will see three main scripts.

* `npm run dev`… local dev server
* `npm run build`… production build
* `npm run preview`… preview the build

---

## Project Layout to Use

```
react-movies/
  index.html
  src/
    App.jsx
    main.jsx
    components/
      MovieDisplay.jsx
      Form.jsx
    App.css
    index.css
  .env
```

Vite uses `src/main.jsx` to mount your app.
`index.html` sits at the project root.

---

## Configure the OMDb API key

Create a `.env` file in the project root.

```env
VITE_OMDB_KEY=98e3fb1f
```

Vite exposes only vars prefixed with `VITE_`.
Never hardcode secrets in source when possible.

---

## src/main.jsx

```jsx
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

---

## src/App.jsx

```jsx
import React from "react";
import "./App.css";
import MovieDisplay from "./components/MovieDisplay.jsx";
import Form from "./components/Form.jsx";

export default function App() {
  // Like moving the mailroom to the lobby… lift shared state up
  const [movie, setMovie] = React.useState(null);
  const [loading, setLoading] = React.useState(false);
  const [error, setError] = React.useState("");

  const apiKey = import.meta.env.VITE_OMDB_KEY;

  const getMovie = async (searchTerm) => {
    if (!searchTerm) return;
    setLoading(true);
    setError("");
    try {
      const url = `https://www.omdbapi.com/?apikey=${apiKey}&t=${encodeURIComponent(
        searchTerm
      )}`;
      const res = await fetch(url);
      const data = await res.json();

      if (data.Response === "True") {
        setMovie(data);
      } else {
        setMovie(null);
        setError(data.Error || "Movie not found.");
      }
    } catch (e) {
      setError("Network error. Try again.");
      setMovie(null);
    } finally {
      setLoading(false);
    }
  };

  // useEffect is your scheduled task… run once on mount
  React.useEffect(() => {
    getMovie("Clueless");
  }, []);

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

## src/components/Form.jsx

```jsx
import React from "react";

export default function Form({ moviesearch }) {
  const [formData, setFormData] = React.useState({ searchterm: "" });

  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    moviesearch(formData.searchterm.trim());
  };

  return (
    <form onSubmit={handleSubmit} className="search">
      <label htmlFor="searchterm" className="sr-only">Search</label>
      <input
        id="searchterm"
        type="text"
        name="searchterm"
        placeholder="Search by title…"
        onChange={handleChange}
        value={formData.searchterm}
        aria-label="Movie title"
      />
      <button type="submit">Search</button>
    </form>
  );
}
```

---

## src/components/MovieDisplay.jsx

```jsx
import React from "react";

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

## src/index.css  …defaults

```css
:root {
  --bg: #0f172a;
  --panel: #111827;
  --text: #e5e7eb;
  --muted: #9ca3af;
  --accent: #22d3ee;
  --danger: #f87171;
}

* { box-sizing: border-box; }
html, body, #root { height: 100%; }

body {
  margin: 0;
  background: linear-gradient(180deg, #0b1020, #111827);
  color: var(--text);
  font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial;
}

.sr-only {
  position: absolute; width: 1px; height: 1px; margin: -1px;
  overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0;
}
```

---

## src/App.css  …simple styling

```css
.container {
  max-width: 960px;
  margin: 2rem auto;
  padding: 1rem;
}

h1 {
  text-align: center;
  letter-spacing: 0.5px;
}

.search {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 0.75rem;
  margin: 1.5rem 0 2rem;
}

.search input {
  padding: 0.8rem 1rem;
  border-radius: 8px;
  border: 1px solid #1f2937;
  background: #0b1324;
  color: var(--text);
  outline: none;
}

.search input:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(34,211,238,.15);
}

.search button {
  padding: 0.8rem 1rem;
  border-radius: 8px;
  border: 0;
  background: var(--accent);
  color: #05202b;
  font-weight: 700;
  cursor: pointer;
}

.card {
  display: grid;
  grid-template-columns: 180px 1fr;
  gap: 1rem;
  background: var(--panel);
  border: 1px solid #1f2937;
  border-radius: 12px;
  overflow: hidden;
}

.card img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.card-body {
  padding: 1rem 1rem 1.25rem 0;
}

.plot {
  color: var(--muted);
}

.muted { color: var(--muted); }
.error { color: var(--danger); font-weight: 600; }
```


## Test the API quickly

Try a direct request in your browser.
Use your key.

```
https://www.omdbapi.com/?apikey=YOURKEY&t=godfather
```

If it returns JSON with `Response: "True"`, you are good.
Keep it on https to avoid mixed content.

---

## Bonus… useEffect random on load

Replace the first `useEffect` with:

```jsx
useEffect(() => {
  const picks = ["Clueless", "The Matrix", "Black Panther", "Interstellar", "The Godfather"];
  const random = picks[Math.floor(Math.random() * picks.length)];
  getMovie(random);
}, []);
```

---

# Group Activity… Build a Mini Movie Explorer

You will work in squads.
Goal is a slightly more complex app using only OMDb and CSS.

### Features to add

1. Search results grid
   Use `s=` to search by keyword. Show poster and title.
   Support `page=` for pagination.

2. Details view
   Click a result. Fetch by `i=` using `imdbID`.
   Show a modal with plot, ratings, and cast.

3. Watchlist
   Add or remove from a watchlist.
   Persist in `localStorage`.

4. Styling polish
   Responsive grid.
   Hover states.
   Modal with backdrop.

### Suggested roles
* PM ... has the laptop, makes the fork and actually writes the code.
* API lead… writes `getResults` and `getById` helpers
* State lead… coordinates context or lifted state
* UI lead… grid, modal, buttons, responsive tweaks
* QA lead… tests edge cases and empty states
* You will soon learn Git Collaboration but for now you should Pair/Group Program

### Timebox

45 minutes build.
5 minutes to collaborate.
10 minutes to present.
Only one submission required per group.

### Acceptance criteria

* Typing a term shows a paged grid of results.
* Clicking a card opens a modal with details.
* A watchlist survives refresh.
* Layout looks clean on mobile and desktop.

### Starter snippets

Helpers:

```jsx
const API = import.meta.env.VITE_OMDB_KEY;

export async function getResults(term, page = 1) {
  const url = `https://www.omdbapi.com/?apikey=${API}&s=${encodeURIComponent(term)}&page=${page}`;
  const res = await fetch(url);
  const data = await res.json();
  if (data.Response === "True") return { items: data.Search, total: Number(data.totalResults) };
  return { items: [], total: 0, error: data.Error };
}

export async function getById(imdbID) {
  const url = `https://www.omdbapi.com/?apikey=${API}&i=${imdbID}&plot=full`;
  const res = await fetch(url);
  return await res.json();
}
```

Watchlist:

```jsx
const WATCHLIST_KEY = "watchlist";

export function loadWatchlist() {
  try { return JSON.parse(localStorage.getItem(WATCHLIST_KEY)) ?? []; }
  catch { return []; }
}

export function saveWatchlist(list) {
  localStorage.setItem(WATCHLIST_KEY, JSON.stringify(list));
}
```

Modal pattern:

```jsx
function Modal({ open, onClose, children }) {
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

CSS hints:

```css
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px,1fr)); gap: 1rem; }
.card-sm { background: var(--panel); border: 1px solid #1f2937; border-radius: 10px; overflow: hidden; cursor: pointer; }
.card-sm img { width: 100%; aspect-ratio: 2/3; object-fit: cover; }
.backdrop { position: fixed; inset: 0; background: rgba(0,0,0,.6); display: grid; place-items: center; padding: 1rem; }
.modal { background: #0b1324; border: 1px solid #1f2937; border-radius: 12px; max-width: 720px; width: 100%; padding: 1rem 1.25rem; }
.close { float: right; font-size: 1.5rem; background: transparent; border: 0; color: var(--text); cursor: pointer; }
.pager { display: flex; gap: .5rem; justify-content: center; margin: 1rem 0; }
```

Stretch goals if time allows.
Add a year filter.
Add a type filter using `type=movie|series|episode`.
Add keyboard navigation and focus traps for the modal.
Add debounced search.

---

## Key takeaways

Vite makes React dev fast.
Env vars use the `VITE_` prefix.

Lift shared state to the nearest common parent.
Think lobby mailroom.

`useEffect` runs when you tell it to.
Think scheduled task.

You styled a clean interface with pure CSS.
You extended the app with search, details, and a watchlist.
