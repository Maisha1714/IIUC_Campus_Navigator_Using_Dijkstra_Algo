# IIUC Campus Navigator — Dijkstra's Shortest Path
<img width="1600" height="894" alt="image" src="https://github.com/user-attachments/assets/00ac8d2f-2724-443d-b836-37cdbb1209af" />

A campus navigation web app for **International Islamic University Chittagong (IIUC)**,
Kumira. Pick a "From" and "To" location, and the app computes the shortest
walking route using **Dijkstra's algorithm**, draws it on a live map, and
animates the algorithm's execution step by step (which node gets finalized,
and how neighboring distances get updated) so you can *see* the algorithm
working, not just get the answer.

- **Backend:** Python (Flask) — Dijkstra implemented from scratch with `heapq`
- **Frontend:** HTML / CSS / JavaScript — Leaflet.js for the interactive map

---

## ⚠️ About the campus data

IIUC doesn't publish an official surveyed map with GPS coordinates for every
building. The 16 locations in `backend/graph_data.py` are a realistic
representation based on publicly known IIUC buildings (Language Building,
Science Building, Shariah & Islamic Studies Faculty, Central Library, Student
Hall, etc.), placed at reasonable positions around the real campus location
in Kumira, Chattogram. The walkway connections and distances are illustrative.

**To make it 100% accurate:** walk the real campus with a phone GPS app (or
pull coordinates from Google Maps), then update the `lat`/`lng` values in
`backend/graph_data.py`. Nothing else in the app needs to change — distances,
routing, and the map all recalculate automatically from those coordinates.

---

## Project structure

```
iiuc-campus-navigator/
├── backend/
│   ├── app.py            # Flask API (routes, CORS)
│   ├── dijkstra.py        # Dijkstra's algorithm (heapq-based priority queue)
│   ├── graph_data.py       # Campus locations (nodes) + walkways (edges)
│   └── requirements.txt
├── frontend/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   └── config.js          # points frontend at the backend URL
└── README.md
```

---

## Running it

### 1. Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
python app.py
```

The API runs at `http://127.0.0.1:5000`. Quick check:

```bash
curl "http://127.0.0.1:5000/api/route?source=main_gate&target=sports_ground"
```

### 2. Frontend

The frontend is plain static HTML/CSS/JS — no build step. Just serve the folder:

```bash
cd frontend
python -m http.server 5500
```

Then open **http://127.0.0.1:5500** in your browser. Leave the Flask backend
running in the other terminal.

> If you open `index.html` directly by double-clicking it (`file://`), the
> browser may block the API requests. Serving it via `http.server` (or any
> local server / VS Code Live Server) avoids that.

`frontend/config.js` controls which backend URL the frontend calls — change
`API_BASE_URL` there if you deploy the backend somewhere else.

---

## How it works

### The graph
Each campus building is a **node** (`backend/graph_data.py`). Each walkway
between two buildings is an **edge**, weighted by real walking distance in
meters — computed automatically from lat/lng coordinates with the Haversine
formula, so the weights stay accurate even if you move a building's
coordinates.

### The algorithm (`backend/dijkstra.py`)
Classic Dijkstra using a binary heap (`heapq`) as the priority queue:

1. Start at the source with distance `0`, everything else `∞`.
2. Repeatedly pop the un-finalized node with the smallest tentative distance.
3. Finalize it, then relax its neighbors — if going through the current node
   gives a shorter path, update that neighbor's tentative distance.
4. Stop once the target is finalized (or the queue is empty).
5. Reconstruct the path by walking backward through `previous[]` pointers.

Every visit and every distance update is recorded into a `steps` list, which
is exactly what the frontend replays to animate the algorithm.

### The API
| Endpoint | Description |
|---|---|
| `GET /api/locations` | All campus locations |
| `GET /api/edges` | Full walkway network (for drawing the base map) |
| `GET /api/route?source=X&target=Y` | Shortest path, distance, ETA, and algorithm trace |
| `GET /api/distances?source=X` | Distance from X to every other location (bonus: full single-source Dijkstra) |

### The frontend
- Leaflet.js renders the real campus map (OpenStreetMap tiles) with markers
  for every building.
- Click two markers (or use the dropdowns) to set From/To, then **Find
  Shortest Path**.
- The route is drawn in gold; the sidebar "Algorithm Trace" panel replays
  each step of Dijkstra's execution as a readable log, and markers light up
  as they're visited on the map — in real time, at adjustable speed.

---

## Ideas to extend it

- Add building floor plans / indoor routing for multi-story buildings.
- Weight edges by *time* instead of distance (e.g. stairs vs. flat path).
- Add an "avoid crowded areas" toggle that inflates certain edge weights.
- Compare Dijkstra vs. A* (using straight-line distance as a heuristic) on
  the same graph, side by side.
- Deploy the backend (Render/Railway) and frontend (GitHub Pages/Netlify)
  so classmates can actually use it.
