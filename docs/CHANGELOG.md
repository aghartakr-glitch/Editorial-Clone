
# Changelog

All notable changes to SelectPaper are documented in this file.

## v22 — Korean Typesetting Engine & Style Composition

### Added

- Added Korean-aware line breaking system using `HANGUL_LINEBREAK_SKIP`.
- Added `chooseAlignmentPolicy()` for automatic justify/ragged fallback.
- Added `cleanKoreanHyphenation()` post-processing to remove invalid Hangul hyphenation.
- Added `composeStyleSpec()` for attribute-based style composition.
- Extended `semanticRerank()` to return top 4 candidates instead of a single result.
- Added data audit utility to inspect DB source, fields, and Sulki & Min coverage.

### Changed

- Replaced single-reference cloning with multi-reference style composition:
  - page / margin / column / typography / folio now selected independently.
- Reworked LaTeX prompt to enforce Korean typesetting rules:
  - no hyphenation
  - conservative linebreak glue
  - ragged fallback for narrow columns
- Updated preview rendering:
  - `textAlign` now follows alignment policy
  - `wordBreak: keep-all`, `hyphens: none`

### Improved

- Prevented excessive spacing in justified Korean text.
- Eliminated use of `\sloppy` and `\tolerance=9999`.
- Improved line-level orphan control (not just page-level).
- Reduced prompt size and removed duplicated rules.

### Fixed

- Fixed issue where Hangul justification caused uneven spacing.
- Fixed unwanted hyphen insertion inside Korean words.
- Fixed cases where one or two syllables were isolated at line edges.

### Notes

- DB source is currently hardcoded (not directly connected to Google Sheet).
- Approximately 254 entries available, including significant Sulki & Min-related data.
- Token usage reduced by ~350 tokens per generation cycle.

---

## v21 — Layout Stability & Token Optimization

### Fixed

- Fixed header clipping and page number overflow issues.
- Ensured header area is included in geometry calculations using `includehead=true`.
- Explicitly defined `\headheight` to prevent overflow.
- Fixed inconsistent spacing between header and body.

### Improved

- Improved Korean line-breaking quality by removing `\tolerance=9999`.
- Applied strict widow/orphan penalties:
  - `\widowpenalty`, `\clubpenalty`, `\displaywidowpenalty`, `\brokenpenalty`
- Reduced LaTeX prompt verbosity and removed redundant sections.

### Changed

- Replaced loose line-breaking settings with:
  - `\pretolerance=100`
  - `\tolerance=400`
  - `\emergencystretch=3em` (fallback only)

### Performance

- Reduced token usage by approximately 300 tokens per request.

---

## v20 — Pipeline Stabilization & Data Integration

### Added

- Added multi-field scoring system based on:
  - summary
  - layout features
  - typography rationale
  - margin intention
- Integrated design-intent fields from database into recommendation pipeline.

### Changed

- Replaced simple keyword matching with enriched scoring model.
- Improved semantic reranking with structured reasoning.
- Updated pipeline:
  - analyzeText → keyword top20 → semantic rerank → LaTeX generation

### Improved

- Improved alignment between recommended design and generated layout.
- Increased relevance of selected references based on actual editorial intent.

---

## v19 — Semantic Matching Pipeline

### Added

- Added `analyzeText()` to extract structured text profile.
- Added `semanticRerank()` to refine candidate selection.
- Added structured rationale output for recommendations.

### Changed

- Moved semantic matching from background to main pipeline.
- Updated scoring logic:
  - reduced genre bias
  - increased weight on summary, layout, and typography fields

### Improved

- Improved recommendation accuracy beyond keyword matching.
- Enabled explainable design selection.

---

## v18 — Typography Base Integration

### Added

- Added TYPO_BASE system for core typography rules.
- Added font-size-based leading calculation.
- Added page-size-based running head sizing.
- Added column combination logic (e.g. 5 → 3+2 / 4+1).
- Added rules for text boxes and prohibition of divider lines.

### Changed

- Moved typography decision logic from AI to system-level constants.
- Reduced dependency on prompt-based typography reasoning.

### Improved

- Prevented basic typography errors (leading, spacing, layout misuse).
- Maintained output quality without increasing token usage.

### Notes

- TYPO_BASE acts as a guardrail, not a full design generator.
- Only essential rules are passed to the model to minimize token cost.

---

## v17 — Margin Control Fix & Layout Adjustment

### Fixed

- Fixed issue where top margin could not be adjusted due to immutable constraints in refine prompt.
- Removed margin values from IMMUTABLE rules while keeping page size fixed.
- Prevented AI from ignoring user requests related to spacing adjustments.

### Improved

- Enabled direct control of top margin via `geometry` settings.
- Improved separation between:
  - page margin (`top`)
  - header spacing (`includehead`, `headsep`)
  - body offset (`\vspace*`)
- Improved layout predictability when adjusting vertical spacing.

### Added

- Support for manual top margin adjustment (e.g. `top=67mm → top=18mm`).
- Support for body start offset control using `\vspace*{}`.

### Notes

- Previous system locked both page size and margins, which caused layout requests (e.g. "reduce top space") to fail.
- Layout control is now divided into:
  - **Page size (immutable)**
  - **Margins (adjustable)**
  - **Content spacing (adjustable)**

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
