# FEM pressure viewer 1.0

## Open and share

Extract the ZIP to a folder, then double-click **FEM-Pressure-Viewer.html**. Use current Microsoft Edge or Google Chrome with WebGL enabled. No Python, Node, local server, installation or internet connection is required. If HTML files open in an editor, right-click → Open with → Microsoft Edge.

Choose **Open FEM files**, or drag files into the viewer. Files are read locally; nothing is uploaded. You can load multiple files, switch the visible model using **Active model**, and retain selected histories across files. Closing/reloading the page clears the session.

Share the ZIP or its extracted contents. The HTML contains all viewer code and dependencies. Keep the third-party licence notice with it. Do not distribute node_modules, src, tests or input FEM files unless separately needed. Memory while viewing depends on the number and size of loaded datasets; the small download size is not a promise of low browser memory consumption. Remove datasets to release their references.

## Controls

| Action | Control |
|---|---|
| Rotate / pan / zoom | Left drag / right drag / mouse wheel |
| Select or deselect panel | Left click without dragging |
| Select a known element | Enter ID, then Add or Enter |
| Change model or load case | Left sidebar menus |
| Fit / standard views | Fit button or F; view menu |
| Step through time | Slider, previous/next buttons, left/right arrows |
| Jump to time | Type time; nearest available frame is selected |
| Play / pause | Play button or Space; 5, 15 or 30 stored frames/s |
| Jump to largest magnitude | Go to largest \|pressure\| (also selects that panel) |
| Zoom/pan history time axis | Wheel / drag over the plot; double-click to reset |
| Inspect original samples | Hover on the history plot; readout gives nearest sample values/times |
| Compare files | Select panels, open another file and select its panels |
| Export | CSV, viewport PNG, plot PNG buttons |

The right-hand cards identify each curve by colour, source file, load case and element ID, and report extrema and occurrence times. Click a card's element title to activate its model/case. Selected panels have yellow outlines. The dashed plot line marks the active model's current time. Panel hover reports the element ID, contour value and arithmetic mean of its corner coordinates.

## Interpretation

- Supported dialect: pCloud/USFOS text with NODE, QUADSHEL and `PressHist case Define LoadElem ID time pressure`. This is not a general reader for every format using the `.fem` extension.
- Static quadrilateral geometry; no displacements. Each quad is rendered as two triangles, both picking the original element ID. Coordinate values remain in file units.
- Raw FEM sign is the default. **Reversed** multiplies pressures by −1; it is a global display convention, not an element-normal correction. The existing Python 3D viewer used the reversed convention by default.
- Input pressure is assumed to be **Pa**, based on this project's existing documentation. Pa/kPa/MPa controls convert values, including exports. The time label defaults to **file units**. Selecting seconds changes the label only; it does not rescale time or verify units.
- Histories and CSV retain original samples and grids. Instantaneous contours use the union of stored times for the chosen load case, with linear interpolation between an element's samples and no extrapolation outside its range.
- Missing histories or times are grey and reported as missing, never zero. The jacket example has 40 panels with no explicit history.
- Min/max envelopes apply the selected sign before computing extrema. Absolute peak is a magnitude. Different panels' extrema may occur at different times; an envelope is not a simultaneous load field.
- Fixed colour limits use the full load-case pressure range (0 to largest magnitude for absolute envelopes). Frame/field scaling changes colour meaning as time changes. Manual limits clip colours but not reported/exported values. Invalid manual limits temporarily use fixed limits, with a warning and an explicit fallback legend; data and sign/unit controls remain live.
- Playback advances stored frames, not physical elapsed time. The supplied files contain widely spaced trailing zero-pressure samples; the slider and plots preserve them.
- Cross-file curves share their original time axes. No matching of elements, meshes, time origins or load events is inferred. Only one mesh is shown at a time. No difference field is calculated for unrelated meshes.
- Solver `PressHist Assign`, load/time multipliers and distance-based mapping are not evaluated. Other metadata records are ignored and listed in the model-info tooltip. Structural forces, areas, impulses and response are not calculated.
- CSV uses long form: file, load_case, element_id, time, pressure, pressure_unit, sign, time_unit, status. Each original sample has one row. Selected missing histories have a row with blank time/pressure and `missing history` status. PNGs include identifying annotations; history PNGs include the curve legend.
- Invalid references, duplicate IDs/histories, nonfinite numbers and non-increasing times produce an error. Input files remain unchanged.

## Developer notes (source folder only)

The release needs no development tools. To rebuild from source, run `pnpm install --frozen-lockfile --ignore-scripts`, then `node build.mjs`. The platform esbuild binary is supplied as an optional package. Build dependencies: Three.js 0.180.0 and esbuild 0.25.10. Do not include development dependencies in the release.

Run `node --test tests/core.test.mjs` for analytic parser/numerical checks. Run `tests/generate_reference.py` with the project's Anaconda interpreter, then `node tests/real-files.mjs` to compare both input files with the existing Python reader. `node tests/browser.mjs` uses Playwright and installed Edge to test the actual release through file:// with network requests blocked. PLAYWRIGHT_PATH can point to another local Playwright installation, BROWSER_PATH to Chrome/Edge, and FEM_VIEWER_HTML to an extracted release file.
