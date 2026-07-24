# Universal Git Visualiser — QA Report & Verification Registry

## Round 2 — Fix Verification Pass (commit 718ade9)
* **Site tested:** `https://nimitwalia.github.io/universal-git-visualiser/` (GitHub Pages, deployed from staging branch)
* **Repos used:** `forcedotcom/sf-skills` (primary), `docker/getting-started-app` (Round 1 cross-check)
* **Date:** July 23, 2026
* **Result:** 13 of 14 items PASS, 0 PARTIAL, 1 FAIL.

### Round 2 Fix Verification Registry

| Item | Round 1 Finding | Round 2 Result | Status |
| :--- | :--- | :--- | :---: |
| **2.1 Syntax highlighting** | Not implemented at all — only line numbers shipped, plain white text with no coloring. | **FIXED.** Self-hosted regex tokenizer confirmed live: JSON/YAML keys, string values, and booleans render in distinct colors; keywords/strings/comments are colored on general files. No CDN dependency. | **PASS** |
| **2.3 Deep-link line highlight** | Highlight CSS class defined and function existed, but zero elements ever carried the class after render. | **STILL BROKEN.** New code wraps the highlight in `requestAnimationFrame` + `setTimeout(100)`. Still zero elements get the class. The `requestAnimationFrame` callback does not reliably fire in the file-load flow. | **FAIL** |
| **3.1 Node scripts** | Picked 'start' if present, otherwise the first key in the scripts object — silently dropped every other script. | **FIXED.** Now lists every defined script individually with its own Copy button. | **PASS** |
| **3.2 Python ecosystem** | Only "pip install -r requirements.txt" shown, no virtual environment creation step. | **FIXED.** Full command reads `python -m venv venv && source venv/bin/activate && pip install -r requirements.txt`. | **PASS** |
| **4.1 Security lint** | Only postinstall/preinstall scripts and curl-pipe-to-shell were checked; unpinned versions and non-standard registries were not. | **FIXED.** Both missing heuristics (unpinned versions, unencrypted registry URLs) are now present alongside the original two under warning box. | **PASS** |
| **4.2 Scoped search** | UI was present and correctly positioned, but the actual cross-file jump had not been independently tested. | **FIXED.** Switched to the correct file in tree and navigated search match successfully. | **PASS** |

---

## Round 3 — Version Comparison Verification Pass (Phase 5)
* **Site tested:** Local HTTP Server (`http://localhost:8000`) / GitHub Pages (staging branch)
* **Date:** July 24, 2026
* **Result:** All newly added Phase 5 features fully verified and functional.

### End-to-End Test Cases (Phase 5: Version Comparison & Diff Viewer)

#### Test Case 5.1: Compare Button & Interface Activation
* **Objective:** Verify that entering Compare Mode dynamically scales the workspace layout.
* **Pre-conditions:**
  1. Open a valid repository (e.g. `nimitwalia/universal-git-visualiser`).
  2. Select any text or code file (e.g. `index.html`).
* **Test Steps:**
  1. Locate and click the **Compare** button (next to the "Copy Raw" button in the preview pane header).
  2. Observe the right-hand action cards column (`#widgetWorkspaceStack`).
  3. Observe the grid span of the code preview container.
  4. Verify that the floating compare settings bar is rendered at the top of the pane.
* **Expected Result:**
  * The right actions sidebar collapses and becomes hidden.
  * The main preview canvas container expands from its normal desktop width (`xl:col-span-8`) to full-screen width (`xl:col-span-12`).
  * The compare settings bar appears with the commits list dropdown, View toggle dropdown, Next/Prev buttons, and match count indicator.

#### Test Case 5.2: Side-by-Side (Split) View Line Alignment
* **Objective:** Verify that removed and added lines align side-by-side sequentially rather than stacking offset.
* **Pre-conditions:**
  1. Enter Compare Mode for a file with contiguous modifications.
  2. Ensure "Side-by-Side (Split)" layout is selected in the View options dropdown.
* **Test Steps:**
  1. Locate a block of contiguous modifications containing both removals and additions.
  2. Verify how the deleted lines (left pane) align with the added lines (right pane).
* **Expected Result:**
  * Contiguous blocks of edits are paired row-by-row sequentially (e.g., Left Row 1 has Removal 1 paired with Right Row 1's Addition 1).
  * If removals outnumber additions (or vice versa), the remaining slots display blank space templates on the opposite side, keeping matching code lines aligned.

#### Test Case 5.3: Inline Code Word Wrapping & Text Contrast
* **Objective:** Verify that long code segments wrap cleanly inline and maintain high contrast against dark theme backgrounds.
* **Pre-conditions:**
  1. Enter Compare Mode on a file with long code lines.
* **Test Steps:**
  1. Observe lines of code that exceed the horizontal width of their column pane.
  2. Verify that there are no horizontal scrollbars within the split columns.
  3. Check the legibility of unchanged code lines and removed code lines.
* **Expected Result:**
  * Long lines wrap inline using `whitespace-pre-wrap break-all`.
  * Unchanged code lines are rendered in bright light gray/white (`text-slate-100`).
  * Removed lines are rendered in legible light yellow text (`text-yellow-100`) on a soft yellow background highlight (`bg-yellow-500/15` and `border-yellow-500`).

#### Test Case 5.4: Next/Prev Floating Navigation & Page-Level Scrolling
* **Objective:** Verify that clicking Next/Prev navigation buttons scrolls only the main page viewport without moving the floating settings bar.
* **Pre-conditions:**
  1. Enter Compare Mode on a file containing multiple separate modification blocks.
  2. Scroll the page vertically so that the main app header is out of view.
* **Test Steps:**
  1. Click **Next** or **Prev** multiple times.
  2. Observe the compare settings bar containing the navigation buttons.
  3. Verify the vertical position of the highlighted code block.
* **Expected Result:**
  * The settings bar remains pinned and sticky at exactly `top-[60px]` (directly below the sticky global app header), remaining stationary under the mouse cursor.
  * The main page viewport scrolls smoothly to align the target change block directly below the sticky header bar (target line is perfectly visible, accounting for header offsets).

#### Test Case 5.5: Navigation Boundaries
* **Objective:** Verify that the Next and Prev navigation buttons disable when boundaries are reached.
* **Pre-conditions:**
  1. Enter Compare Mode on a file with a known number of change blocks.
* **Test Steps:**
  1. Click **Next** repeatedly until the final change block is reached and highlighted.
  2. Click **Prev** repeatedly until the first change block is reached and highlighted.
  3. Verify the button disabled states and the text indicator format.
* **Expected Result:**
  * When on the first change block, the **Prev** button is disabled (`disabled=true`) and styled gray.
  * When on the last change block, the **Next** button is disabled (`disabled=true`) and styled gray.
  * The text indicator displays progress accurately (e.g. `1 of 5 changes`, `5 of 5 changes`).

#### Test Case 5.6: Exit Compare & Layout Reset
* **Objective:** Verify that exiting Compare Mode correctly restores the multi-column workspace layout.
* **Pre-conditions:**
  1. Enter Compare Mode.
* **Test Steps:**
  1. Click the **Exit Compare** button in the header.
  2. Or click another file in the explorer tree.
* **Expected Result:**
  * Compare settings bar is removed.
  * The right actions sidebar (`#widgetWorkspaceStack`) returns and becomes visible.
  * The main preview canvas container reverts to its normal width (`xl:col-span-8`).
  * The original code view or rendered Markdown document is restored.
