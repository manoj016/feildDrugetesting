# Digital Companion for Field Drug Testing

> **IMPORTANT LEGAL & REGULATORY NOTICE**  
> **“Presumptive field-test result — laboratory confirmation required.”**  
> *This application provides a presumptive field-test result and supporting digital record. It does not replace laboratory confirmatory testing (e.g. GC/MS, LC/MS, FTIR) and has NOT been validated for forensic, medical, legal, or evidentiary use in court proceedings without accredited laboratory confirmation.*

---

## 1. Project Overview

The **Digital Companion for Field Drug Testing** is a responsive, mobile-first Progressive Web Application (PWA) designed for authorized field testing operators. It enables operators to photograph completed colorimetric field tests alongside a reference colour card, perform ambient light normalization and colorimetry calibration, evaluate presumptive indications using a modular classification engine, and generate tamper-evident digital records sealed with cryptographic SHA-256 hashes and verification QR codes.

No proprietary hardware is required: the system utilizes standard smartphone/tablet/webcam sensors and the Web Crypto API.

---

## 2. Key Features

- **Mobile-First Guided Workflow:**
  1. Operator authorization & test metadata entry (Test ID, Case Reference, Lot number).
  2. Live camera viewfinder with high-tech HUD overlays (framing guides for reference card and reagent window, live luminance meter, sharpness detector).
  3. 4-point image quality validation (Lighting, Sharpness, Reference card detection, Reagent window isolate).
  4. Prototype 6-patch reference color card calibration (White balance multipliers, brightness offset, $\Delta E$ swatch delta calculation).
  5. Modular classification engine outputting POSITIVE, NEGATIVE, or INCONCLUSIVE indication with confidence percentage and detailed forensic reasoning.
  6. GPS location capture and ISO 8601 timestamping.
  7. Dual SHA-256 cryptographic hashing (Raw image binary digest + Canonical record seal).
  8. Interactive verification QR code generation.
  9. Printable Court-Ready Presumptive Field Certificate with security guilloche styling.
  10. Downloadable structured JSON record.

- **Evaluator Demo Mode:**
  Built-in 5-scenario simulator for live evaluations without physical reagents:
  - Scenario 1: **Positive Indication** (Deep violet chromogenic shift)
  - Scenario 2: **Negative Indication** (Pale yellow baseline unreacted profile)
  - Scenario 3: **Inconclusive Indication** (Ambiguous intermediate spectrum)
  - Scenario 4: **Poor Lighting / Low Quality** (Underexposure warning trigger)
  - Scenario 5: **Missing Reference Card** (Alignment guidance prompt)

- **Searchable Field Records Archive:**
  - Search by Test ID, Operator, Case Reference, or SHA-256 hash.
  - Filter by Positive, Negative, Inconclusive, and date range.
  - Batch export complete database archive to JSON.

- **Cryptographic Verification Portal:**
  - Independent verification gateway allowing auditors to upload JSON records or scan QR codes.
  - Recalculates the canonical SHA-256 digest from record fields in real time and detects tampering.
  - Interactive "Simulate Tampering" feature demonstrating cryptographic mismatch alerts.

- **Offline-First Resilience:**
  - Full local persistence using browser IndexedDB with transparent fallback to localStorage.
  - Works seamlessly offline in remote field areas without active network connectivity.

---

## 3. Architecture

```
┌───────────────────────────────────────────────────────────────┐
│              Presentation Layer (React 18 + TS)               │
│  [Dashboard]  [NewTest Wizard]  [History]  [Verify] [Settings]│
└───────────────┬───────────────────────────────┬───────────────┘
                │                               │
┌───────────────▼──────────────┐┌───────────────▼──────────────┐
│       Sensor & Capture       ││      Cryptographic Engine    │
│  - MediaDevices Camera API   ││  - Web Crypto API (SHA-256)  │
│  - Geolocation API           ││  - Canonical Payload Builder │
│  - Canvas Pixel Sampler      ││  - Tamper Detection Auditor  │
└───────────────┬──────────────┘└───────────────┬──────────────┘
                │                               │
┌───────────────▼──────────────┐┌───────────────▼──────────────┐
│  Image & Calibration Engine  ││     Modular Classifier       │
│  - 4-Point Quality Metrics   ││  - classificationEngine.ts   │
│  - 6-Patch Swatch Sampler    ││  - RGB / HSL / ΔE Distances  │
│  - White Balance Matrix      ││  - Configurable Thresholds   │
└───────────────┬──────────────┘└───────────────┬──────────────┘
                │                               │
┌───────────────▼───────────────────────────────▼──────────────┐
│                 Persistent Local Storage                      │
│      - IndexedDB (FieldDrugTestingDB) / localStorage          │
│      - JSON Archive Exporter / Certificate Renderer          │
└───────────────────────────────────────────────────────────────┘
```

---

## 4. Cryptographic Hashing Specification

To ensure records are tamper-evident, the application computes two distinct SHA-256 hashes:

1. **Original Image Binary SHA-256:**
   Computed on the uncompressed raw image byte buffer using:
   ```ts
   window.crypto.subtle.digest('SHA-256', rawImageBuffer)
   ```
2. **Canonical Record SHA-256:**
   Computed deterministically across all primary metadata fields:
   ```
   testId=<ID>||timestamp=<ISO>||operatorId=<OP>||latitude=<LAT>||longitude=<LNG>||imageHash=<HASH>||classification=<STATUS>||confidence=<CONF>||testType=<TYPE>||caseReference=<REF>
   ```

---

## 5. Modular Classification Engine

Located in [`src/services/classificationEngine.ts`](file:///c:/Users/MOUNIKACHARY/manoj%20hack/hackethon/src/services/classificationEngine.ts), the engine processes calibrated pixel data and outputs:
- `classification`: `"POSITIVE" | "NEGATIVE" | "INCONCLUSIVE"`
- `confidence`: `0 - 100%`
- `reason`: Human-readable presumptive interpretation
- `opticalMetrics`: Hue, Saturation, Lightness, and $\Delta E$ Euclidean distance metrics.

The architecture is structured so that machine learning backends (such as ONNX Runtime Web or TensorFlow.js) can replace or enhance the prototype colorimetric rules.

---

## 6. Installation & Running Locally

### Prerequisites
- Node.js version 18+ or 22+
- npm version 9+

### Steps
```bash
# 1. Clone or navigate to the project directory
cd "c:\Users\MOUNIKACHARY\manoj hack\hackethon"

# 2. Install dependencies
npm install

# 3. Start the local development server
npm run dev

# 4. Or build and preview the production bundle
npm run build
npm run preview
```

Open `http://localhost:3000` in your web browser.

---

## 7. Demo Credentials

For demonstration and testing purposes, you can select any preset profile or enter custom credentials on the login screen:
- **Operator ID:** `OP-001`
- **Name:** `Officer J. Thorne`
- **Organization:** `State Forensic Field Operations`
- **Unit:** `Mobile Narcotics Screen Unit 4`

*(No live sensitive credentials are required or exposed).*

---

## 8. Limitations & Security Considerations

- **Presumptive Only:** Colorimetric field testing is inherently presumptive and cannot distinguish between structural analogues or confirm purity. Laboratory testing is required.
- **Client-Side Prototype:** Cryptographic hashes are computed in-browser using Web Crypto. In an enterprise deployment, records would also be signed with an operator hardware token or PKI private key and synchronized with a central evidence vault.
- **Ambient Lighting:** While 6-patch color card normalization compensates for moderate lighting shifts, extreme low-light or direct specular glare will trigger the built-in quality warning.

---

## 9. Future Roadmap

1. Integration of ONNX Runtime Web for multi-panel test cassette deep-learning segmentation.
2. Integration with state police CAD (Computer Aided Dispatch) and LIMS (Laboratory Information Management System).
3. Digital signatures via hardware security keys (FIDO2 / WebAuthn).
