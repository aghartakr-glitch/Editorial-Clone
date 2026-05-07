
# Changelog

All notable changes to SelectPaper are documented in this file.

## v27 — Genre Hint Priority & Mixed Typeface Stabilization

### Added

- Added genre-first filtering when the user explicitly selects a genre.
- Added `hint` parameter to `semanticRerank()`.
- Added separate behavior for:
  - automatic genre detection
  - user-selected genre mode
- Added publication-type-aware genre pool filtering.
- Added font-family switching directly inside heading commands for mixed typeface layouts.
- Added `\bodyf` command to return from heading font to body font.

### Changed

- Changed genre hint behavior from score bonus to candidate pool filtering.
- Updated `run()` pipeline so selected genre is applied before content scoring.
- Updated `semanticRerank()` so it knows whether candidates were selected from a user-chosen genre.
- Removed genre hint scoring responsibility from `scoreKw()`.
- Updated heading typography commands so mixed serif/sans-serif layouts do not rely on AI inserting `\sffamily` manually.

### Improved

- Improved consistency between user-selected genre and generated style.
- Improved distinction between automatic genre detection and manual genre selection.
- Improved literature-category behavior by preventing content matching from overriding the selected genre too easily.
- Improved mixed typeface layouts:
  - serif body
  - sans-serif headings
- Improved heading command reliability in LaTeX output.

### Fixed

- Fixed issue where user-selected genre could be overridden by content-based semantic matching.
- Fixed issue where genre bonus was too weak compared to content matching scores.
- Fixed issue where `semanticRerank()` did not know the selected genre.
- Fixed issue where mixed serif/sans-serif layouts required the AI to manually insert font switching commands.
- Fixed potential inconsistency between heading font and body font in mixed typeface layouts.

### Notes

- When no genre is selected, the system still uses content-based automatic genre detection.
- When a genre is selected, the system prioritizes that genre first, then selects the best reference within that genre pool.
- v27 clarifies the difference between “automatic recommendation” and “user-directed style selection.”

---

## v26 — Context-Based Genre Detection & Layout Safety Fix

### Added

- Added context-based genre detection signals to `analyzeText()`.
- Added `structure` and `pub` fields to better distinguish narrative, essay, interview, list-based, and exhibition-style texts.
- Added genre-to-publication-type preference mapping.
- Added negative prompt constraints to prevent single-keyword genre misclassification.
- Added layout safety check for body/note split layouts.

### Changed

- Changed genre matching from single keyword influence to multi-signal judgment.
- Updated semantic rerank to consider:
  - writing style
  - text structure
  - publication type
  - genre confidence
  - layout suitability
- Adjusted genre hint behavior so selected genres prefer matching publication types.
- Updated body/note split handling so note columns are only used when actual note content exists.
- Changed unsafe side-note layouts to downgrade into regular multi-column or single-column layouts when no note content is provided.

### Improved

- Improved consistency between selected genre and generated layout.
- Reduced cases where “문학” selection produces magazine-like layouts.
- Reduced misclassification caused by isolated words such as “전시”.
- Improved layout safety for texts without side notes.
- Improved editorial appropriateness by using neutral layouts when genre confidence is low.

### Fixed

- Fixed issue where a single keyword could dominate genre selection.
- Fixed issue where literature-like texts could be matched with magazine/journal-style layouts.
- Fixed issue where body text appeared only on half of the page due to unnecessary body/note split.
- Fixed issue where normal `\footnote{}` content was moved into side columns.
- Fixed issue where `gray!10` or other LaTeX color strings appeared in the visible body text.
- Removed unsafe use of `\colorbox`, `\fbox`, `\color`, `\textcolor`, and `xcolor`.

### Notes

- v26 focuses on preventing semantic matching failure caused by shallow keyword matching.
- The goal is not to make genre detection more aggressive, but to make it more cautious and context-aware.
- If genre confidence is low, the system should prefer a neutral editorial layout over a strongly genre-specific style.

 ---

## v25 — Heading Typography System & Prompt Compression

### Added

- Added heading typography system based on `TYPO_BASE.headingSizes()`.
- Added generated LaTeX heading commands:
  - `\hone` for main titles
  - `\htwo` for subtitles / chapter headings
  - `\hthree` for section headings
- Added `TYPO_BASE.leadingTable()` to guide heading leading values.
- Added mandatory heading typography instructions to the LaTeX prompt.

### Changed

- Moved heading size and leading decisions from AI guesswork to JavaScript-calculated values.
- Updated LaTeX prompt so larger text elements cannot reuse body leading.
- Reduced semantic rerank candidates from 12 to 8 to lower token usage.
- Shortened candidate summaries and design-feature snippets sent to semantic rerank.
- Compressed `RULES` and `preambleSummary` sections.

### Improved

- Improved title, subtitle, and section heading spacing.
- Reduced cases where headings appeared too tight or visually compressed.
- Improved hierarchy consistency between body text and headings.
- Reduced unnecessary prompt tokens while keeping key typography constraints.

### Performance

- Estimated net token reduction: approximately 260 tokens per generation cycle.
- Reduced semantic rerank input size without removing core matching fields.

### Notes

- v25 focuses on typographic hierarchy stability, not visual style expansion.
- The main design goal is to prevent basic heading/leading errors while preserving the existing SelectPaper pipeline.

---

## v24 — Running Head Fix & Multi-Column Body Size Correction

### Fixed

- Fixed issue where DB `running` values such as `8pt`, `10pt`, or `10.5pt` were passed as visible running head text.
- Updated LaTeX generation to use the user-provided `fields.면주` value instead of DB `p.running`.
- Updated refine prompt so running head text also comes from `fields.면주`.
- Prevented unwanted metadata-like text from appearing in the running head area.

### Added

- Added automatic body-size correction for multi-column layouts.
- Added adjusted body-size state for UI display.
- Added visual indication when body size has been reduced due to column count.

### Changed

- Body size is now adjusted based on effective column count:
  - 2–3 columns: reduce body size by 0.5pt
  - 4+ columns: reduce body size by 1.0pt
  - minimum body size: 7pt
- Leading is recalculated after body-size correction.
- Preamble, preamble summary, and LaTeX prompt now use the corrected body size and leading.

### Improved

- Improved readability in narrow multi-column layouts.
- Reduced cases where only 8–9 Hangul characters fit per line before wrapping.
- Made dense layouts more appropriate for long-form Korean body text.
- Improved UI transparency by showing when body-size correction has been applied.

### Notes

- v24 addresses a practical typesetting bug: DB metadata should not be treated as user-visible text.
- The version also begins correcting layout behavior based on column density.

---

## v23 — Handoff Baseline & Pipeline Audit

### Added

- Added handoff baseline for continuing development from the current SelectPaper architecture.
- Documented current pipeline state:
  - `analyzeText`
  - `scoreKw`
  - `semanticRerank`
  - `applyTextCorrections`
  - LaTeX generation
  - `generateRationale`
- Documented remaining technical tasks for the next versions.

### Confirmed

- Confirmed that the system uses a hardcoded editorial design DB rather than a live Google Sheet connection.
- Confirmed that the recommendation pipeline uses both keyword scoring and semantic reranking.
- Confirmed that the LaTeX generation flow still depends on a fixed preamble + body-only generation structure.

### Known Issues

- Running head metadata could still be passed as visible running head text.
- Semantic rerank and LaTeX generation could still drift if the selected reference changes after generation.
- Font file mapping still required more precise handling.
- TYPO_BASE values still needed further updates for heading and hierarchy control.
- Refine flow still needed a stronger preamble reinsertion strategy.

### Notes

- v23 functions as the checkpoint version before targeted typography and layout fixes.
- The version clarified what should be fixed in v24 and v25 rather than introducing a major new system layer.

---

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
