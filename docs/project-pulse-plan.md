# Project Pulse implementation plan

## Goal

Build **Project Pulse**, a polished static dashboard that helps contributors quickly understand the state of Mona's team projects. The dashboard shows a card for each project with its name, owner, status, recent activity, and priority. It reads its data from a JSON file and runs locally from a VS Code launch configuration.

The Orchestrator coordinates the work, the **Designer** owns the visual and accessibility decisions, and the **Coder** owns the implementation.

## File assignments

| File | Owner | Purpose |
| --- | --- | --- |
| `app/project-data.json` | Coder | Sample data: a top-level `projects` array where each project has `name`, `owner`, `status`, `recentActivity`, and `priority`. |
| `app/index.html` | Coder (markup structure reviewed by Designer) | Page titled "Project Pulse". Links `styles.css`, fetches `project-data.json`, and renders one `.project-card` per project inside a `.dashboard` container. |
| `app/styles.css` | Designer | Polished, responsive, card-based styling: `.dashboard` and `.project-card` selectors, status and priority badges, `border-radius`, `box-shadow`, and readable typography and contrast. |
| `.vscode/launch.json` | Coder | Strict JSON (no comments) with a **Run Project Pulse Dashboard** configuration that runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. A `serverReadyAction` opens `http://localhost:%s/index.html`. |

## Implementation phases

### Phase 1: Data contract and design direction (parallel)

- **Coder:** create `app/project-data.json` with 4–6 realistic projects. Use a small, fixed set of status values (for example `On track`, `At risk`, `Blocked`) and priority values (`High`, `Medium`, `Low`).
- **Designer:** define the visual direction. This covers the layout grid, the card anatomy, badge colors for each status and priority value, typography, spacing, and accessibility rules such as color contrast and not relying on color alone.

### Phase 2: Markup and rendering (sequential, after Phase 1)

- **Coder:** build `app/index.html`. It needs:
  - semantic structure: `header`, `main.dashboard`, and an `article.project-card` for each project
  - a `<title>` and visible heading that read "Project Pulse"
  - a link to `styles.css`
  - a script that runs `fetch('project-data.json')` and renders each project's `status`, `recentActivity`, and `priority`
  - a visible error message if loading the data fails
  - CSS classes and data attributes that match the Designer's direction, for example `data-status` and `data-priority` on each card

### Phase 3: Styling (sequential, after Phase 2)

- **Designer:** build `app/styles.css` around the final markup class names, including:
  - `.dashboard`: responsive grid using `auto-fill` / `minmax` so it stacks to one column on narrow screens
  - `.project-card`: `border-radius`, `box-shadow`, and comfortable padding
  - status and priority badges
  - visible focus styles
  - `prefers-reduced-motion` handling

### Phase 4: Run configuration (parallel with Phase 3)

- **Coder:** create `.vscode/launch.json`. It has no file overlap with `styles.css`, so it can run alongside Phase 3.

### Phase 5: Integration and validation (sequential, last)

- **Orchestrator:** confirm that the markup, styles, data, and launch configuration fit together, then report the results.

## Dependencies

1. The markup in `app/index.html` depends on the data shape in `app/project-data.json`, including field names and status and priority values.
2. `app/styles.css` depends on the class names and data attributes used in `app/index.html`.
3. `.vscode/launch.json` depends only on the `app/` folder layout and on `index.html` being the entry point.
4. Validation depends on all four files being in place.

## Parallel work decisions

| Work | Parallel or sequential | Reason |
| --- | --- | --- |
| Data file (Coder) and design direction (Designer) | **Parallel** | They touch different files and don't depend on each other's output. |
| `index.html` after the data and design direction | Sequential | It needs the data contract and the class naming from the design direction. |
| `styles.css` (Designer) and `launch.json` (Coder) | **Parallel** | The file scopes don't overlap and neither depends on the other. |
| `styles.css` after `index.html` | Sequential | The styles target the final markup hooks. |
| Final integration | Sequential | It needs every file to be finished. |

No two agents edit the same file in the same phase.

## Edge cases

- `project-data.json` can't be fetched, for example when `index.html` is opened via `file://`. Show a readable error and tell the user to use the launch configuration.
- The `projects` array is empty. Show a "No projects yet" message instead of a blank page.
- A project is missing a field. Render a fallback such as "—" instead of `undefined`.
- Long project names or activity text. Wrap the text without breaking the card layout.
- Port 5500 is already in use. Note this in the validation output.

## Validation expectations

- `app/index.html` contains "Project Pulse", links `styles.css`, references `project-data.json`, and renders `.project-card` elements that show `status`, `recentActivity`, and `priority`.
- `app/styles.css` includes the `.dashboard` and `.project-card` selectors and uses `border-radius`, `box-shadow`, and a responsive layout.
- `app/project-data.json` parses as valid JSON (`python3 -m json.tool app/project-data.json`). It has a top-level `projects` key, and every project includes `name`, `owner`, `status`, `recentActivity`, and `priority`.
- `.vscode/launch.json` parses as strict JSON and contains **Run Project Pulse Dashboard**. Its `cwd` is `${workspaceFolder}/app`, it runs `python3 -m http.server 5500`, and it opens `http://localhost:%s/index.html`, not a directory listing.
- Manual check: run **Run Project Pulse Dashboard** from Run and Debug. The browser should show the styled dashboard with every project card, the layout should stay readable at mobile width, and there should be no errors in the browser console.

## Open questions

- Should the dashboard support filtering or sorting by status or priority? This plan leaves both out of scope.
- Should the dashboard support a dark theme? This plan leaves it out of scope unless the Designer recommends `prefers-color-scheme` support.
