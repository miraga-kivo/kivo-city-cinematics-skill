# Validation — 2026-11-21

## Local verification

Clean `npm ci` succeeded using the included package lock. Node 22.23.3, Sharp 0.35.5, FFmpeg 4.4 and FFprobe 4.4.1 were used on macOS ARM64. Doctor checks passed, including libx264 availability. Skill Creator validation passed.

All **six automated tests passed**, including real landscape/portrait encoding, HUD overlay, exact timing, contact-sheet generation, rejection of too-short sources, refusal to overwrite existing video, brief validation, XML-safe names, and Chinese prompt/graphic rendering. English landscape and Chinese portrait maps were visually checked. Synthetic test fixtures are used only for local encoder tests, not presented as model-generated media.

## Live Kivo end-to-end run

**Passed:** the `character-intro` recipe, 16:9, using the user's explicitly authorised MCP account and a 500-credit cap. Models, limits and quotes were read live; quote/model version was `2026-11-20-seedance-pricing-v4`, billing version `2026-11-08-starter-120`.

| Step | Delivered | Actual credits |
| --- | --- | ---: |
| Local concept map, S3 upload and inspection | One ready original map asset | 0 |
| Seedream 5.5 reference images | City and consistent character, 1280×720 | 10 |
| Seedance 2.5 Draft batch | Two five-second candidates, 854×480 | 90 |
| Selected Draft → native final | One upstream 1920×1080 final | 360 |
| Local graphics, editing and review | Twelve-second 1920×1080 cut | 0 |
| **Total** | **Five paid jobs succeeded** | **460** |

The account balance delta also matched 460 credits. No paid retries were required. Prices here describe this run, not future fixed prices.

The image references retained the blue-hour waterfront architecture, teal jacket, grey cargo trousers, boots and orange bag. Six time-spaced frames were inspected from each candidate and the selected native final. Candidate 1 was selected because its full character remained visible from the opening; candidate 2 began with the head partly cropped before the camera pulled back. The final preserved candidate 1's sampled composition and appearance. No obvious wardrobe changes or limb distortions were visible in the reviewed frames; frame sampling cannot guarantee the absence of transient artifacts between samples.

The upstream videos contain 5.042 seconds at 24fps; the local edit uses exactly five seconds of the selected native final. The finished cut is **12 seconds, 1920×1080, 24fps, silent**: map 2.5s, city still 3s, generated character shot 5s, endcard 1.5s. Its mini-map was positioned at bottom right to avoid the character. This is an edited cinematic concept, not twelve seconds of generated gameplay. A 720p distribution preview and poster are under `examples/demo/`.

The full-resolution cut, native selected final, editable graphics/brief/manifest and review sheets are delivered separately. The project, named video prompt, both candidates and final remain available in Kivo Studio to the connected project owner.

## Corrections made from the run

- Codex automatic metadata registration produced `invalid_redirect_uri`. Reauthorising with dynamic client registration (DCR) and explicit generation scope succeeded; the README includes the command. The already-running chat client retained an old grant, so live generation used a fresh authenticated MCP session with the newly consented scopes. No permissions were bypassed or server checks weakened.
- Confirmed that `stage:asset` images already appear in the library. Calling addToLibrary for them returned `NOT_GENERATION_RESULT`; the workflow now distinguishes them from Draft/final results.
- Added configurable HUD corners after checking character placement; changed the default to bottom right. Added map-boundary clipping for river graphics and checked the resulting portrait layout.
- Added the reusable contact-sheet/metadata script and tested it with real generated videos. Confirmed native final dimensions separately from the local edit.

## Scope and limitations

The 15-second `city-trailer` plan, Chinese labels and portrait local rendering were checked locally. A second paid city flyover and a portrait model-generation run were **not** submitted within this test budget. Geography is an illustrative concept, not a real street map. Character and layout consistency remain model-dependent. Maps and the HUD are original graphics, not a copy of an existing game interface. Source clip audio is intentionally disabled for this silent proof; requested sound needs a separate budget and an explicit assembly audio track.

No website/backend changes, deployments, GitHub publication or blog publication were performed. This distributable includes no account credentials, signed URLs or personal account identifiers.
