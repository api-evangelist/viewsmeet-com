---
name: viewsmeet-score-assessment-vector
description: Retrieve one of ViewsMeet's exact public-domain self-report instruments (IPIP-50, IPIP-IPC 32, 36QB6, or Mini-IPIP 20), inspect its methodology, and score a complete response vector statelessly.
api: openapi/viewsmeet-com-openapi.yml
operations: [listStatelessAssessments, getStatelessAssessmentInstrument, getStatelessAssessmentMethodology, scoreStatelessAssessmentResponses, getMiniIpipPersonalityInstrument, getMiniIpipMethodology, scoreMiniIpipPersonalityResponses]
generated: '2026-09-19'
method: generated
source: Grounded in openapi/viewsmeet-com-openapi.yml and https://viewsmeet.com/developers#assessments; every operationId below exists verbatim in the spec.
---

# Score a self-report vector without persistence

Every operation here is anonymous and stateless. Scoring responses are returned with `Cache-Control: private,
no-store`; the vector and scale values are not written to any ViewsMeet store, account, friend link or model.

## Three-instrument catalog (IPIP-50, IPIP-IPC 32, 36QB6)

1. `listStatelessAssessments` — GET `/api/v1/assessments`. The closed catalog returns exactly three entries with
   `slug`, `id` (e.g. `ipip-big-five-50-en-v1`), `version`, `itemCount`.
2. `getStatelessAssessmentInstrument` — GET `/api/v1/assessments/{slug}/instrument`. Use the fixed item wording,
   order and `responseOptions` exactly; display order may vary, wording and scoring do not.
3. `getStatelessAssessmentMethodology` — GET `/api/v1/assessments/{slug}/methodology`. Read `publicDomain`,
   `sourceUrls`, `scoring`, `limitations` and `excludedInterpretations` BEFORE interpreting any result.
4. Collect every item exactly once, then `scoreStatelessAssessmentResponses` — POST `/api/v1/assessments/{slug}/score`
   with `{"instrumentId": "<id from the catalog>", "version": <version>, "responses": [{"itemId": "...", "response": n}, ...]}`.
   A wrong id/version or an incomplete vector is a 400 with `issues[]`; an unknown slug is 404; 503 means scoring is
   temporarily unavailable — retry later, never substitute sample output.
5. Report `scales[]` as continuous raw sums and means only, with the returned `disclosure`.

## Mini-IPIP 20 (the MCP-shared path)

- `getMiniIpipPersonalityInstrument` — GET `/api/v1/personality/instrument` (20 items, five 1-5 anchors).
- `getMiniIpipMethodology` — GET `/api/v1/personality/methodology`.
- `scoreMiniIpipPersonalityResponses` — POST `/api/v1/personality/score` with `{"answers": [20 x {itemId, response}]}`
  returns five raw factor means. The same scorer is the MCP tool `score_personality_responses` (argument name
  `responses`) — see mcp/viewsmeet-com-tool-crosswalk.yml.

## Rules that apply throughout

- These instruments were developed for human self-report. A software-produced vector "is only a labeled configuration
  response and is not a validated agent personality" (llms.txt). Never present output as a diagnosis, type, percentile,
  IQ, compatibility result, hiring signal or proof of consciousness or identity (Terms; spec descriptions).
- Rate limit 30 requests / 60 s per operation; instrument and methodology documents are public-cached (max-age 86400)
  and may be revalidated per their cache headers.
- Idempotency is not a concern here: scoring is stateless and repeat-safe (conventions/viewsmeet-com-conventions.yml).
