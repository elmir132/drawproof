# The AI verifier and the design preview

Notes from [Elmir Abdullaiev](https://github.com/elmir132) on the two parts of Drawproof he built at the AI & Fintech XRPL Hackathon NYC
(Oct 24-25, 2026). The rest of the system (XRPL money path, orchestrator, web app, inspector app) was built by the teammates credited in the
[README](../README.md).

## What the verifier does

`packages/verifier` decides whether a milestone's evidence looks consistent enough to feed the release gate. It checks evidence consistency,
not construction quality. A passing verdict is an input to the orchestrator's release gate. It does not sign or release an escrow: the core
attests the exact verdict hash and the inspector's signature is still required.

For each milestone it runs a suite of checks over the builder's photos, invoice and paperwork, next to the inspector's signed audit:

- **Required evidence is present** for the milestone type.
- **Photo timing and location.** Capture time must fall after the escrow was created and after the previous milestone was approved, and the
  location must be near the site. EXIF is spoofable, so these are labelled as catching lazy fraud only.
- **No duplicate photos**, using a SHA-256 hash and a perceptual hash compared against earlier milestones and other projects. The perceptual
  hash survives re-compression, resizing and small crops, which is why a reused photo is caught.
- **Invoice against the budget line** within a tolerance band, and a **lien waiver** check.
- **Audit consistency.** The inspector's checklist (done, partial, not done) is compared with what the evidence shows, so a "done" line with
  no supporting photo is flagged.
- **A release policy** turns the results into `pass`, `fail` or `needs_human`, and a hash of the verdict is what the core attests on-chain.

It also checks a builder's license and insurance documents against the builder record and a **mock** registry before the verified-builder
credential is decided, and it assigns an inspector. Today only text and JSON documents are parsed (deterministically, in both modes);
reading PDF or photo documents with Claude is still a TODO.

Judgement checks (does this photo look like framing, does the audit match the photos) go through a provider adapter:

| Provider | Status |
|---|---|
| `mock` | Deterministic and key-free, the default. Stage judgement uses filename tokens and audit judgement uses checklist status. It does no real image analysis. |
| Claude (Anthropic SDK) | Implemented and tested against fake responses. |
| Claude on Amazon Bedrock | Implemented and merged. The one live call we tried was denied by an IAM permission error, so it is not verified live. |

## What is and is not validated

- The demo ran the verifier in **mock mode**. Every judgement reason in mock mode is labelled `[mock]`.
- Real Claude stability on real photos is **unvalidated**. The live path was exercised with fake responses, not scored over repeated real runs.
- The package has more than 100 test cases, adversarial fixtures that must never auto-pass, and an evaluation
  harness (`pnpm --filter @xrlp/verifier eval --mode mock --repeat 3`). Those scenarios were written by the same team that wrote the checks, so
  a pass on them is a regression guard, not an accuracy figure.

See [`packages/verifier/API.md`](../packages/verifier/API.md) for commands and the adapter contract.

## The design preview

`design/drawproof-ui` is a clickable Next.js preview of the investor, builder and inspector screens on **mock data**, built to settle the
look and flow before the live apps adopted it. It includes the inspector side (queue, audit with signing, site visits, history, read-only
audit view). It was merged upstream as PRs
[#204](https://github.com/Jalil-g/XRLP-Hack/pull/204) and [#206](https://github.com/Jalil-g/XRLP-Hack/pull/206), and the live apps then
restyled to match. It also holds the recorder (`tools/record-demo.mjs`) that drives the real preview in Chrome to capture demo footage.

## The narrated demo video

A 90-second narrated walkthrough made from footage of that preview: the money bar, an honest builder, a reused photo caught, the compare
view, the inspector signing, and the desktop, tablet and phone layouts. It was made with [HyperFrames](https://github.com/heygen-com/hyperframes)
(HTML rendered to video) and a local text-to-speech voice. All data is mock and the video says so on screen.

Watch or download it from the [release page](https://github.com/elmir132/drawproof/releases/tag/demo-videos-2026-10-25).
