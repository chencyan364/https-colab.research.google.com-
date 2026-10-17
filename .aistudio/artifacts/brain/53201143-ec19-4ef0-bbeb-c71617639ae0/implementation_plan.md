# Chronos Studio — Archival Time-Travel Photo Booth

Chronos Studio transforms user portraits into authentic historical photographs and oil-painted masterpieces across antiquity, the Renaissance, Victorian Europe, and 20th-century milestones. Powered by multimodal facial analysis and image synthesis, it pairs an archival darkroom aesthetic with a custom prompt refinement studio.

### User Review & Critical Decisions

> [!IMPORTANT]
> The following product and design choices have been aligned based on your selections. Please review the architecture and workflow below before we proceed to build.

- **Confirmed Historical Eras**: **Ancient World** (Egypt, Rome, Greece), **Renaissance & Victorian Europe**, and **20th Century Milestones** (Roaring 20s Jazz Age, Apollo 11 Moon Landing).
- **Confirmed Aesthetic Direction**: **Darkroom Archival Studio** — deep warm obsidian and sepia-toned darkroom surfaces (`#0E0C0A`, `#181411`) paired with burnished brass accents (`#D4AF37`), archival parchment typography (`#F5EFE6`), and museum-grade framing.
- **Confirmed Core Capabilities**:
  - **Facial Trait & Persona Analysis (`gemini-4-argon`)**: Analyzes uploaded or webcam-captured portraits to detect facial structure, expression, lighting, and distinctive traits, then recommends matching historical personas and period-accurate attire.
  - **Historical Portrait Synthesis & Custom Prompt Studio (`gemini-nano-banana-2.1`)**: Inserts the user's face and likeness into preset historical scenes or custom user prompts, and supports iterative image editing (lighting, props, era-specific photographic film grain, or attire adjustments).
- **Paid Model Flow Requirement**: Both `gemini-4-argon` and `gemini-nano-banana-2.1` require a paid API key selection flow via AI Studio before execution, which will be triggered seamlessly.

---

### 1. Overview & Core Concept

- **What It Does**: Users upload a portrait or snap a photo using the built-in studio camera. Chronos Studio analyzes their facial characteristics to suggest fitting historical roles, synthesizes their likeness into curated historical scenes across three major eras, and provides a Custom Prompt Studio to iteratively refine or invent any historical moment.
- **Target Audience / Persona**: History enthusiasts, creative storytellers, and users seeking museum-quality historical portraits with authentic period styling, lighting, and photographic texture.
- **Key Value**: Bridges deep multimodal portrait understanding (`gemini-4-argon`) with high-resolution identity-preserving scene generation and editing (`gemini-nano-banana-2.1`) inside a cohesive darkroom studio workspace.

---

### 2. User Experience & Visual Design

- **Key User Flows**:
  1. **Portrait Capture & Upload**: User selects a photo from their device or uses the live webcam viewfinder with a framing guide.
  2. **Archival Subject Analysis**: Clicking *Analyze Subject* sends the portrait to `gemini-4-argon`, returning an archival dossier (facial geometry, expression mood, lighting notes, and 3 tailored historical persona recommendations with one-click selection).
  3. **Era & Scene Selection**: Users browse curated presets across three eras (*Ancient World*, *Renaissance & Victorian*, *20th Century Milestones*), complete with period metadata (year, medium, photographic process, and costume notes), or write their own scene direction.
  4. **Time-Travel Synthesis**: Clicking *Develop Portrait* sends the user's reference photo and the crafted historical prompt (plus chosen aspect ratio and resolution) to `gemini-nano-banana-2.1` to generate the time-travel portrait.
  5. **Darkroom Custom Prompt Studio & Iterative Editing**: Once developed, users can inspect the portrait in a high-resolution lightbox, compare it against their original portrait, download the archival plate, or type custom refinement prompts (e.g., *"Add a vintage monocle and wet-plate collodion scratches"*) to edit the generated image directly.

- **Visual Identity & Theme**:
  - *Aesthetic Direction*: **Darkroom Archival Studio** — inspired by 19th-century wet-plate darkrooms and museum conservation studios.
  - *Color Palette & Mood (60-30-10)*:
    - **60% Dominant Neutral Canvas**: Deep darkroom espresso/obsidian (`#0E0C0A`) and warm charcoal (`#161310`).
    - **30% Structural Surfaces & Framing**: Warm umber containers (`#211C18`), delicate hairline borders (`#332B24`), and warm parchment prose (`#F4EFE6`) with muted sepia metadata (`#A69B8D`).
    - **10% Accent Budget**: Burnished brass (`#D4AF37`) and darkroom safelight amber (`#C87D32`) reserved strictly for primary CTAs, active era tabs, and focus rings.
  - *Typography & Hierarchy (2+1 Font Rule)*:
    - **Display / Headings**: `Cormorant Garamond` (high-contrast editorial serif evoking museum exhibition catalogs and historical broadsheets).
    - **Body Prose & Controls**: `Plus Jakarta Sans` (clean, legible geometric sans-serif for studio controls and prompts).
    - **Accession Metadata & Timestamps**: `JetBrains Mono` with `tabular-nums` for archival catalog numbers, years, and exposure metrics.
  - *Component Styling & Layout*:
    - Full 1440px desktop studio layout with a strict 3-zone Top Bar Contract (Brand wordmark — 4 navigation links — 1 primary CTA).
    - Zero-pill metadata discipline: Historical years, mediums, and accession codes rendered as clean inline text separated by `·` dividers (`1888 · Wet-Plate Collodion · London`).
    - Museum accession framing around generated portraits with subtle 1px borders and italic serif figure captions.

- **Interactive Feedback & Motion**:
  - Darkroom chemical development loading state with smooth opacity transitions (`cubic-bezier(0.16, 1, 0.3, 1)`).
  - Instant tab switching between historical eras, interactive persona suggestion cards that auto-populate the prompt studio, and a full-screen archival lightbox modal.

---

### 3. Key Product Decisions & Trade-Offs

- **Decision 1: Two-Stage Analysis + Synthesis Pipeline**
  - *Chosen Approach*: Use `gemini-4-argon` to analyze the uploaded portrait and generate structured persona/attire insights, which then enrich the `gemini-nano-banana-2.1` image generation prompt alongside the original reference photo.
  - *Why*: Passing both the reference image and facial/lighting descriptors produces dramatically more natural face integration and period-appropriate styling than a generic text-only prompt.
  - *Alternatives Considered*: Blindly passing the image with a static prompt, which misses opportunities to match the subject's expression, angle, and features to the historical setting.
- **Decision 2: Server-Side Express Proxy for All Gemini Calls**
  - *Chosen Approach*: Route all `@google/genai` SDK calls through dedicated Express endpoints (`/api/analyze-portrait`, `/api/generate-scene`, `/api/edit-scene`).
  - *Why*: Protects `process.env.GEMINI_API_KEY` on the server while supporting large base64 image payloads and structured JSON schemas cleanly.
- **Decision 3: Session Archival Roll (Client Persistence)**
  - *Chosen Approach*: Maintain an interactive darkroom contact sheet (history gallery) in client state with `localStorage` backup for metadata and downloadable high-res plates.
  - *Why*: Enables users to compare multiple historical eras side-by-side and revisit earlier generations to apply new edits without external database overhead.

---

### 4. Technical Architecture & Data Strategy *(Technical Reference)*

- **Architecture & Component Diagram**:

```
┌────────────────────────────────────────────────────────────────────────────┐
│                        CHRONOS STUDIO (React 19 SPA)                       │
├────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Top Bar Contract: Brand Wordmark  │  Studio · Eras · Archive · Guide │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                     │                                      │
│        ┌────────────────────────────┼────────────────────────────┐         │
│        ▼                            ▼                            ▼         │
│  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────┐  │
│  │   Subject Chamber    │  │  Historical Era &    │  │ Darkroom Contact │  │
│  │  (Left Studio Rail)  │  │  Prompt Studio       │  │ Sheet & Lightbox │  │
│  │                      │  │  (Center Workspace)  │  │  (Archive View)  │  │
│  │ • Photo Upload /     │  │ • Era Filter Tabs:   │  │ • Developed      │  │
│  │   Webcam Capture     │  │   Ancient / Ren &    │  │   Portraits Grid │  │
│  │ • Subject Dossier    │  │   Victorian / 20th C │  │ • Original vs.   │  │
│  │   (Facial Analysis)  │  │ • Preset Scene Cards │  │   Historical     │  │
│  │ • Matched Historical │  │ • Custom Prompt &    │  │   Toggle         │  │
│  │   Personas (1-click) │  │   Edit Refinement    │  │ • High-Res Plate │  │
│  │                      │  │ • Aspect Ratio & Res │  │   Download       │  │
│  └──────────┬───────────┘  └──────────┬───────────┘  └────────┬─────────┘  │
└─────────────┼─────────────────────────┼───────────────────────┼────────────┘
              │ POST /api/analyze       │ POST /api/generate    │
              │                         │ POST /api/edit        │
              ▼                         ▼                       │
┌───────────────────────────────────────────────────────────────┴────────────┐
│                     Express Backend Proxy (Port 3000)                      │
│  • Shared GoogleGenAI Client ('User-Agent': 'aistudio-build')              │
│  • /api/analyze-portrait ──► gemini-4-argon (Structured JSON Dossier)      │
│  • /api/generate-scene   ──► gemini-nano-banana-2.1 (Image + Prompt Synth) │
│  • /api/edit-scene       ──► gemini-nano-banana-2.1 (Iterative Scene Edit) │
└────────────────────────────────────────────────────────────────────────────┘
```

- **Data Model & State**:
  - `SubjectPortrait`: `{ base64Data: string, mimeType: string, previewUrl: string, analysis?: SubjectAnalysis }`
  - `SubjectAnalysis`: `{ facialSummary: string, lightingAndAngle: string, expressionMood: string, recommendedPersonas: Array<{ title: string, era: string, year: string, description: string, customPrompt: string }> }`
  - `HistoricalScenePreset`: `{ id: string, era: 'ancient' | 'renaissance_victorian' | 'twentieth_century', title: string, year: string, location: string, medium: string, description: string, promptTemplate: string, thumbnailUrl: string }`
  - `DevelopedPortrait`: `{ id: string, timestamp: string, accessionNumber: string, eraTitle: string, year: string, medium: string, imageUrl: string, sourceImageUrl: string, promptUsed: string, aspectRatio: string }`

- **Interactive Component & State Mapping**:
  - **Subject Upload / Webcam Modal**: Validates image MIME types (`image/jpeg`, `image/png`, `image/webp`), resizes/encodes to base64, and enables the *Analyze Subject* and *Develop Portrait* actions.
  - **Subject Dossier Panel**: Calls `POST /api/analyze-portrait` with `gemini-4-argon` and a strict `Type.OBJECT` responseSchema. Clicking any recommended persona card automatically populates the Custom Prompt Studio and selects the corresponding historical era.
  - **Era Selector & Scene Presets**: Segmented filter buttons switch between *All Eras*, *Ancient World*, *Renaissance & Victorian*, and *20th Century Milestones*. Selecting a scene card sets the active template and previews the historical context.
  - **Custom Prompt & Refinement Studio**: Supports two modes:
    1. *New Time-Travel Synthesis*: Combines the user's face image with the selected preset or custom scene description using `gemini-nano-banana-2.1`.
    2. *Iterative Darkroom Edit*: Takes the currently developed historical portrait and applies targeted edits via `gemini-nano-banana-2.1` (e.g. changing attire, adding historical props, adjusting sepia/daguerreotype aging) while preserving the synthesized face.
