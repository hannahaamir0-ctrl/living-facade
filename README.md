# Living Facade

A project website and playable simulation of a fleet of maintenance drones keeping a vertical green-wall facade alive.
Static files only, no build step and no server:

- `index.html` is the project site: the concept, how the drones decide, the propose-and-refuse loop, what is measured, and the playground checklist.
- `play.html` is the playground itself.

**Live link:** _add your GitHub Pages URL here once it is published_

## What the deliverable asks for, and where it is

| Requirement | Where it is in the playground |
|---|---|
| **(a) Live simulation of the behavior** | The wall elevation at the top of the page. 140 planted modules (10 levels × 14 bays) dry out, lose nutrients, get pests and decay. Up to 10 drones fly along the joints of the grid from a roof dock, carrying water and fertilizer, using local rules only. Runs at 1×–20× speed. |
| **(b) Propose mechanism** | **Suggest next move** button in the *Propose a move* panel. It turns the live state into a prompt for a language model and brings the model's moves onto the board. See *The prompt ritual* below. |
| **(c) Visible refusal** | Every proposed move is checked against the simulation before it runs. A move that cannot work is refused and drawn on the wall: the failing route in red dashes, an **X** where the battery runs out, the drones or modules involved circled, the overflowing dock pads drawn dashed, plus a red label with the refusal code and a message under the wall. |
| **(d) Score or trace** | Side panel: healthy modules, season score, a healthy-over-time chart. *Trace* panel: what survived (healthy, dead, revived), jobs done by type, throughput (jobs per game day), coverage (share of modules tended in the last 24 game hours), proposals accepted and refused by reason. **Copy trace as JSON** exports all of it with the full proposal log. |

## The prompt ritual

There are two ways to bring a model's proposals onto the board. Both use the same prompt and the same proposal format.

**With a Claude API key (one click)**
1. Paste your key into *Claude API key*. It is held only in that input field for the current tab and sent only to the Claude API.
2. Press **Suggest next move**. The simulation pauses, the page sends the prompt to Claude (`claude-opus-5-5`, low effort), puts the reply in the *Proposal* box, applies it, and resumes.

**With any chatbot (no key)**
1. Press **Suggest next move** (or **Copy prompt**). The prompt is copied to the clipboard. If the browser blocks copying, it is selected in *Show the last prompt*.
2. Paste it into any chatbot (Claude, or any other language model).
3. Paste the reply into *Proposal* and press **Apply proposal**. Text around the JSON, and ```` ```json ```` fences, are ignored.

The prompt contains: how the wall and drones work, the exact refusal rules, the current time and weather, the settings, every drone's position, job, battery, water and fertilizer, and the 24 most urgent modules. Open *Show the last prompt* to see exactly what was sent.

**Try an impossible move** sends a deliberately bad proposal, picked from what is impossible right now, so you can see a refusal immediately.

## Proposal format

```json
{"moves": [
  {"type": "dispatch", "drone": "D3", "module": "L7·B4", "task": "water", "why": "soil at 12%"},
  {"type": "set", "param": "fleet", "value": 8, "why": "falling behind"},
  {"type": "flag", "module": "L2·B11", "why": "outside every drone's reach"},
  {"type": "wait", "why": "fleet is coping"}
]}
```

- One to three moves per proposal.
- Modules are named by level and bay. L1 is the street level, L10 the top. B1 is the left bay, B14 the right. `L7·B4`, `L7-B4` and `L7 B4` all work.
- `task` is one of `water`, `feed`, `treat`, `revive`.
- `param` is one of `fleet` (1–10), `trigger` (water trigger, 20–70%) or `reach` (sensing reach, 2–15 modules).
- An accepted dispatch makes the drone drop its current job, claim the module and fly there. Its route is drawn in blue, tagged with the proposal number.

## Refusal codes

| Code | Refused when | Shown on the wall as |
|---|---|---|
| `UNREACHABLE` | The drone's battery can't cover the trip, the job and the flight home. | The full route in red dashes, with an X where the battery would run flat. |
| `JAM` | Another drone has already claimed the module. | Both drones circled and linked, the module outlined. |
| `BUSY` | The drone is in the middle of a job. | The drone circled, its current module outlined. |
| `OVERWATER` | Soil is above 90% moisture. | The module outlined and pulsing. |
| `OVERFEED` | Nutrients are above 80%. | The module outlined and pulsing. |
| `NOTHING TO DO` | Treating a module without pests, or reviving a healthy one. | The module outlined and pulsing. |
| `DEAD` | Anything but revive on a dead module. | The module outlined and pulsing. |
| `EMPTY` | The drone's water tank or fertilizer hopper is too low. | The drone circled. |
| `DOCK FULL` | Fleet above 10. The roof has 10 pads. | The extra pads drawn dashed beyond the roof. |
| `OUT OF RANGE` | A setting outside the controls' range, or more than 8 flags. | Message and log entry. |
| `UNREADABLE` | No JSON found, or an unknown drone, module, task or move type. | Message and log entry. |

## How the drones behave without proposals

1. Scan every module within sensing reach, plus any flagged module.
2. Skip modules another drone has claimed.
3. Score the rest. Decaying and dead modules need reviving, pests need treating, dry soil needs water, hungry soil needs fertilizer. Closer modules score higher.
4. Check the battery covers the trip, the job and the flight home. If not, return to the dock.
5. Fly along the joints of the wall: across on the slab lines between levels, up and down on the bay lines between modules.
6. Give way to working or docked drones and to lower-numbered drones. Hold position, step aside, or take another rail. Drones never overlap.
7. Water with fertilizer mixed in when a module is running low on nutrients.
8. If nothing is in view, fly home. After an hour on the dock, fly a scouting run.

## Publish on GitHub Pages

1. Create a public repository and add `index.html`, `play.html`, `.nojekyll` and `README.md` from this folder.
2. In the repository, open **Settings → Pages**, set *Source* to **Deploy from a branch**, and pick `main` and `/ (root)`.
3. After a minute the site is live at `https://<your-username>.github.io/<repository>/` and the playground at `…/play.html`. Put that link at the top of this README.

With git and the GitHub CLI, from this folder:

```bash
git init -b main
```

```bash
git add index.html play.html .nojekyll README.md
```

```bash
git commit -m "Living Facade playground"
```

```bash
gh repo create living-facade --public --source . --push
```

Then turn on Pages as in step 2, or run:

```bash
gh api -X POST repos/{owner}/living-facade/pages -f "source[branch]=main" -f "source[path]=/"
```

## Notes

- The page loads fonts from Google Fonts. It loads the Anthropic SDK (`@anthropic-ai/sdk@0.131.0`, from jsDelivr) only when you press Suggest with a key.
- Calling the Claude API straight from a browser exposes the key to that browser. That is fine for your own key on your own machine. Don't hard-code a key into the page.
- Some hosts block requests to outside APIs (the claude.ai preview does). There, use the chatbot route: the playground works the same way.
