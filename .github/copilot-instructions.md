# Copilot Instructions for LearningHub-Evaluaciones

- Scope: static iframe-embeddable questionnaire builder + answering flow. No bundler; use CDN Vue (v3), Tailwind CDN, Lucide icons, Google Font Inter. Keep ASCII.
- Entry point: single `index.html` with Vue app. Route-like mode chosen from iframe URL param (`mode=create|edit|answer`). Parent owns persistence; we postMessage to exchange data.
- Question types to support (match UI): Opción múltiple, Casillas de verificación, Verdadero/Falso, Emparejamiento, Ordenar elementos, Completar espacios.
- Creation/Edit flow: build form UI that matches provided mockups. Allow adding/removing/reordering questions and nested options/pairs/steps/words. Persist in reactive state; debounce local preview if needed. Provide a "Save" button that serializes schema and sends it to parent via `postMessage`.
- Answering flow: load schema from parent (postMessage), present one question per screen with progress bar, Prev/Next, attempts and minimum score constraints. At completion, compute score (%) and per-question correctness, then send results via `postMessage`. Show summary view similar to mockup.
- postMessage contract (window.parent): use `targetOrigin='*'` unless instructed. Outgoing events: `{type:'form-ready'}`, `{type:'schema-request'}`, `{type:'schema-update', payload:schema}`, `{type:'result', payload:{answers, scorePercent, correctCount, total, passed}}`. Incoming events: `{type:'schema', payload:schema}` to load for edit/answer; `{type:'config', payload:{attempts,minPercent}}`; `{type:'mode', payload:'create'|'edit'|'answer'}`.
- Schema shape (JSON): `{title?, description?, attempts, minPercent, questions:[{id, type, prompt, options?, pairs?, order?, blanks?, correctAnswers, feedback?}]}`. Keep stable `id` per question/option for reactivity. For blanks: `blanks:[{id, placeholder, answers:["word1"]}]` and store sentence with `[blank]` markers.
- UI conventions: Tailwind utility classes; Lucide icons for add/delete/reorder; soft rounded cards per mockup; Spanish labels; keep form controls keyboard-friendly; use progress bar based on answered/total.
- Validation: require prompt text; enforce correct answer selection per type (e.g., at least one correct option; both sides for pairs; order list non-empty; blanks must map to bank). Guard against empty schema before sending to parent.
- Data flow: Creation/Edit only mutates local state; emits `schema-update` on save. Answering never mutates schema; keeps `answers` map `{questionId: value}` where value varies by type (single string, array, pair array, ordered array, blanks map). Result payload includes raw `answers` and computed correctness per question.
- Testing/dev workflow: simple static hosting (e.g., `python3 -m http.server` or VS Code Live Server). No npm install expected.
- File organization: keep everything in `index.html` (Vue app, templates, minimal CSS overrides). If splitting later, prefer `/app/` folder with plain ES modules; avoid build steps unless necessary.
- Performance: small DOM; avoid heavy watchers; use computed properties for derived progress/score.
- Accessibility: use native inputs where possible; include `aria-label` on controls; ensure focus moves on next/prev question.
- When unsure about parent contract (origins, event names), ask for clarification before implementation.
