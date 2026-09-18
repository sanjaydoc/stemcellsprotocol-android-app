# StemCells Protocol — Android app

An **offline, on-device AI healthcare assistant** for rural / low-connectivity areas
(parts of India & sub-Saharan Africa) where the nearest doctor can be hours away and
there is often no signal. It is a Capacitor build of [stemcellsprotocol.com](https://stemcellsprotocol.com).

> **Download:** [stemcells-protocol.apk](https://github.com/sanjaydoc/stemcellsprotocol-android-app/raw/main/stemcells-protocol.apk)
> Sideload: enable "Install unknown apps" for your browser, open the APK, install.

⚕️ **Not a medical device. Educational / decision-support only. It never replaces a
clinician — by design it defers to one whenever it is unsure.**

---

## Two modes

| Mode | Engine | Internet |
|------|--------|:--------:|
| **Online** | Cloud assistant (Anthropic Claude via a Cloudflare Worker), same as the website | required |
| **Offline** | **On-device LLM + RAG + LSM reliability gate** — runs entirely on the phone | **none** |

A toggle in the chat switches modes. Offline is the reason this app exists: it works
with no signal, and no data ever leaves the device.

---

## Offline architecture — LLM + RAG + LSM

```
question
   │
   ├─▶ 1. RETRIEVE   embed the question on-device → cosine search over a bundled
   │                 medical knowledge base → top-k evidence passages
   │
   ├─▶ 2. GENERATE   a small quantized medical LLM (Gemma / MedGemma via MediaPipe
   │                 LLM Inference) answers, grounded in those passages, with citations
   │
   └─▶ 3. DEFER GATE the LSM reliability layer combines the model's (calibrated)
                     confidence with the retrieval score. Low evidence OR low
                     confidence  →  "I'm not sure — please confirm with a clinician."
```

**1 · On-device LLM** — a small quantized medical model (2–4B, 4-bit) run locally via
Google **MediaPipe LLM Inference**. Small models are fallible, which is why steps 1 and 3 exist.

**2 · RAG (Retrieval-Augmented Generation)** — a compact, license-clean medical
**knowledge base** (the clinic's own protocols / therapies / FAQs plus open sources such
as StatPearls, MedlinePlus and WHO guidelines) is chunked and embedded at build time. On
the phone a small embedding model retrieves the most relevant passages and the LLM
answers **from that evidence**, with sources cited. Grounding raises accuracy and makes
"is there evidence for this?" a usable reliability signal.

**3 · LSM reliability / defer gate** — the layer whose defining trait is **knowing when
it is uncertain**. It fuses temperature-scaled model confidence with the retrieval score
and **hands back to a human clinician** when either is weak. On a small on-device model
this defer gate is the core safety feature, not an add-on.

### Why this design
Reliability first, not raw confidence. A grounded, cited answer that defers when the
evidence or confidence is thin is safe to put in a health worker's hand; a confident
guess is not.

---

## What's stored on the phone
After a one-time first-run setup (over Wi-Fi), everything is local and works offline:

| Piece | Size | Delivery |
|---|---|---|
| App shell (UI + native code) | ~5–10 MB | this APK |
| Embedding model | ~30–90 MB | bundled or first-run download |
| Knowledge base (text + vectors) | ~20–40 MB | bundled or first-run download |
| On-device LLM (4-bit) | ~1.5–2.8 GB | first-run download over Wi-Fi |

---

## Status
- ✅ **v1 (this APK):** the full app + working **online** chat; Android launcher icon = the StemCells Protocol mark.
- 🚧 **Offline stack (LLM + RAG + LSM):** in development — benchmarked per medical speciality
  before it ships (accuracy, calibration, and defer-gate safety guide the go/no-go).

_© Dr. Sanjay Anbu · StemCells Protocol. Confidential design — not for reproduction._
