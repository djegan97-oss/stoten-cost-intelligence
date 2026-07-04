# ARCHIVED — 2026-07-04

**What this was:** Stoten Construction Cost Intelligence — the demo/pitch prototype. Single self-contained ~120KB HTML file with 11 panels (portfolio, regional, trade pricing, bid leveling, AI query mock), all dummy data, built May 2026 to show Stoten's construction managers what a production tool would look like.

**State when archived:** Functionally complete prototype with one known bug: 3 panels (AI Query, Data Foundation, New Estimate) render blank after scrolling (scroll-position not resetting; 16 fix attempts failed). Demo still live at https://pub.hyperagent.com/p/7kaTQ7s8VAwHvnET2R9H1A (v16). Local folder pushed as snapshot branch to the stoten-cost-intelligence repo at archive time.

**Why archived:** Purpose fulfilled — it sold the concept.

**Superseded by:** djegan97-oss/stoten-app — the v3 production rebuild (Next.js + Supabase, invite-only login, role-based access, deterministic cost engine), live as of July 2026.

**To revive:** `gh repo unarchive djegan97-oss/stoten-cost-intelligence`, restore from `~/Archive/Stoten_Construction_Intel`.
