# Marketing Content Rules

These rules apply to all files under `marketing/**`.

## Tone
- Write for a retail investor audience. No jargon without definition.
- First person plural ("we found," "our data shows"). Never third person ("the company's data shows").
- Active voice. "Congress traded $2.4M in tech stocks" not "Tech stocks were traded by Congress."

## Data Integrity
- Every chart must have a labeled source line (e.g., "Source: SEC EDGAR, via Acme Analytics").
- No dark-mode backgrounds on newsletters. Email clients render dark backgrounds inconsistently.
- Inline all CSS for email outputs. External stylesheets get stripped by Gmail and Outlook.

## Brand
- Company name is "Acme" in casual context, "Acme Analytics" in formal/legal context. Never "ACME" (all caps).
- Brand color: #3BB89A. Use for buttons and callout borders. Never for body text.
- No em dashes. Use periods, commas, or colons instead.

## Review Gates
- All customer-facing content requires review by the CEO before sending.
- Newsletter editions require a data freshness footer: "Data as of [date]."
- Quantitative claims require formula + source citation per the Quantitative Rigor Standard in CLAUDE.md.
