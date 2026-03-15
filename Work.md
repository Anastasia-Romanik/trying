# Project Plan: Implementing PDF/X-1a Industry Standard in fpdf2

**Engineering Goal:** Expand the architecture of the `fpdf2` library to support the ISO 15930 (PDF/X-1a) standard, ensuring automatic validation (Compliance Enforcement) and color profile embedding.

**Total Workload:** 300 hours (100 hours per role).

---

## Roles & Responsibilities

### Role 1: Lead Software Architect (Yeliena) — 100 hours
*The "Passport Office": Ensuring the file officially identifies itself as a PDF/X.*

* **Focus:** Metadata, architectural design, and integration.
* **Main Challenge:** High concentration on documentation and abstract concepts. A single incorrect XML tag or placement will fail the standard compliance.
* **Tasks:**
    * Design architectural changes (UML Class Diagram) showing how new components interact with the FPDF core.
    * Implement an XMP metadata generator to add the required identification block for PDF/X.
    * Refactor the `pdf.output` method to introduce a `pdf_x_mode` (e.g., `pdf.output(pdf_x=True)`).
    * Create a comparative report: "PDF vs. PDF/X Architecture".
* **Resources to Study:**
    * Adobe XMP Specification (PDF section).
    * `fpdf2` source code, specifically `syntax.py` to understand object creation.

### Role 2: Core Graphics Developer (Polina) — 100 hours
*The "Geometer": Managing physical dimensions, boundaries, and typography.*

* **Focus:** Page geometry (Page Boxes) and font restrictions.
* **Main Challenge:** Font embedding is a technical nightmare. If a non-embeddable font is chosen, the system must fail gracefully with a clear error.
* **Tasks:**
    * Develop Page Boxes logic: Modify `add_page` to set `MediaBox`, `TrimBox`, and `BleedBox` using proper print offsets (converting mm to points, where 1 pt = 1/72 inch).
    * Implement Font Embedding Checks: Block the use of standard PDF fonts if they are not fully embedded into the document.
    * Create a Sequence Diagram for the page rendering process in PDF/X mode.
* **Resources to Study:**
    * PDF Reference (v1.7), Section 10.1 ("Page Boundaries") and Section 5.7 ("Font Descriptors").
    * Prepress guides on Bleed, Trim, and Media Boxes.
    * `fpdf2` documentation on Unicode & TrueType Fonts (`fpdf.py` and `fonts.py`).

### Role 3: Compliance & DevOps Engineer (Anastasia) — 100 hours
*The "Colorist & Police": Enforcing strict rules and validating the entire team's work.*

* **Focus:** Color management (ICC), validation, and CI/CD.
* **Main Challenge:** Configuring CI/CD environments. Integrating a Java-based tool (VeraPDF) into GitHub Actions alongside Python tests can be complex.
* **Tasks:**
    * Implement ICC profile embedding (e.g., `ISOcoated_v2`) via `OutputIntents` in the final PDF.
    * Develop a "Strict Mode": Intercept and block RGB colors and transparency (alpha channels). Restrict color models strictly to CMYK or Grayscale in methods like `set_draw_color`, `set_fill_color`, and `image`.
    * Write automated tests (pytest) to verify Page Boxes, fonts, and colors.
    * Set up a CI/CD Pipeline (GitHub Actions) to run tests and validate generated PDFs via VeraPDF on every push.
* **Resources to Study:**
    * Color profiles (ICC) and Output Intents (PDF Reference).
    * VeraPDF documentation and CLI usage.
    * Guides on "CMYK vs RGB" for prepress understanding.

---

## Engineering Components

### 1. Architectural Design
Before writing code, the team must document the solutions:
* **UML Class Diagram:** Interaction of new classes (e.g., `XMPManager`, `ColorCompliance`) with the core.
* **Sequence Diagram:** Data flow from object creation to byte writing.

### 2. Quality Assurance
* **Metrics:** Code Coverage and Benchmarking (generation time).
* **VeraPDF Integration:** The project is only considered complete when VeraPDF yields zero validation errors.

### 3. Compliance Enforcement
This is the core engineering logic: methods must verify color models, forbid transparency, and raise clear, user-friendly exceptions when rules are violated.

---

## Roadmap

| Phase | Tasks | Deadline |
| :--- | :--- | :--- |
| **Research & Design** | UML diagrams creation, ISO study, ICC profile selection. | Week 2 |
| **Prototype** | Page Boxes implementation and basic metadata (without color). | Week 4 |
| **Core Dev** | Output Intents, ICC embedding, and "Strict Mode" for colors. | Week 7 |
| **Automation** | CI/CD pipeline setup with VeraPDF validation. | Week 8 |
| **Validation** | Fixing validator issues, Benchmarking. | Week 9 |
| **Final Docs** | Finalizing reports and architectural documentation. | Week 10 |

---

## Risks & Mitigation

1.  **Font Complexity:** Embedding fonts is challenging.
    * *Solution:* Verify if `fpdf2` already embeds the selected font; if not, throw an explicit error during initialization.
2.  **Version Incompatibility:** Different PDF/X versions have conflicting requirements.
    * *Solution:* Strictly limit the project scope to the **PDF/X-1a:2001** standard.

---

## Quick Start (First 5-10 hours)
A recommended "Reverse-Engineering" exercise for all team members:
1. Find a valid PDF/X-1a file online.
2. Open it in a raw text editor (e.g., Notepad++).
3. Search for tags: `/GTS_PDFXVersion`, `/TrimBox`, `/OutputIntents`.
4. Understand that these tags are simply text that your Python code needs to generate dynamically.
