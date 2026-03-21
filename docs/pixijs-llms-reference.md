# PixiJS LLM Reference for Agents

This repository uses this document as a short agent-facing bridge to the official PixiJS reference for language models.

- Official source of truth: https://pixijs.com/llms-full.txt

Use the official PixiJS LLM reference first when implementing, reviewing, or debugging PixiJS code. This file stays intentionally concise so agents can find the right v8 rules quickly without copying a huge third-party document into the repo.

## Default Assumption

- Assume **PixiJS v8** unless the target project clearly pins a different version.
- Do **not** guess legacy v6/v7 APIs when the codebase or docs do not show them.
- If a project already uses older APIs, follow the project instead of forcing a migration.

## Practical Rules for Agents

1. **Initialize `Application` asynchronously.**
   - Prefer:
     ```ts
     const app = new Application();
     await app.init({ resizeTo: window });
     document.body.appendChild(app.canvas);
     ```
   - Do not assume old constructor-time initialization is correct for v8.

2. **Use `app.canvas`, not legacy `app.view`, unless the project already does otherwise.**
   - The official v8 examples append `app.canvas` to the DOM.

3. **Treat `app.stage` plus `Container` composition as the default scene structure.**
   - Keep world, HUD, overlays, and presentation concerns separated.
   - Prefer clear container ownership over dumping everything into one root.

4. **Load assets with the v8 asset system.**
   - Prefer `await Assets.load(...)` and `Assets.get(...)`.
   - Avoid introducing the legacy Loader API unless the existing project already uses it.

5. **Drive frame updates through the ticker with delta-aware logic.**
   - Prefer `app.ticker.add((ticker) => { ... ticker.deltaTime ... })` or equivalent frame-independent math.

6. **Do not invent outdated interaction APIs.**
   - Follow current pointer/event patterns from the official docs and the target codebase.
   - For touch UI, use explicit hit areas and predictable interaction zones.

7. **Use advanced rendering features deliberately.**
   - `RenderGroup` and `RenderLayer` can help with performance and draw ordering, but they are not the default answer for every scene.
   - Profile before overusing them.

8. **Clean up GPU-backed resources in long-lived or dynamic flows.**
   - Call `destroy()` on objects you no longer need.
   - Use texture unloading strategically when asset churn is high.

9. **Keep browser/build caveats in mind.**
   - The official docs note a Vite production caveat around top-level await; wrapping Pixi bootstrap in an async function remains the safe default.

10. **Check ecosystem docs instead of guessing adjacent tooling.**
    - Pixi React requires React 19+.
    - Layout, Filters, Sound, UI, AssetPack, and DevTools have separate docs in the official ecosystem.

## Suggested Agent Workflow

When a task involves PixiJS implementation or review:

1. Read this file.
2. Open the official reference at https://pixijs.com/llms-full.txt.
3. Match the target project's installed PixiJS version and current patterns.
4. Prefer small, version-correct changes over speculative framework rewrites.

## What Not to Do

- Do not paste large chunks of third-party PixiJS documentation into commits.
- Do not confidently recommend `app.view`, legacy Loader examples, or older initialization patterns unless the project already uses them.
- Do not mix DPI/sharpness concerns with logical layout rules.
- Do not assume every performance issue requires filters, render groups, or custom shaders.
