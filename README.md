# DiagramLab-IEM 📐

**A free, browser-based UML & DFD diagramming studio with rule-based code generation — built for Software Engineering students.**

🔗 **Live tool:** `[index.html](https://drajdeep.github.io/DiagramLab-IEM/)`

Made with ❤️ by **Rajdeep Das** (Dept. of CSE-AIML, IEM-UEM Kolkata) for the students of the **Institute of Engineering and Management, Kolkata**.

---

## 1. Why I Built This (Purpose)

While studying the **Software Engineering** subject, we constantly need to draw DFDs and UML diagrams and understand how designs map to code. The existing options all have problems:

- **Visual Paradigm, StarUML, Enterprise Architect** — powerful, but **paid** (or crippled trial versions with watermarks).
- **Free tools like draw.io** — good for drawing, but they have **no code generation or reverse engineering** at all.
- **Umbrello and similar desktop tools** — need installation, are platform-dependent, and are not accessible from a lab machine or phone with one click.

No single free tool gave students **everything in one place**: proper notation palettes for *all* the diagrams taught in the syllabus, drag-and-drop editing, export for assignments/reports, **diagram → code**, and **code → diagram** — with **zero cost, zero installation, and zero sign-up**.

So I built DiagramLab-IEM: **one HTML file**, hosted free on GitHub Pages, that any student can open in a browser and start working immediately. It works offline after the first load (only the PDF export library comes from a CDN), stores nothing on any server, and costs nothing no matter how many students use it.

---

## 2. Features

| Area | What you get |
|---|---|
| **Diagram types** | Class, Use Case, Sequence, Activity, State Chart, Component/Structured, and Data Flow Diagram (Yourdon/DeMarco notation) |
| **Editing** | Drag & drop from palette, click-to-place, move, resize, double-click rename, properties panel, Delete key, Undo (Ctrl+Z), zoom, snap-to-grid |
| **Connectors** | Inheritance ▷, Realization (dashed ▷), Association, Aggregation ◇, Composition ◆, Dependency, «include», «extend», data flows, transitions with `event [guard] / action`, sync/async/return messages |
| **Sequence extras** | Object & actor lifelines with dashed lines, **activation bars that snap to the nearest lifeline**, self-message loops |
| **Diagram → Code** | Rule-based generation to **Java, C++ and Python** for every diagram type |
| **Code → Diagram** | Paste Java / C++ / Python source → auto-built class diagram with inheritance & interfaces detected |
| **Export** | PNG (2× resolution), PDF (A4, auto orientation), JSON save/load for continuing work later |
| **Cost** | Free forever. No AI, no API keys, no accounts, no server. |

---

## 3. Tech Stack (Detailed)

| Layer | Technology | Why |
|---|---|---|
| Structure | **Plain HTML5** | Single-file deployment on GitHub Pages; no build step, no framework |
| Styling | **Pure CSS3** (flexbox, CSS variables) | Academic white/blue/black theme; grid background via CSS gradients |
| Diagram canvas | **SVG (Scalable Vector Graphics)** | Crisp shapes at any zoom; markers for arrowheads; easy PNG/PDF export |
| Logic | **Vanilla JavaScript (ES6+)** | No dependencies; template literals build all SVG; regex-based parsers |
| PDF export | **jsPDF 2.5.1** (cdnjs) | Only external library; converts the exported bitmap into an A4 PDF |
| Persistence | **JSON file download/upload** | Diagrams are serialized and restored fully; no server storage |
| Hosting | **GitHub Pages** | Free static hosting; the whole app is `index.html` |

There is **no backend, no database, no AI, and no API**. Everything — rendering, editing, code generation, parsing — runs 100% in the student's browser.

---

## 4. How Everything Works (Technology Behind Each Feature)

### 4.1 The canvas and rendering pipeline
The drawing area is one large `<svg>` element (3000×2000). The application keeps a single JavaScript **state object**:

```js
state = { diagramType, nodes: [...], edges: [...] }
```

Every node is plain data — `{id, type, x, y, w, h, label, attrs, methods}` — and every connector is `{id, type, from, to, label}` referring to node ids. After **any** change, a `render()` function rebuilds the SVG markup from this state using template literals. This "state → re-render" pattern (the same core idea behind React) means the picture on screen is always a pure function of the data, which is also exactly what gets saved to JSON and exported.

Each element type has its own drawing routine: a class is a 3-compartment rectangle whose height is **auto-computed** from the number of attributes/methods; a DFD process is a circle; a use case is an ellipse; an actor is a stick figure built from lines and a circle; fork/join bars are filled rectangles; notes are folded-corner paths.

### 4.2 Drag & drop, moving, and resizing
Two placement paths exist:
- **HTML5 Drag & Drop API** — every palette item has `draggable="true"`; on `dragstart` it stores its element type in `dataTransfer`; the canvas's `drop` handler reads it, converts the mouse position to SVG coordinates, and pushes a new node into state.
- **Click-to-place** — clicking a palette item arms a "pending element"; the next canvas click places it (works on touch devices too).

**Moving:** `mousedown` on a node records the offset between the cursor and the node's corner; `mousemove` updates `node.x/y` (rounded to a 10 px grid — this is the snap-to-grid) and re-renders; `mouseup` ends the drag. Coordinates are corrected for the current zoom by dividing by the zoom factor.

**Resizing:** when a node is selected, a small blue handle is drawn at its bottom-right corner. Dragging that handle switches the drag mode to `resize` and updates `node.w/h` instead, with minimum sizes enforced.

**Container dragging:** system boundaries and packages detect which nodes lie geometrically inside them, and moving the container moves its members together — done by storing each member's relative offset at drag start.

### 4.3 Connectors and arrowheads
Selecting a connector in the palette puts the app in **connect mode** (crosshair cursor). The first node you click becomes the source; the second becomes the target; an edge object is added.

Each edge is drawn as a line from border to border — an `anchor()` function computes where the center-to-center line **intersects the node's boundary** (rectangle clipping for boxes, radius math for circles, ellipse math for use cases), so arrows touch shapes cleanly instead of starting at centers.

UML arrowheads are **SVG `<marker>` definitions**: hollow triangle (inheritance/realization), filled diamond (composition), hollow diamond (aggregation), filled arrow (sync message), open arrow (everything else). Dashed strokes mark dependencies, realizations, «include»/«extend» and return messages. A transparent 12-px-wide "hit line" is drawn under every visible edge so thin lines are easy to click and select.

### 4.4 Sequence diagrams and lifelines
Lifelines are nodes with a header box and a **dashed vertical line**. Messages are special edges: instead of connecting borders, they are drawn **horizontally** at the y-coordinate where you clicked the source lifeline, so message ordering is visual and natural. A self-message (source = target) is drawn as a small rectangular loop. **Activation bars** are thin rectangles that, on placement or drag, automatically **snap to the nearest lifeline's x-center** (within 120 px) and clamp themselves onto that line — implemented by measuring the distance to every lifeline and repositioning.

### 4.5 Diagram → Code (rule-based generation)
No AI is involved — generation is **pure deterministic rules**, the same approach used by Umbrello and classic CASE tools. The diagram's data model *is* the program's structure:

- **Class diagram:** each attribute line like `+name: String` is parsed by a small parser (`+`/`−`/`#` → public/private/protected, text after the last `:` → type). Method lines like `+enroll(course: String): bool` are split into name, parameter list, and return type. Then:
  - an **Inheritance** edge becomes `extends` (Java), `: public Base` (C++), or `class Child(Base):` (Python);
  - a **Realization** edge becomes `implements` / pure-virtual base / duck-typed base;
  - **Association/Dependency** edges become member fields referencing the other class;
  - **Aggregation/Composition** put the *part* as a field inside the *whole* (pointer vs value semantics commented in C++);
  - a type-mapping table converts UML types per language (`String → std::string → str`), and non-void methods get correct default `return` stubs.
- **State chart:** states become an `enum`; every transition label is parsed with the regex pattern `event [guard] / action`, producing a `handle(event)` **state machine** — a `switch` in Java, `if/else` chain in C++, and a transition dictionary in Python. The transition leaving the Initial State picks the starting state.
- **Use case diagram:** each **actor** becomes an interface (`StudentActions`) and every use case linked to it by an association becomes a camelCased operation (`registerCourse()`); «include»/«extend» relationships are emitted as documentation comments.
- **Sequence diagram:** messages are sorted by their **y-coordinate** (top-to-bottom = execution order); each lifeline `obj: Class` declares a variable; each message `login(user, pass)` becomes a real method call on the target object; async and return messages are annotated.
- **Activity diagram:** a **depth-first traversal** starts at the Initial Node and follows control-flow edges; actions become function calls, decision nodes become `if/elif`/`else-if` blocks using the **edge labels as conditions**, fork bars are marked as parallel sections, and a visited-set prevents infinite loops in cyclic flows.
- **DFD:** each process becomes one function; **incoming data flows become its parameters** and outgoing flows its outputs; flows touching data stores generate read/write comments — directly teaching the "process = transformation of data" idea.
- **Component diagram:** components become classes/modules; dependency edges become held references; provided interfaces become interface definitions.

### 4.6 Code → Diagram (reverse engineering)
Also fully rule-based, using **regular-expression parsers** per language:

- **Java:** matches `class/interface Name extends X implements Y, Z {`, then a brace-counting routine extracts the exact class body, from which field declarations (`private int rollNo;`) and method signatures are pulled and converted back into UML notation (`-rollNo: int`).
- **C++:** matches `class Name : public Base {`, tracks the current `public:/private:/protected:` section line-by-line, and recognizes both member functions and data members.
- **Python:** matches `class Name(Base):`, collects the indented body, reads `self.x = ...` assignments in `__init__` (leading underscore → private), and `def` signatures with type hints.

The detected classes are laid out automatically on a grid, and inheritance/interface edges are created by **matching parent names to detected class names** — instantly giving students a class diagram of any code they paste.

### 4.7 Export: PNG, PDF, JSON
- **PNG:** the current SVG markup is wrapped with a white background rectangle and a `viewBox` cropped to the **bounding box of all elements** (computed from node coordinates), serialized to a data-URL, drawn onto an off-screen `<canvas>` at **2× scale** for sharpness, and downloaded via `canvas.toBlob()`.
- **PDF:** the same bitmap is placed onto an A4 page by **jsPDF**, automatically choosing landscape or portrait based on the diagram's aspect ratio, scaled to fit, with a title header.
- **JSON:** `JSON.stringify(state)` → downloadable file; loading validates the structure and restores nodes, edges and diagram type exactly — so work can be saved and continued across sessions with no server.

### 4.8 Undo & keyboard support
Before every mutation, a JSON snapshot of the state is pushed onto an **undo stack** (capped at 60 entries). Ctrl+Z pops and restores. Esc cancels any pending placement/connection; Delete removes the selection along with any connectors attached to it.

---

## 5. How to Use

1. Open the tool (GitHub Pages URL).
2. Pick a **diagram type** from the dropdown — the palette changes to the correct notation.
3. **Drag** elements onto the canvas (or click an element, then click the canvas).
4. Pick a **connector**, click the **source** element, then the **target**.
5. Select anything to edit it in the **Properties panel**; double-click for quick rename.
6. **Generate Code** → choose Java / C++ / Python → copy or download.
7. **Code → Diagram** → paste source code → get a class diagram instantly.
8. **Export** PNG for assignments, PDF for reports, JSON to continue later.

## 6. Deploying Your Own Copy

1. Fork or clone this repo (`DiagramLab-IEM`).
2. Ensure `index.html` is at the repo root.
3. Repo **Settings → Pages → Deploy from branch → main → / (root)**.
4. Your tool is live at `https://<username>.github.io/DiagramLab-IEM/`.

## 7. License

Released under the **MIT License** — free to use, modify and share. See [LICENSE](LICENSE).

---

*Made for the Software Engineering subject. Made with ❤️ by Rajdeep Das (Dept. of CSE-AIML, IEM-UEM Kolkata) for the students of the Institute of Engineering and Management, Kolkata.*
