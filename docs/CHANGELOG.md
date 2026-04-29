
# Changelog

All notable changes to SelectPaper are documented in this file.

---

## v16 — Output Stability & Page-Size Control

### Added
- Added real-time generation progress UI using `runLog`.
- Added step-based progress states for keyword matching, LaTeX generation, semantic matching, and layout rationale generation.

### Improved
- Removed unused LaTeX parameter from `generateRationale`.
- Reduced unnecessary token usage by avoiding transmission of unused LaTeX code.
- Improved user visibility during generation.

### Fixed
- Fixed A4 output issue caused by memoir's default stock size.
- Added explicit stock-size settings to preserve selected publication dimensions.
- Added immutable page-size and margin rules to the refine prompt.
- Prevented refine from solving overflow by changing page size or margins.

---

## v15 — LaTeX Output Cleanup

### Added
- Added system prompt constraints to LaTeX generation.
- Added system prompt constraints to refinement.

### Improved
- Strengthened LaTeX post-processing.
- Removed all text before `\begin{document}` when necessary.

### Fixed
- Fixed issue where AI-generated reasoning text such as “Hmm” or “Actually” appeared inside the output code.
- Prevented non-LaTeX text from being mixed into the generated result.

---

## v14 — Real-Time Progress Panel & Token Review

### Added
- Added `runLog` state.
- Added real-time output progress panel.
- Added progress display for:
  - keyword matching
  - LaTeX generation
  - semantic matching
  - layout rationale generation

### Improved
- Replaced simple loading UI with a more informative progress panel.
- Removed unused LaTeX argument from `generateRationale`.

### Fixed
- Reduced repeated token waste caused by passing full LaTeX code to a function that did not use it.

---

## v13 — AI-Based Variable Layout Redesign

### Changed
- Removed manual section builder for variable layouts.
- Reworked variable layout behavior so AI analyzes the full body text and determines layout variation automatically.
- Changed variable layout from user-defined section control to content-analysis-based editorial judgment.

### Improved
- Added clearer variable layout instructions:
  - headlines and openers may use full-width one-column layout
  - body paragraphs follow the publication's base column structure
  - quotes or emphasis sections may use alternate column widths
  - column transitions should happen by paragraph, not mid-sentence

### Removed
- Removed `sections` from `styleConfig`.
- Removed section label constants.
- Removed section builder UI.

---

## v12 — Style Tab & Column Mode Controls

### Added
- Added a style instruction tab.
- Added controls for:
  - automatic layout
  - 1-column
  - 2-column
  - 3-column
  - 4-column
  - variable layout
- Added extra style directive input field.

### Changed
- Reworked Step 0 UI into tabbed interface.
- Treated “열” notation as module-grid structure rather than literal body columns.
- Updated column logic so high column counts are interpreted as flexible layout grids.

### Improved
- Improved support for layouts such as 12-column grids, where the body should use merged column blocks instead of literal 12-column text.

---

## v11 — Long Text Input Expansion

### Improved
- Removed body text truncation during LaTeX generation.
- Increased LaTeX output token limit.
- Increased timeout for longer generation.
- Expanded body textarea height.
- Added approximate page count display.

### Fixed
- Fixed issue where only a small portion of long body text was being sent to the model.

---

## v10 — Footnote State Initialization Fix

### Fixed
- Fixed `Cannot access 'hasFootnote' before initialization` error.
- Moved footnote-related variable declarations before dependent logic.
- Removed duplicated declarations after refactoring.

---

## v9 — Line Spacing System Fix

### Changed
- Removed `\linespread`-based line spacing.
- Replaced line spacing logic with direct `\fontsize{size}{leading}\selectfont`.

### Fixed
- Fixed excessive line spacing caused by incorrect `\linespread` calculation.
- Ensured DB-defined font size and leading values are reflected directly in LaTeX output.

---

## v8 — Footnote Automation & Endnote Removal

### Added
- Added automatic footnote marker parsing.
- Supported footnote markers such as:
  - superscript numbers
  - `[1]`
  - `^1`

### Changed
- Removed endnote input field.
- Reworked body generation to inject footnotes directly from detected markers.

### Fixed
- Removed invalid `\footnotemargin` usage that caused unit text such as `4mm` to appear in the output.

---

## v7 — LaTeX Preamble Separation

### Changed
- Rebuilt LaTeX generation pipeline.
- Moved preamble generation from AI output into fixed JavaScript-generated code.
- AI now generates only the document body between `\begin{document}` and `\end{document}`.

### Fixed
- Prevented LaTeX setup commands from leaking into visible body text.
- Reduced risk of units such as `mm` appearing as printed text.

---

## v6 — Unit Artifact Removal

### Fixed
- Removed visible `1em` artifacts from generated pages.
- Replaced problematic `em`-based footnote margin setting with safer values.
- Added explicit rules preventing unit strings from being output as visible text.

---

## v5 — Divider Line Removal

### Changed
- Removed footnote divider rules.
- Removed header and footer rules.
- Added explicit restrictions against inserting divider lines.

### Fixed
- Prevented unwanted horizontal rules from appearing in the layout.

---

## v4 — Footnote Layout Improvement

### Added
- Added footnote configuration block based on DB data.
- Added column-aware footnote guidance.

### Improved
- Improved behavior of footnotes in multi-column layouts.
- Adjusted footnote size and leading based on DB values.

### Fixed
- Removed default `\footnoterule`.
- Prevented full-width footnote divider lines.

---

## v3 — Module Grid & Column Logic Redesign

### Changed
- Reworked column parsing logic.
- Distinguished between simple multi-column layouts and module-grid layouts.
- Used `layout_type` to understand actual editorial usage.
- Added support for body and note area width calculations.

### Fixed
- Fixed issue where high column counts such as 10 or 11 columns were interpreted literally as `multicols{10}`.
- Prevented unrealistic body text layouts based on raw column numbers.

---

## v2 — Structured Input UI

### Added
- Added structured input fields:
  - title
  - subtitle
  - body
  - running head
  - footnote
  - endnote
- Added genre selector.

### Changed
- Renamed “genre hint” to “genre”.
- Reworked matching to use body text primarily, with title and subtitle fallback.
- Reworked LaTeX prompt input into structured content blocks.

### Removed
- Removed sample text selector.
- Removed large sample text constants from the interface.

---

## v1 — Initial Optimization

### Improved
- Removed unused and duplicated code.
- Reduced token usage in repeated API calls.
- Added rationale caching.
- Compressed refine input by removing comments and limiting history.
- Reduced file size from the earlier prototype.

### Removed
- Removed unused `profileText()` function.
- Removed unused `generate()` function.
- Removed unused `textProfile` state.
- Removed syntax noise from state initialization.

---

## Initial Prototype — SelectPaper_3-2

### Added
- Initial AI-based editorial design system.
- Text profiling flow.
- Publication DB matching.
- LaTeX generation pipeline.
- Page preview component.
- Sample genre texts.
- Style principle data.
- Publication reference database.

### Known Issues
- High token usage during generation and refinement.
- Some unused functions and duplicated logic.
- LaTeX generation was not yet stable enough for long-form use.
