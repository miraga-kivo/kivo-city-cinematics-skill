# Kivo MCP operations

Connect a remote Streamable HTTP MCP server at `https://kivst.com/mcp`. The agent must actually support remote OAuth MCP; pasting the URL alone is not sufficient in every client. Use browser login (email or Google) and consent. Never put the website password, Firebase token or Lambda secrets into this package. Call the connected tools rather than inventing a direct bearer token.

Tool names below are dotted logical names; clients can expose them as `kivo_jobs_create` or prefixed equivalents. Discover their live schemas. Current required scopes: `kivo:read`, `kivo:write`, `kivo:generate`; `kivo:billing` is useful for the caller's ledger. OAuth refresh support varies by client.

## Inspect and select

- `account.get {}`: current MCP identity and personal balance.
- `catalog.get {}`: live model IDs, types, Draft/final resolutions, limits and billing/model versions.
- `projects.get {projectId}`: creator pays, including shared projects; this is the relevant generation balance.
- `projects.create {name, description}`: name ≤50 characters, description ≤500. Keep resulting `id`.

Prefer the current enabled image default for reference plates. Select a video model with native Draft support when finalising from a selected candidate. Check supported ratio, format, duration, image references and maxCount. As of the validation run Seedance's Draft was 480p and finals supported 1080p/4K; these are observations, not hardcoded requirements. Budget the whole proposed workflow from live quotes.

## Upload a locally rendered map

1. Stat the file. `assets.prepare {projectId,name,contentType:"image/png",byteSize:ACTUAL_BYTES,category:"scene",rightsConfirmed:true,noRealPeople:true}`. Only confirm rights for owned/authorised sources. An original local concept map contains no real people.
2. Save the response `{id,key,contentType,byteSize,upload:{url,fields}}` in a private temporary JSON file (mode 0600), not in Git. Upload helpers accept this response directly, not the outer MCP text envelope.
3. Run `node scripts/upload-s3.mjs --prepared PRIVATE.json --file MAP.png`. The helper sends all form fields and real bytes to the signed S3 POST endpoint. It does not call complete or generate anything. Do not send base64 through MCP.
4. `assets.complete {assetId:ID}`, then `assets.list {projectId,scope:"all"}` until that asset is `ready`. Failed inspection must be resolved before use. Remove private signed upload details once done.

Name collisions or interrupted prepare calls can leave pending assets. Reconcile with assets.list before preparing another. Do not delete unrelated user assets.

## Quote and create reference images

Example logical arguments (replace symbolic values):

```json
{
  "projectId":"PROJECT_ID",
  "modelId":"LIVE_IMAGE_MODEL_ID",
  "stage":"asset",
  "name":"City environment",
  "promptText":"Original city environment ...",
  "referenceIds":["READY_MAP_ASSET_ID"],
  "parameters":{"resolution":"1K","ratio":"16:9","format":"png","count":1}
}
```

`jobs.quote` returns `totalCredits`, `billingVersion`, `modelVersion` and normalised parameters. Show price and obtain agreement. For `jobs.create`, pass the same creative inputs and parameters, plus:

```json
{
  "acceptedCredits":5,
  "billingVersion":"RETURNED_BILLING_VERSION",
  "modelVersion":"RETURNED_MODEL_VERSION",
  "idempotencyKey":"UNIQUE_LOGICAL_SUBMISSION_ID"
}
```

The number 5 illustrates the response shape, not a guaranteed price. Never manufacture quote versions. Create returns a batch and job IDs; inspect the actual response. Persist those IDs immediately. `jobs.get {jobId}` and `jobs.list {projectId}` return status, charges and output asset metadata. Wait for `succeeded` and the returned output asset. Do not infer success from an upstream submission ID. Use the successful city asset as the character image's reference. Results generated with `stage:asset` already appear in `assets.list {projectId,scope:"library"}`. Do not call addToLibrary for them. That operation is for promoting a Draft/final result into reusable references and otherwise can return `NOT_GENERATION_RESULT`.

## Named prompts and candidates

`prompts.save` is for videos:

```json
{
  "projectId":"PROJECT_ID",
  "name":"City — character arrival",
  "text":"Use @[City](asset:CITY_ID) for environment and @[Character](asset:CHARACTER_ID) for appearance. Smooth third-person follow ...",
  "referenceIds":["CITY_ID","CHARACTER_ID"]
}
```

Use returned prompt ID and actual version in both the Draft quote and creation:

```json
{
  "projectId":"PROJECT_ID",
  "modelId":"LIVE_VIDEO_MODEL_ID",
  "stage":"draft",
  "name":"Character arrival candidates",
  "promptId":"SAVED_PROMPT_ID",
  "promptVersion":1,
  "parameters":{"resolution":"480p","ratio":"16:9","duration":5,"format":"mp4","count":2,"generateAudio":false}
}
```

Add approved quote fields and one idempotencyKey to create. For updates, `prompts.save` takes `promptId` and `expectedVersion`. Re-query on a version conflict. Do not reconstruct bindings as `[Reference 1]`, image1, or audio tags yourself. Images, video and audio have different provider notation; the server handles it.

## Selected final

Quote `stage:"production"` with `sourceJobId` of the successful same-model Draft; `parameters` specify supported final resolution, `count:1`, format, and the inherited ratio/duration. Do not attach a different prompt or different reference set. After agreement create with the quote fields and a new final-specific idempotencyKey. A Draft may expire at the provider; obtain the actual expiry from the result. Do not substitute a downloaded low-resolution Draft and describe it as native finalisation.

Use `assets.url {assetId,download:true}` for a fresh, short-lived URL; download actual bytes, then FFprobe. URLs are not permanent asset identities. Avoid logging their query strings. Poll jobs about every ten seconds, return control or a progress update during long waits, and stop polling terminal jobs. If a submission outcome is unknown, retain the pending budget and reconcile instead of spawning duplicates.

## Optional depth reference

For a user-supplied motion guide, discover the current internal depth model; upload a permitted video, quote the conversion and verify its duration/limits. It may be free in the catalog but still creates an asynchronous task. A depth result can remove appearance detail from a motion reference; it does not guarantee geometry or motion transfer. Count video plus depth durations together against the output duration before the video quote. Depth conversion is optional, not required for the two-image proof.
