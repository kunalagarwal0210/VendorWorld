# VMS / VendorWorld — Prototype Handoff

**Artifact:** `vms_prototype.html` — a single-file, in-memory, hash-routed clickable prototype of a Vendor Management System (branded **VendorWorld** on the landing/login, with an embossed **n̄.Ai+ve Tribe** logo mark; **VMS** in-app).
**Status:** Clickable prototype — front-end only, no backend, no build step.
**Handoff date:** 2026-08-11
**Audience:** Anyone who needs to view, demo, or extend the prototype.

---

## 1. How to run it (identical rendering guaranteed)

1. Copy the **entire** code block in [§8 Full source](#8-full-source) — everything between the `BEGIN` and `END` markers.
2. Save it as a file named **`vms_prototype.html`** (UTF-8, no changes).
3. Double-click it / open it in any modern browser (Chrome, Edge, Safari, Firefox).

That's it — no install, no server, no internet connection required.

### Why the UI renders the same on every machine
This is intentional so a teammate sees exactly what you see:

- **Single self-contained file.** All HTML, CSS, and JavaScript are inline. There is nothing to bundle or link.
- **Zero external dependencies.** No CDN scripts, no web-font downloads, no icon library, no analytics. (The only `http` string in the file is an SVG XML *namespace* on the Google logo — an identifier, not a network request.) Icons are inline Unicode/SVG.
- **No build tooling.** It is plain ES-safe vanilla JS in one `<script>`; the browser runs it directly.
- **Design tokens are inline CSS variables**, so colors/spacing are locked in the file, not inherited from any host stylesheet.

> The one thing that varies slightly by OS is the **system UI font** (the stack is `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, …`). This is by design — the prototype uses each OS's native font rather than shipping a font file. Layout, colors, spacing, and behavior are identical everywhere. If you need pixel-identical type too, tell the receiving team and we can pin a bundled font (adds weight to the file).

**Do not** re-save through a rich-text editor, reformat/minify, or "clean up" the HTML — that is the only way rendering could drift. Keep it byte-for-byte.

---

## 2. State reset behavior

All state lives in a single in-memory `state = {…}` object. **Refreshing the page resets everything** to the seed data. This is expected for a prototype — there is no persistence layer.

---

## 3. Architecture (for anyone extending it)

- **Global state:** one `let state = {…}` object holds the current user, vendor list, filters, activity log, and transient per-flow UI flags (`onbStep`, `pennyState`, `docChecks`, `bankUploads`, …).
- **Screens are functions that return HTML strings.** Two chrome wrappers:
  - `shell(active, body)` — top-bar chrome (most internal screens).
  - `clientShell(active, body)` — dark left sidebar (Home + Admin Console).
- **Routing:** a `routes` array of `[regex, handler]` pairs; `render()` runs on `hashchange` and injects into `#root`. A guard ignores non-`#/` hashes so in-page anchors don't blank the screen.
- **Interaction handlers** are attached as `window.*` functions and called from inline `onclick`/`oninput`.

### Routes / screens
| Hash | Screen |
|------|--------|
| `#/` | Landing (VendorWorld marketing + Request-a-Demo modal) |
| `#/login`, `#/login/vendor`, `#/login/client`, `#/login/client/reset`, `#/login/google` | Auth (Vendor OTP · Client email+password · Admin Google SSO, all simulated) |
| `#/signup`, `#/org-setup` | Admin signup wizard + org setup |
| `#/home` | Client **Home** (sidebar + KPIs + vendor-list preview) — nav item renamed from "Dashboard" |
| `#/admin` | **Admin Console** — analytics dashboard (KPIs, funnel, TAT, vendor tracking + CSV export, activity) |
| `#/vendors` | Vendor directory / list (search + filters) |
| `#/vendor/:id` | Vendor profile |
| `#/request` → `#/invited/:id` | Raise a new request → hand-off to vendor flow |
| `#/onboard/:id` | Vendor onboarding (Details · Documents · **Declaration** · Bank Details + penny drop) |
| `#/verify/:id` → `#/risk/:id` → `#/approve/:id` → `#/activate/:id` | Due diligence → risk (reviewer assessment) → approvals → activation |
| `#/activity` | Org-wide activity log |
| `#/analytics` | Analytics (parked) |

---

## 4. Recent changes in this build

### Risk process split — vendor declaration vs reviewer assessment (2026-08-12)
The single risk questionnaire (previously answered entirely by the reviewer on `#/risk`) is now split by audience:
- **Vendor self-declaration** — a new **Declaration** step in onboarding (`#/onboard/:id`, after Documents). The vendor self-selects applicability (`v.drivers`) and self-attests the domain questions (stored in new **`v.declAnswers`**), then ticks a declaration checkbox. The onboarding stepper is now **4 steps** (Details · Documents · Declaration · Bank Details).
- **Reviewer assessment** — the `#/risk` screen now shows a **different, reviewer-oriented question set** (`REVIEWER_DOMAINS`) answered by the reviewer (`v.riskAnswers`). **These reviewer answers are what `computeRisk()` scores** into Low/High. The vendor's declaration is shown alongside as a read-only **context card** for comparison; applicability drivers on this screen are pre-filled from the vendor's declaration and remain adjustable.
- `DOMAINS` (unchanged wording) is now the *vendor declaration* content; `REVIEWER_DOMAINS` (new, same keys/weights/applicability) is the *scored* reviewer set. The scoring engine (weighted supporting score, K1 critical knockouts, K2 bank mismatch, ≥75% → Low) is otherwise unchanged.

### Latest session (2026-08-11 — rebrand + UX pass)
- **Rebrand: Vendor IQ → VendorWorld.** All product-name copy on the landing/login now reads **VendorWorld**. The old gradient **"IQ"** logo mark is replaced by an **embossed `n̄.Ai+ve Tribe`** pill (the team name, used as-is) in the landing header. In-app **VMS** naming is unchanged.
- **Removed the "V" logo badge** (`<span class="logo">V</span>`) from the top bar and all auth pages; the top bar keeps the "VMS" wordmark.
- **Sidebar nav "Dashboard" → "Home"** (route `#/home` unchanged); the post-activation "Back to dashboard" link is now "Back to home". Frees "dashboard" to mean the Admin Console.
- **Vendor list columns** — "Primary contact" split into **Vendor Contact** (`v.contact`) and a new **Client Contact** (`v.raisedBy`) column; all headers title-cased. Client-Contact seed names were changed to a distinct internal pool (N. Kulkarni, V. Deshpande, P. Joshi, R. Kamat, S. Borse) so they no longer collide with Vendor-Contact names (repeats across rows are intentional — one manager owns several vendors).
- **Filters as dropdowns** — Status, Risk, and Department filters in `vendorSearchPanel()` are now `<select>` dropdowns (shared `sel()` helper) instead of chips.
- **Documents step uploads now match Bank Details** — real hidden file input per doc, shows the actual chosen filename (`onDocFile`), no more fake `uploadDoc`.
- **Upload remove via ✕** — on both Documents and Bank Details, an uploaded file shows a **✕ remove** control (`.file-x`, `removeDoc`/`removeBankFile`) instead of a Replace button; the ✕ appears only when a file is present and reverts the row to the Upload state.
- **Penny-drop screen** — removed the "Verification complete" subtitle from the onboarding header on the penny-drop step.

### Earlier build
- **Bank step renamed** to **"Bank Details"** in the onboarding stepper.
- **Vendor name propagates everywhere** — the legal-name and account-holder fields feed the vendor record, so the onboarding heading, penny-drop screen, and verification screen all show the entered name (no more hardcoded placeholder company).
- **Onboarding heading is live** — "Onboarding for New Vendor" updates to "Onboarding for &lt;name&gt;" as the vendor types.
- **Working uploads** — Bank proof document and a new **Cancelled cheque** field use a real file picker (shows the filename). *(The post-upload control is now a ✕ remove, not Replace — see the latest session above.)*
- **"Submit & run penny drop" flow** — submitting the Bank Details step shows a running/spinner state, then the three-way-match result.
- **Admin Console** (new `#/admin`, added to the left nav) — 6 KPI tiles, onboarding funnel, turnaround-time panel with a 48-hr target line, vendor tracking table with filters, an **Export CSV** button (downloads `vendor-tracking.csv`), and a recent-activity feed.

---

## 5. Key data models (in `state`)

Seed data is hardcoded; wire to real APIs when productionizing.

- **`state.vendors[]`** — `{ id, name, contact, email, phone, category, dept, status, risk, riskScore, onboarded, raisedBy, buying, docs[], bank, activity[], drivers{}, riskAnswers{}, declAnswers{} }`. `status ∈ active | onboarding | flagged | suspended`.
  - **`drivers{}`** — applicability flags, **vendor-declared** in the onboarding Declaration step; reviewer can adjust on `#/risk`.
  - **`declAnswers{}`** — the **vendor's** self-declaration answers (keyed by `DOMAINS` question ids). Context only, **not scored**.
  - **`riskAnswers{}`** — the **reviewer's** assessment answers (keyed by `REVIEWER_DOMAINS` question ids). This is the **scored** set.
- **`state.activity[]`** — `{ when, who, vendor, ev, system?, sub? }` audit entries.
- **Risk engine** — two questionnaires: `DOMAINS` (vendor declaration) and `REVIEWER_DOMAINS` (reviewer assessment, scored). `computeRisk()` scores the reviewer set: applicability → weighted scoring → knockouts, internally `Standard`/`Elevated`, displayed as **Low/High** via `riskLabel()`.
- **Admin Console sample data** (funnel, TAT, and three of the six KPI figures) is illustrative and lives inside `adminConsole()` — replace with API data as needed. `exportVendorsCSV()` exports all vendors.

---

## 6. How to edit & verify safely

Because it's one file with big template literals, a stray character can silently break a screen. Recommended loop:

1. **Syntax check:** extract the `<script>` block and run `node --check` on it (catches unclosed template literals, dangling refs).
2. **Headless smoke test:** with a minimal DOM shim, `eval` the script, drive `location.hash` + `render()`, call `window.*` handlers, and assert on `#root.innerHTML`.
3. **Open in a browser** to eyeball the result.

> Harness note: an outer/eval scope cannot read the `let state` binding — assert via rendered HTML or `location.hash`, not `state.*`.

---

## 7. Companion files in the repo (context, not required to run)

`shubham_Wireframe.md` (design source), `risk-framework_isha.md` (risk logic), `VMS_Solution_Spec_VP.md` (scope/journey), `vendor-dashboard-handoff.md` (Admin Console dashboard spec). These are background; the running prototype is fully contained in the source below.

---

## 8. Full source

Everything between the two markers below is the complete `vms_prototype.html`. Copy it verbatim into a file of that name.

**⟶ BEGIN `vms_prototype.html` ⟵**


````html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>VMS — Vendor Management System (Prototype)</title>
<style>
  :root{
    --bg:#f1f5f9; --surface:#ffffff; --surface-2:#f8fafc; --border:#e2e8f0;
    --border-strong:#cbd5e1; --ink:#0f172a; --ink-2:#475569; --ink-3:#94a3b8;
    --accent:#4f46e5; --accent-ink:#ffffff; --accent-soft:#eef2ff;
    --ok:#16a34a; --ok-soft:#dcfce7; --warn:#d97706; --warn-soft:#fef3c7;
    --danger:#dc2626; --danger-soft:#fee2e2;
    --radius:12px; --radius-sm:8px; --shadow:0 1px 2px rgba(15,23,42,.04),0 4px 16px rgba(15,23,42,.06);
    --shadow-lg:0 12px 40px rgba(15,23,42,.16);
    --sp:16px;
    --font: -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
  }
  *{box-sizing:border-box}
  html,body{margin:0;padding:0}
  html{scroll-behavior:smooth}
  body{font-family:var(--font);background:var(--bg);color:var(--ink);font-size:14px;line-height:1.5;-webkit-font-smoothing:antialiased}
  a{color:var(--accent);text-decoration:none}
  h1,h2,h3,h4{margin:0;font-weight:650;letter-spacing:-.01em}
  .app-wrap{min-height:100vh;display:flex;flex-direction:column}

  /* ---------- Buttons ---------- */
  .btn{display:inline-flex;align-items:center;gap:8px;justify-content:center;padding:9px 16px;border-radius:var(--radius-sm);
       border:1px solid transparent;background:var(--accent);color:var(--accent-ink);font:inherit;font-weight:600;cursor:pointer;
       transition:.15s;white-space:nowrap}
  .btn:hover{filter:brightness(1.06)}
  .btn:active{transform:translateY(1px)}
  .btn-secondary{background:var(--surface);color:var(--ink);border-color:var(--border-strong)}
  .btn-secondary:hover{background:var(--surface-2);filter:none}
  .btn-ghost{background:transparent;color:var(--ink-2);border-color:transparent}
  .btn-ghost:hover{background:var(--surface-2)}
  .btn-danger{background:var(--danger)}
  .btn-ok{background:var(--ok)}
  .btn-warn{background:var(--warn)}
  .btn:disabled{opacity:.45;cursor:not-allowed;filter:none;transform:none}
  .btn-sm{padding:6px 11px;font-size:13px}
  .btn-block{width:100%}

  /* ---------- Layout ---------- */
  .topbar{position:sticky;top:0;z-index:30;display:flex;align-items:center;gap:20px;height:58px;padding:0 22px;
          background:rgba(255,255,255,.85);backdrop-filter:blur(10px);border-bottom:1px solid var(--border)}
  .brand{font-weight:750;font-size:16px;letter-spacing:-.02em;color:var(--ink);display:flex;align-items:center;gap:8px}
  .brand .logo{width:26px;height:26px;border-radius:7px;background:linear-gradient(135deg,var(--accent),#7c74f0);display:inline-flex;
               align-items:center;justify-content:center;color:#fff;font-size:13px;font-weight:800}
  .nav{display:flex;gap:4px;margin-left:8px}
  .nav a{padding:7px 12px;border-radius:8px;color:var(--ink-2);font-weight:550}
  .nav a:hover{background:var(--surface-2);color:var(--ink)}
  .nav a.active{background:var(--accent-soft);color:var(--accent)}
  .spacer{flex:1}
  .userchip{display:flex;align-items:center;gap:9px;padding:5px 10px 5px 5px;border:1px solid var(--border);border-radius:999px;cursor:pointer;background:var(--surface)}
  .avatar{width:28px;height:28px;border-radius:50%;background:var(--accent);color:#fff;display:inline-flex;align-items:center;justify-content:center;font-size:12px;font-weight:700}
  .container{width:100%;max-width:1120px;margin:0 auto;padding:26px 22px 60px}
  .container-sm{max-width:560px}

  /* ---------- Cards ---------- */
  .card{background:var(--surface);border:1px solid var(--border);border-radius:var(--radius);box-shadow:var(--shadow)}
  .card-pad{padding:22px}
  .card + .card{margin-top:16px}
  .section-label{font-size:11px;font-weight:700;letter-spacing:.07em;text-transform:uppercase;color:var(--ink-3);margin-bottom:12px}
  .muted{color:var(--ink-2)}
  .meta{color:var(--ink-3);font-size:13px}
  .divider{height:1px;background:var(--border);margin:18px 0}
  .row{display:flex;gap:16px;flex-wrap:wrap}
  .grid{display:grid;gap:16px}
  .breadcrumb{color:var(--ink-2);font-weight:550;display:inline-flex;align-items:center;gap:6px;margin-bottom:14px;cursor:pointer}
  .breadcrumb:hover{color:var(--ink)}
  .page-head{display:flex;align-items:flex-start;justify-content:space-between;gap:16px;margin-bottom:20px;flex-wrap:wrap}
  .page-head h1{font-size:22px}

  /* ---------- Status ---------- */
  .dot{display:inline-block;width:9px;height:9px;border-radius:50%;margin-right:6px;vertical-align:middle}
  .dot.active{background:var(--ok)} .dot.flagged{background:var(--danger)} .dot.onboarding{background:var(--warn)}
  .dot.suspended{background:var(--ink-3)}
  .badge{display:inline-flex;align-items:center;gap:6px;padding:3px 10px;border-radius:999px;font-size:12px;font-weight:600;background:var(--surface-2);border:1px solid var(--border);color:var(--ink-2)}
  .badge.ok{background:var(--ok-soft);color:#15803d;border-color:transparent}
  .badge.warn{background:var(--warn-soft);color:#b45309;border-color:transparent}
  .badge.danger{background:var(--danger-soft);color:#b91c1c;border-color:transparent}
  .badge.accent{background:var(--accent-soft);color:var(--accent);border-color:transparent}
  .glyph{font-weight:700;display:inline-block;width:1.1em;text-align:center}
  .g-ok{color:var(--ok)} .g-warn{color:var(--warn)} .g-bad{color:var(--danger)} .g-todo{color:var(--ink-3)}

  /* ---------- Forms ---------- */
  .field{margin-bottom:14px}
  .field > label{display:block;font-weight:600;font-size:13px;margin-bottom:6px}
  .field .hint{color:var(--ink-3);font-size:12px;margin-top:5px}
  input[type=text],input[type=email],input[type=password],input[type=date],select,textarea{
    width:100%;padding:10px 12px;border:1px solid var(--border-strong);border-radius:var(--radius-sm);font:inherit;background:var(--surface);color:var(--ink)}
  input:focus,select:focus,textarea:focus{outline:2px solid var(--accent-soft);border-color:var(--accent)}
  textarea{resize:vertical;min-height:80px}
  .field-inline{display:flex;gap:10px}
  .radio-row{display:flex;gap:10px;flex-wrap:wrap}
  .radio-opt{display:flex;gap:8px;align-items:center;padding:9px 13px;border:1px solid var(--border-strong);border-radius:var(--radius-sm);cursor:pointer;flex:1;min-width:120px}
  .radio-opt.sel{border-color:var(--accent);background:var(--accent-soft)}
  .check{display:flex;gap:9px;align-items:flex-start;cursor:pointer}
  .grid-2{display:grid;grid-template-columns:1fr 1fr;gap:14px}
  @media(max-width:640px){.grid-2{grid-template-columns:1fr}}

  /* ---------- Chips / filters ---------- */
  .chips{display:flex;gap:8px;flex-wrap:wrap;align-items:center}
  .chip{display:inline-flex;align-items:center;gap:7px;padding:7px 13px;border:1px solid var(--border-strong);border-radius:999px;background:var(--surface);cursor:pointer;font-weight:550;color:var(--ink-2)}
  .chip:hover{background:var(--surface-2)}
  .chip.active{background:var(--accent);color:#fff;border-color:var(--accent)}
  .flabel{font-size:13px;color:var(--ink-3);font-weight:600}
  .flabel-on{color:var(--accent);font-weight:800}
  .search{position:relative;flex:1;min-width:220px}
  .search input{padding-left:38px}
  .search .ico{position:absolute;left:13px;top:50%;transform:translateY(-50%);color:var(--ink-3)}

  /* ---------- Table ---------- */
  .table-wrap{overflow-x:auto}
  table{width:100%;border-collapse:collapse;min-width:640px}
  th{font-size:11px;text-transform:uppercase;letter-spacing:.05em;color:var(--ink-3);text-align:left;padding:12px 16px;border-bottom:1px solid var(--border);font-weight:700}
  td{padding:13px 16px;border-bottom:1px solid var(--border);vertical-align:middle}
  tr.clickable{cursor:pointer}
  tr.clickable:hover td{background:var(--surface-2)}
  .rowmenu{position:relative}
  .rowmenu .dots{cursor:pointer;padding:4px 8px;border-radius:6px;color:var(--ink-2)}
  .rowmenu .dots:hover{background:var(--surface-2)}
  .menu{position:absolute;right:0;top:26px;background:var(--surface);border:1px solid var(--border);border-radius:10px;box-shadow:var(--shadow-lg);min-width:150px;overflow:hidden;z-index:20}
  .menu button{display:block;width:100%;text-align:left;padding:10px 14px;background:none;border:none;font:inherit;cursor:pointer;color:var(--ink)}
  .menu button:hover{background:var(--surface-2)}
  .menu button.danger{color:var(--danger)}

  /* ---------- Stepper ---------- */
  .stepper{display:flex;align-items:center;gap:0;margin:4px 0 6px}
  .step{display:flex;flex-direction:column;align-items:center;gap:7px;flex:1;position:relative}
  .step .bub{width:30px;height:30px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-weight:700;background:var(--surface);border:2px solid var(--border-strong);color:var(--ink-3);z-index:2}
  .step.done .bub{background:var(--ok);border-color:var(--ok);color:#fff}
  .step.current .bub{background:var(--accent);border-color:var(--accent);color:#fff}
  .step .lbl{font-size:12px;font-weight:600;color:var(--ink-2)}
  .step.current .lbl{color:var(--accent)}
  .step .bar{position:absolute;top:15px;left:50%;width:100%;height:2px;background:var(--border-strong);z-index:1}
  .step.done .bar{background:var(--ok)}
  .step:last-child .bar{display:none}
  .dots-progress{display:flex;gap:7px}
  .dots-progress i{width:9px;height:9px;border-radius:50%;background:var(--border-strong);display:inline-block}
  .dots-progress i.on{background:var(--accent)}

  /* ---------- Timeline ---------- */
  .timeline{list-style:none;margin:0;padding:0}
  .timeline li{position:relative;padding:0 0 16px 22px;border-left:2px solid var(--border);}
  .timeline li:last-child{border-color:transparent}
  .timeline li::before{content:"";position:absolute;left:-6px;top:3px;width:10px;height:10px;border-radius:50%;background:var(--accent)}
  .timeline li.system::before{background:var(--ink-3)}
  .timeline .t-when{font-size:12px;color:var(--ink-3)}

  /* ---------- Gauge ---------- */
  .gauge{width:150px;height:150px;border-radius:50%;display:flex;align-items:center;justify-content:center;position:relative}
  .gauge::before{content:"";position:absolute;inset:14px;background:var(--surface);border-radius:50%}
  .gauge .val{position:relative;text-align:center}
  .gauge .val b{font-size:34px;display:block;line-height:1}
  .gauge .val span{font-size:12px;color:var(--ink-3)}

  /* ---------- Misc panels ---------- */
  .split{display:grid;grid-template-columns:1.4fr 1fr;gap:16px}
  @media(max-width:820px){.split{grid-template-columns:1fr}}
  .placeholder{border:2px dashed var(--border-strong);border-radius:var(--radius);padding:40px 24px;text-align:center;color:var(--ink-2);background:repeating-linear-gradient(45deg,var(--surface-2),var(--surface-2) 10px,#fff 10px,#fff 20px)}
  .empty-state{text-align:center;padding:44px 24px}
  .empty-state .em{font-size:30px;margin-bottom:10px}
  .doc-line{display:flex;align-items:center;gap:12px;padding:11px 0;border-bottom:1px solid var(--border)}
  .doc-line:last-child{border:none}
  .doc-line .name{flex:1;font-weight:550}
  .file-x{border:none;background:none;cursor:pointer;color:#64748B;font-size:16px;line-height:1;padding:4px 7px;border-radius:6px}
  .file-x:hover{color:var(--danger);background:var(--danger-soft)}
  .kv{display:flex;justify-content:space-between;padding:7px 0;border-bottom:1px dashed var(--border)}
  .kv:last-child{border:none}
  .kv .k{color:var(--ink-2)}
  .spinner{width:34px;height:34px;margin:0 auto;border-radius:50%;border:3px solid var(--border);border-top-color:var(--accent);animation:spin .8s linear infinite}
  @keyframes spin{to{transform:rotate(360deg)}}
  .toast{position:fixed;bottom:22px;left:50%;transform:translateX(-50%) translateY(20px);background:var(--ink);color:#fff;padding:12px 20px;border-radius:10px;box-shadow:var(--shadow-lg);opacity:0;transition:.25s;z-index:100;font-weight:550}
  .toast.show{opacity:1;transform:translateX(-50%) translateY(0)}
  .pill-nav{display:flex;gap:6px;flex-wrap:wrap;margin-bottom:18px}
  .pill-nav a{padding:7px 14px;border-radius:999px;background:var(--surface);border:1px solid var(--border);color:var(--ink-2);font-weight:550;font-size:13px}
  .pill-nav a.active{background:var(--ink);color:#fff;border-color:var(--ink)}

  /* ---------- Landing (VendorIQ layout, indigo theme) ---------- */
  .lp-page{width:100%;background:var(--surface)}
  .lp-wrap{max-width:1280px;margin:0 auto}
  .lp-nav{position:sticky;top:0;z-index:30;display:flex;justify-content:space-between;align-items:center;padding:18px 80px;
          border-bottom:1px solid var(--border);background:rgba(255,255,255,.9);backdrop-filter:blur(10px)}
  .lp-nav-left{display:flex;align-items:center;gap:40px}
  .lp-logo{display:flex;align-items:center;gap:8px}
  .lp-logo-mark{width:auto;height:auto;padding:5px 11px;border-radius:9px;background:linear-gradient(135deg,#6366f1 0%,#3730a3 71%);
                display:inline-flex;align-items:center;justify-content:center;color:#fff;font-weight:800;font-size:13px;letter-spacing:-.2px;white-space:nowrap;
                text-shadow:0 1px 1px rgba(0,0,0,.45),0 -1px 0 rgba(255,255,255,.22);
                box-shadow:inset 0 1px 0 rgba(255,255,255,.28),inset 0 -2px 3px rgba(0,0,0,.28),0 1px 2px rgba(0,0,0,.18)}
  .lp-logo-text{font-weight:700;font-size:20px;color:var(--ink)}
  .lp-links{display:flex;gap:24px}
  .lp-links a{font-size:14px;font-weight:500;color:#334155}
  .lp-links a.active{color:var(--accent);font-weight:600}
  .lp-links a:hover{color:var(--accent)}
  .lp-nav-right{display:flex;align-items:center;gap:16px}
  .lp-login{font-size:14px;font-weight:600;color:#334155;cursor:pointer}
  .lp-login:hover{color:var(--accent)}
  .lp-hero{display:flex;flex-direction:column;align-items:center;gap:24px;padding:80px 24px;background:var(--surface-2);text-align:center}
  .lp-badge{background:var(--accent-soft);color:var(--accent);border-radius:20px;padding:6px 16px;font-size:12px;font-weight:600;letter-spacing:.02em}
  .lp-hero h1{font-size:56px;line-height:1.15;font-weight:800;color:var(--ink);max-width:820px;letter-spacing:-.02em}
  .lp-hero .sub{font-size:18px;line-height:1.5;color:var(--ink-2);max-width:600px}
  .lp-ctas{display:flex;gap:16px;flex-wrap:wrap;justify-content:center}
  .lp-ctas .btn{padding:14px 28px;font-size:16px;border-radius:6px}
  .lp-section{padding:70px 80px}
  .lp-section.alt{background:var(--surface-2)}
  .lp-sec-head{display:flex;flex-direction:column;align-items:center;gap:8px;text-align:center;margin-bottom:44px}
  .lp-sec-head h2{font-size:32px;font-weight:700;color:var(--ink)}
  .lp-sec-head p{font-size:16px;color:var(--ink-2)}
  .lp-steps{display:grid;grid-template-columns:repeat(4,1fr);gap:24px}
  .lp-step{background:var(--surface-2);border-radius:12px;padding:24px;display:flex;flex-direction:column;gap:16px}
  .lp-section.alt .lp-step{background:var(--surface)}
  .lp-step-badge{width:36px;height:36px;border-radius:50%;background:var(--accent);color:#fff;font-weight:700;font-size:16px;display:flex;align-items:center;justify-content:center}
  .lp-step h3{font-size:18px;color:var(--ink)}
  .lp-step p{font-size:14px;line-height:1.4;color:var(--ink-2)}
  .lp-checklist{background:var(--accent-soft);border-radius:16px;padding:40px;display:flex;gap:40px;align-items:center}
  .lp-check-info{flex:1;min-width:280px}
  .lp-check-info h2{font-size:28px;font-weight:700;color:var(--accent);margin-bottom:14px}
  .lp-check-info p{font-size:15px;line-height:1.5;color:#334155}
  .lp-check-card{flex:0 0 560px;max-width:100%;background:var(--surface);border-radius:12px;padding:32px;box-shadow:0 4px 12px rgba(15,23,42,.10);display:flex;flex-direction:column;gap:16px}
  .lp-check-item{display:flex;gap:12px;align-items:flex-start;font-size:14px;line-height:1.4;color:#334155}
  .lp-check-item .ck{flex:none;width:20px;height:20px;border-radius:50%;border:2px solid var(--accent);color:var(--accent);display:flex;align-items:center;justify-content:center;font-size:11px;font-weight:800;margin-top:1px}
  .lp-about{display:flex;gap:80px;align-items:flex-start}
  .lp-about-col{flex:1;min-width:280px}
  .lp-eyebrow{font-size:12px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;color:var(--accent);margin-bottom:14px}
  .lp-about-col h2{font-size:32px;font-weight:700;color:var(--ink);margin-bottom:14px}
  .lp-about-col h3{font-size:20px;font-weight:700;color:var(--ink);margin-bottom:22px}
  .lp-about-col > p{font-size:15px;line-height:1.6;color:var(--ink-2)}
  .lp-flist{display:flex;flex-direction:column;gap:18px}
  .lp-flist .ft b{display:block;font-size:16px;color:var(--ink);margin-bottom:4px}
  .lp-flist .ft span{font-size:14px;line-height:1.4;color:var(--ink-2)}
  .lp-faq-list{max-width:800px;margin:0 auto;display:flex;flex-direction:column;gap:16px}
  details.lp-faq{background:var(--surface);border:1px solid var(--border);border-radius:8px;padding:0 24px}
  details.lp-faq summary{list-style:none;cursor:pointer;display:flex;justify-content:space-between;align-items:center;gap:16px;padding:22px 0;font-size:16px;font-weight:600;color:var(--ink)}
  details.lp-faq summary::-webkit-details-marker{display:none}
  details.lp-faq summary::after{content:"⌄";color:var(--ink-3);font-size:18px;transition:.2s}
  details.lp-faq[open] summary::after{transform:rotate(180deg)}
  details.lp-faq .ans{font-size:14px;line-height:1.55;color:#334155;padding:0 0 22px}
  .lp-footer{display:flex;justify-content:space-between;align-items:center;padding:30px 80px;border-top:1px solid var(--border);flex-wrap:wrap;gap:16px}
  .lp-footer, .lp-footer a{font-size:14px;color:var(--ink-2)}
  .lp-footer-links{display:flex;gap:24px}
  @media(max-width:960px){
    .lp-nav,.lp-section,.lp-footer{padding-left:32px;padding-right:32px}
    .lp-steps{grid-template-columns:repeat(2,1fr)}
    .lp-checklist,.lp-about{flex-direction:column;align-items:stretch}
    .lp-check-card{flex-basis:auto}
    .lp-hero h1{font-size:40px}
  }
  @media(max-width:600px){
    .lp-links{display:none}
    .lp-steps{grid-template-columns:1fr}
    .lp-hero h1{font-size:32px}
    .lp-nav,.lp-section,.lp-footer{padding-left:20px;padding-right:20px}
  }
  .center{text-align:center}
  .auth-wrap{min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px;background:radial-gradient(1200px 600px at 50% -10%,#e7e9fb,transparent),var(--bg)}
  .auth-card{width:100%;max-width:420px}
  .center-brand{display:flex;flex-direction:column;align-items:center;gap:10px;margin-bottom:24px}
  .center-brand .logo{width:44px;height:44px;font-size:20px}
  /* Google SSO (simulated) */
  .btn-google{display:flex;align-items:center;justify-content:center;gap:10px;width:100%;padding:11px 16px;border:1px solid var(--border-strong);border-radius:8px;background:#fff;color:#3c4043;font:inherit;font-weight:600;font-size:14px;cursor:pointer}
  .btn-google:hover{background:#f8faff;box-shadow:0 1px 3px rgba(60,64,67,.18)}
  .btn-google svg{width:18px;height:18px;flex:none}
  .g-card{max-width:450px;background:#fff;border:1px solid #dadce0;border-radius:12px;padding:44px 40px}
  .g-acct{display:flex;align-items:center;gap:14px;width:100%;text-align:left;padding:12px 12px;border:1px solid var(--border);border-radius:10px;background:#fff;cursor:pointer;font:inherit;margin-bottom:10px}
  .g-acct:hover{background:var(--surface-2);border-color:var(--border-strong)}
  .g-acct .avatar{background:var(--accent);flex:none}
  .g-acct-ic{width:32px;height:32px;border-radius:50%;border:1px dashed var(--border-strong);display:flex;align-items:center;justify-content:center;color:var(--ink-3);font-size:20px;flex:none}
  /* Modal (demo request) */
  .modal-overlay{position:fixed;inset:0;background:rgba(15,23,42,.5);display:flex;align-items:center;justify-content:center;padding:20px;z-index:200}
  .modal{background:var(--surface);border-radius:16px;box-shadow:var(--shadow-lg);width:100%;max-width:440px;padding:28px}
  .modal h3{font-size:20px;margin-bottom:4px}
  .modal .modal-sub{color:var(--ink-2);font-size:14px;margin-bottom:18px}
  .modal .consent{color:var(--ink-3);font-size:12px;line-height:1.5;margin-top:14px}
  .modal-actions{display:flex;gap:10px;justify-content:flex-end;margin-top:20px}
  .modal .err{color:var(--danger);font-size:13px;margin-top:10px}
  .modal-success{text-align:center;padding:8px 4px}
  /* Client dashboard shell (sidebar) */
  .cshell{display:flex;min-height:100vh}
  .csidebar{width:260px;flex:none;background:#0F172A;color:#fff;display:flex;flex-direction:column;justify-content:space-between;padding:24px;position:sticky;top:0;height:100vh}
  .cbrand{display:flex;align-items:center;gap:12px;margin-bottom:34px}
  .cbrand .mk{width:32px;height:32px;border-radius:8px;background:var(--accent);display:flex;align-items:center;justify-content:center;font-size:15px}
  .cbrand b{font-size:18px;font-weight:800;display:block;line-height:1.2}
  .cbrand span{font-size:10px;letter-spacing:.06em;text-transform:uppercase;color:#64748B}
  .cnav{display:flex;flex-direction:column;gap:6px}
  .cnav a{display:flex;align-items:center;padding:12px 16px;border-radius:8px;color:#94A3B8;font-weight:500;font-size:13px;cursor:pointer}
  .cnav a:hover{background:#1E293B;color:#fff}
  .cnav a.active{background:#1E293B;color:#fff;font-weight:600}
  .cbottom{border-top:1px solid #1E293B;padding-top:20px}
  .cprofile{display:flex;align-items:center;gap:12px;margin-bottom:14px}
  .cprofile .av{width:40px;height:40px;border-radius:50%;background:var(--accent);color:#fff;display:flex;align-items:center;justify-content:center;font-weight:700;font-size:13px}
  .cprofile b{font-size:13px;display:block;color:#fff}
  .cprofile span{font-size:11px;color:#64748B}
  .csignout{display:flex;align-items:center;color:#94A3B8;font-size:12px;cursor:pointer;background:none;border:none;padding:4px 0;font:inherit;font-size:12px}
  .csignout:hover{color:#fff}
  .cmain{flex:1;min-width:0;padding:40px;background:var(--surface-2)}
  .kpi-row{display:grid;grid-template-columns:repeat(3,1fr);gap:20px;margin-bottom:28px}
  .kpi{display:flex;align-items:center;gap:16px;background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:20px}
  .kpi .ic{width:40px;height:40px;border-radius:8px;display:flex;align-items:center;justify-content:center;font-size:18px;font-weight:700;flex:none}
  .kpi b{font-size:24px;display:block;line-height:1.1;color:var(--ink)}
  .kpi span{font-size:12px;color:var(--ink-2)}
  @media(max-width:900px){.csidebar{display:none}.kpi-row{grid-template-columns:1fr}.cmain{padding:24px}}
  /* Admin console — KPI 6-up + funnel / TAT bars */
  .kpi-row.six{grid-template-columns:repeat(3,1fr)}
  @media(max-width:1100px){.kpi-row.six{grid-template-columns:repeat(2,1fr)}}
  .afunnel{display:flex;flex-direction:column;gap:14px}
  .afrow{display:grid;grid-template-columns:130px 1fr 78px;align-items:center;gap:12px}
  .afrow .flab{font-size:12px;color:var(--ink-2);font-weight:600}
  .abar-wrap{position:relative;background:var(--surface-2);border-radius:8px;height:28px;overflow:hidden}
  .abar{height:100%;border-radius:8px;background:var(--accent);display:flex;align-items:center;padding-left:10px;color:#fff;font-size:12px;font-weight:700;min-width:38px;transition:.3s}
  .abar.good{background:var(--ok)}
  .afrow .fmeta{font-size:12px;color:var(--ink-3);text-align:right}
  .atarget{position:absolute;top:-2px;bottom:-2px;width:2px;background:var(--ok);z-index:2}
  .acols{display:grid;grid-template-columns:1.5fr 1fr;gap:16px;align-items:start}
  @media(max-width:1080px){.acols{grid-template-columns:1fr}}
</style>
</head>
<body>
<div class="app-wrap" id="root"></div>
<div class="toast" id="toast"></div>

<script>
/* ============================================================
   VMS Prototype — single file, in-memory, hash-routed SPA
   Built from shubham_Wireframe.md (design source of truth).
   ============================================================ */

/* ---------------- State ---------------- */
const STATUS = {active:{lbl:'Active',cls:'active'},flagged:{lbl:'Flagged',cls:'flagged'},
  onboarding:{lbl:'Onboarding',cls:'onboarding'},suspended:{lbl:'Suspended',cls:'suspended'}};

let state = {
  currentUser: {name:'S. Borse', initials:'SB', role:'Admin', dept:'Product', cc:'CC-4471', type:'admin'},
  filters: {search:'', status:'', dept:'', risk:''}, showMoreFilters:false,
  signupStep: 1,
  vendors: [
    { id:'acme', name:'Acme Ltd', contact:'R. Mehta', email:'r.mehta@acme.in', phone:'+91 98xxx xxxxx',
      category:'IT', dept:'IT', status:'active', risk:'Low', riskScore:62, onboarded:'12 Mar 2026',
      raisedBy:'S. Borse', buying:'Cloud infrastructure & managed IT support',
      docs:[
        {name:'PAN card', st:'ok', note:'no expiry'},
        {name:'GST certificate', st:'warn', note:'expires in 14 days'},
        {name:'Udyam certificate', st:'ok', note:'expires 04 Jan 2027'},
        {name:'Address proof', st:'bad', note:'MISSING'},
        {name:'Signed vendor declaration', st:'ok', note:'no expiry'}
      ],
      bank:{acct:'●●●●●●●●4821', ifsc:'HDFC0001234', rails:'NEFT / RTGS', terms:'Net 30'},
      activity:[
        {when:'12 Mar', who:'S. Borse', ev:'Activated'},
        {when:'11 Mar', who:'A. Rao', ev:'Approved — Finance'},
        {when:'10 Mar', who:'M. Khan', ev:'Approved — Business owner'},
        {when:'08 Mar', who:'system', ev:'Bank verified · penny drop passed', system:true}
      ]},
    { id:'borse', name:'Borse Co', contact:'S. Iyer', email:'s.iyer@borse.co', phone:'+91 90xxx xxxxx',
      category:'Ops', dept:'Ops', status:'flagged', risk:'High', riskScore:78, onboarded:'—', raisedBy:'N. Kulkarni',
      buying:'Logistics & warehousing', docs:[], bank:null, activity:[] },
    { id:'cygnet', name:'Cygnet Pvt', contact:'A. Rao', email:'a.rao@cygnet.in', phone:'+91 99xxx xxxxx',
      category:'Legal', dept:'Legal', status:'onboarding', risk:'—', riskScore:null, onboarded:'—', raisedBy:'S. Borse',
      buying:'Legal advisory retainer', docs:[], bank:null, activity:[], stage:'onboarding' },
    { id:'delta', name:'Delta Serv', contact:'M. Khan', email:'m.khan@delta.in', phone:'+91 97xxx xxxxx',
      category:'Facilities', dept:'Facilities', status:'active', risk:'Low', riskScore:34, onboarded:'02 Feb 2026',
      raisedBy:'V. Deshpande', buying:'Facilities management', docs:[], bank:null, activity:[] },
    { id:'nimbus', name:'Nimbus Cloud', contact:'P. Nair', email:'p.nair@nimbus.io', phone:'+91 98xxx xxxxx',
      category:'IT', dept:'IT', status:'active', onboarded:'20 Jan 2026', raisedBy:'S. Borse',
      buying:'Cloud hosting & storage', docs:[], bank:null, activity:[], drivers:{dataAccess:true} },
    { id:'orion', name:'Orion Logistics', contact:'K. Das', email:'k.das@orion.in', phone:'+91 90xxx xxxxx',
      category:'Ops', dept:'Ops', status:'active', onboarded:'15 Jan 2026', raisedBy:'N. Kulkarni',
      buying:'Freight & last-mile delivery', docs:[], bank:null, activity:[], drivers:{poshApplies:true} },
    { id:'sterling', name:'Sterling Legal', contact:'J. Fernandes', email:'j.fernandes@sterling.in', phone:'+91 99xxx xxxxx',
      category:'Legal', dept:'Legal', status:'flagged', onboarded:'—', raisedBy:'P. Joshi',
      buying:'Litigation support', docs:[], bank:null, activity:[], drivers:{bankMismatch:true} },
    { id:'meridian', name:'Meridian Consulting', contact:'T. Bose', email:'t.bose@meridian.co', phone:'+91 97xxx xxxxx',
      category:'Consulting', dept:'Strategy', status:'onboarding', onboarded:'—', raisedBy:'S. Borse',
      buying:'Management consulting', docs:[], bank:null, activity:[],
      drivers:{dataAccess:true}, declAnswers:{labor_s2:false, dpdp_s2:false, dpdp_s3:false} },
    { id:'pinnacle', name:'Pinnacle Facilities', contact:'R. Menon', email:'r.menon@pinnacle.in', phone:'+91 96xxx xxxxx',
      category:'Facilities', dept:'Facilities', status:'active', onboarded:'08 Jan 2026', raisedBy:'R. Kamat',
      buying:'Housekeeping & maintenance', docs:[], bank:null, activity:[], drivers:{poshApplies:true} },
    { id:'quantum', name:'Quantum Analytics', contact:'S. Reddy', email:'s.reddy@quantum.ai', phone:'+91 95xxx xxxxx',
      category:'IT', dept:'IT', status:'active', onboarded:'28 Dec 2025', raisedBy:'S. Borse',
      buying:'Data analytics platform', docs:[], bank:null, activity:[], drivers:{dataAccess:true, pmlaApplies:true} },
    { id:'vertex', name:'Vertex Pharma', contact:'D. Kapoor', email:'d.kapoor@vertex.in', phone:'+91 94xxx xxxxx',
      category:'Healthcare', dept:'R&D', status:'flagged', onboarded:'—', raisedBy:'P. Joshi',
      buying:'Clinical research services', docs:[], bank:null, activity:[],
      drivers:{dataAccess:true}, declAnswers:{dpdp_c1:false} },
    { id:'horizon', name:'Horizon Foods', contact:'A. Shah', email:'a.shah@horizon.in', phone:'+91 93xxx xxxxx',
      category:'Procurement', dept:'Procurement', status:'active', onboarded:'12 Dec 2025', raisedBy:'V. Deshpande',
      buying:'Catering & pantry supplies', docs:[], bank:null, activity:[] },
    { id:'apex', name:'Apex Security', contact:'M. Verma', email:'m.verma@apex.io', phone:'+91 92xxx xxxxx',
      category:'Security', dept:'Admin', status:'onboarding', onboarded:'—', raisedBy:'N. Kulkarni',
      buying:'Physical security & guarding', docs:[], bank:null, activity:[], drivers:{poshApplies:true} }
  ],
  activity: [
    {when:'12 Mar 14:02', who:'S. Borse', vendor:'Acme Ltd', ev:'Activated'},
    {when:'11 Mar 16:40', who:'A. Rao', vendor:'Acme Ltd', ev:'Approved — Finance'},
    {when:'10 Mar 09:15', who:'M. Khan', vendor:'Acme Ltd', ev:'Approved — Bus. owner'},
    {when:'08 Mar 11:23', who:'system', vendor:'Acme Ltd', ev:'Penny drop passed', system:true},
    {when:'08 Mar 11:20', who:'vendor', vendor:'Acme Ltd', ev:'Bank details submitted'},
    {when:'07 Mar 15:02', who:'S. Iyer', vendor:'Borse Co', ev:'Completed on behalf', sub:'reason: vendor has no email access'},
    {when:'06 Mar 10:44', who:'system', vendor:'Cygnet Pvt', ev:'Expired — no response', system:true},
    {when:'05 Mar 09:30', who:'R. Mehta', vendor:'Delta Serv', ev:'Withdrawn'},
    {when:'04 Mar 16:11', who:'system', vendor:'Acme Ltd', ev:'Duplicate check — none', system:true}
  ],
  // transient per-flow UI state
  onbStep: 0, pennyState:'', approvalFinanceDone:false, docChecks:{}, bankUploads:{}, orgSetup:{risk:false, security:false, decl:false},
  approvals: {}, vendorLogin:{email:'', otpSent:false},
};

/* Known users — drives the "not a registered Client User" identification/flag.
   type: 'vendor' (OTP login) · 'client' (email+password) · 'admin' (Google SSO). */
const KNOWN_USERS = {
  'buyer@company.com':'client', 'a.rao@company.com':'client', 's.iyer@company.com':'client',
  's.borse@company.com':'admin',
  'r.mehta@acme.in':'vendor', 's.iyer@borse.co':'vendor', 'a.rao@cygnet.in':'vendor', 'm.khan@delta.in':'vendor',
};
function userType(email){ return KNOWN_USERS[(email||'').trim().toLowerCase()] || null; }
const isEmail = e => /^[^@\s]+@[^@\s]+\.[^@\s]+$/.test((e||'').trim());

/* ============================================================
   Risk engine — Isha's framework (risk-framework_isha.md)
   Stage 1 applicability → Stage 2 weighted scoring → Stage 3 knockouts
   Result: Standard | Elevated, plus reviewing departments.
   ============================================================ */
const DOMAINS = {
  labor: { name:'Labor Laws', weight:20, appliesText:'Always — every engagement involves labour',
    applies:()=>true,
    critical:[
      {id:'labor_c1', q:'Workers covered by statutory benefits (PF / ESI) where applicable?'},
      {id:'labor_c2', q:'Hold all required labour registrations and licences?'}],
    supporting:[
      {id:'labor_s1', q:'Maintain statutory registers and wage records?'},
      {id:'labor_s2', q:'No labour dispute, prosecution, or penalty in the last 3 years?'},
      {id:'labor_s3', q:'Carry out periodic labour law compliance audits?'}] },
  dpdp: { name:'DPDP Data Protection', weight:25, appliesText:'Vendor accesses / processes / stores personal data',
    applies:v=>!!v.drivers.dataAccess,
    critical:[
      {id:'dpdp_c1', q:'Documented data protection policy aligned to the DPDP Act?'},
      {id:'dpdp_c2', q:'Appointed a Data Protection Officer or equivalent grievance officer?'},
      {id:'dpdp_c3', q:'Documented personal data breach notification process?'}],
    supporting:[
      {id:'dpdp_s1', q:'Personal data encrypted at rest and in transit?'},
      {id:'dpdp_s2', q:'Run data protection training at least annually?'},
      {id:'dpdp_s3', q:'Sub-processors contractually bound to equivalent obligations?'}] },
  posh: { name:'POSH Act compliance', weight:15, appliesText:'On-site staff and 10+ employees',
    applies:v=>!!v.drivers.poshApplies,
    critical:[
      {id:'posh_c1', q:'Constituted Internal Complaints Committee (ICC)?'},
      {id:'posh_c2', q:'Documented POSH policy communicated to all employees?'}],
    supporting:[
      {id:'posh_s1', q:'Run POSH awareness training at least annually?'},
      {id:'posh_s2', q:'Filed the annual POSH report for the most recent year?'}] },
  pmla: { name:'Anti-bribery / PMLA', weight:25, appliesText:'High-value spend or government-adjacent',
    applies:v=>!!v.drivers.pmlaApplies,
    critical:[
      {id:'pmla_c1', q:'Documented anti-bribery and anti-corruption policy?'},
      {id:'pmla_c2', q:'Beneficial ownership fully disclosed?'},
      {id:'pmla_c3', q:'Entity, directors, and owners NOT on any sanctions / debarment / adverse-media list?'}],
    supporting:[
      {id:'pmla_s1', q:'Screen your own third parties and intermediaries for bribery risk?'},
      {id:'pmla_s2', q:'Operate a whistleblower or reporting mechanism?'}] },
  rbi: { name:'RBI Know-Your-Vendor', weight:15, appliesText:'Client is a BFSI entity',
    applies:v=>!!v.drivers.clientBFSI,
    critical:[
      {id:'rbi_c1', q:"Accept RBI's right-to-audit and inspection clauses (incl. by the regulator directly)?"},
      {id:'rbi_c2', q:'All regulated data will remain within India?'}],
    supporting:[
      {id:'rbi_s1', q:'Maintain a BCP / DR plan tested within the last 12 months?'},
      {id:'rbi_s2', q:'Hold a current ISO 27001, SOC 2, or equivalent certification?'}] },
};

/* Reviewer-assessment questionnaire — a DIFFERENT set from the vendor declaration
   (DOMAINS above). Same domain keys / weights / applicability so the risk engine is
   reused unchanged; only the wording differs (reviewer verifies/judges rather than the
   vendor self-attesting). Reviewer answers → v.riskAnswers → scored by computeRisk(). */
const REVIEWER_DOMAINS = {
  labor: { name:'Labor Laws', weight:20, appliesText:DOMAINS.labor.appliesText, applies:DOMAINS.labor.applies,
    critical:[
      {id:'r_labor_c1', q:'Submitted labour registrations / licences verified as valid and current?'},
      {id:'r_labor_c2', q:'PF / ESI coverage evidenced for the workforce where applicable?'}],
    supporting:[
      {id:'r_labor_s1', q:'Statutory registers and wage records provided and internally consistent?'},
      {id:'r_labor_s2', q:'No-dispute declaration corroborated by adverse-media / watchlist checks?'}] },
  dpdp: { name:'DPDP Data Protection', weight:25, appliesText:DOMAINS.dpdp.appliesText, applies:DOMAINS.dpdp.applies,
    critical:[
      {id:'r_dpdp_c1', q:'Submitted data-protection policy adequate and DPDP-aligned on review?'},
      {id:'r_dpdp_c2', q:'Named DPO / grievance officer confirmed contactable?'}],
    supporting:[
      {id:'r_dpdp_s1', q:'Evidence of encryption at rest and in transit satisfactory?'},
      {id:'r_dpdp_s2', q:'Sub-processor contractual flow-downs evidenced?'}] },
  posh: { name:'POSH Act compliance', weight:15, appliesText:DOMAINS.posh.appliesText, applies:DOMAINS.posh.applies,
    critical:[
      {id:'r_posh_c1', q:'Constituted Internal Complaints Committee evidenced (members / notification)?'}],
    supporting:[
      {id:'r_posh_s1', q:'Latest annual POSH return or training evidence provided?'}] },
  pmla: { name:'Anti-bribery / PMLA', weight:25, appliesText:DOMAINS.pmla.appliesText, applies:DOMAINS.pmla.applies,
    critical:[
      {id:'r_pmla_c1', q:'Sanctions / adverse-media screening returns clear?'},
      {id:'r_pmla_c2', q:'Beneficial ownership disclosed and plausible?'}],
    supporting:[
      {id:'r_pmla_s1', q:'Anti-bribery / anti-corruption policy provided and credible?'}] },
  rbi: { name:'RBI Know-Your-Vendor', weight:15, appliesText:DOMAINS.rbi.appliesText, applies:DOMAINS.rbi.applies,
    critical:[
      {id:'r_rbi_c1', q:'Right-to-audit and data-localisation commitments acceptable?'}],
    supporting:[
      {id:'r_rbi_s1', q:'Current ISO 27001 / SOC 2 (or equivalent) certificate provided and valid?'}] },
};

// Applicability drivers + questionnaire answers seeded per sample vendor.
// `drivers` = applicability (vendor-declared). `answers` = REVIEWER assessment answers
// (REVIEWER_DOMAINS ids) → seeded into v.riskAnswers, the scored set. `decl` = vendor
// declaration answers (DOMAINS ids) → seeded into v.declAnswers, context only.
const RISK_SEED = {
  acme:   { drivers:{dataAccess:true}, answers:{} },                                      // Labor + DPDP, all pass → Standard
  borse:  { drivers:{poshApplies:true, pmlaApplies:true, bankMismatch:true}, answers:{} },// K2 fires → Elevated
  cygnet: { drivers:{dataAccess:true}, answers:{r_labor_s2:false, r_dpdp_s2:false} },     // reviewer-scored Elevated
  delta:  { drivers:{poshApplies:true}, answers:{} },                                     // Labor + POSH → Standard
};
state.vendors.forEach(v=>{
  const seed = RISK_SEED[v.id] || {};
  v.drivers = Object.assign({dataAccess:false, poshApplies:false, pmlaApplies:false, clientBFSI:false, bankMismatch:false}, v.drivers||{}, seed.drivers||{});
  v.riskAnswers = Object.assign({}, v.riskAnswers||{}, seed.answers||{}); // reviewer assessment (scored)
  v.declAnswers = Object.assign({}, v.declAnswers||{}, seed.decl||{});    // vendor declaration (context)
});

const SENIORITY = { Standard:['Analyst'], Elevated:['Manager','Senior Manager'] };

// Scored on the REVIEWER's assessment answers (v.riskAnswers vs REVIEWER_DOMAINS).
// The vendor's declaration (v.declAnswers vs DOMAINS) is context only, not scored.
function computeRisk(v){
  const ans = v.riskAnswers || {};
  const applicable = Object.entries(REVIEWER_DOMAINS).filter(([,d])=>d.applies(v));
  const sumW = applicable.reduce((s,[,d])=>s+d.weight,0) || 1;
  const domains = applicable.map(([key,d])=>{
    const nw = d.weight/sumW;
    const passed = d.supporting.filter(q=>ans[q.id]!==false).length;
    const score = d.supporting.length ? passed/d.supporting.length*100 : 100;
    const critFails = d.critical.filter(q=>ans[q.id]===false);
    return {key, name:d.name, weight:d.weight, nw, score, passed, total:d.supporting.length, critFails, critical:d.critical, supporting:d.supporting};
  });
  const overall = Math.round(domains.reduce((s,dm)=>s+dm.score*dm.nw,0));
  const knockouts = [];
  domains.forEach(dm=>dm.critFails.forEach(q=>knockouts.push({code:'K1', text:`${dm.name} — critical control failed: ${q.q}`})));
  if(v.drivers.bankMismatch) knockouts.push({code:'K2', text:'Bank country ≠ registered vendor country'});
  const level = (knockouts.length || overall<75) ? 'Elevated' : 'Standard';
  const depts = new Set(['Finance','Legal']);
  if(domains.some(d=>d.key==='dpdp'||d.key==='rbi')) depts.add('IT');
  if(domains.some(d=>d.key==='posh')) depts.add('HR');
  const nonApplicable = Object.entries(REVIEWER_DOMAINS).filter(([,d])=>!d.applies(v)).map(([,d])=>({name:d.name, reason:d.appliesText}));
  return {domains, overall, level, knockouts, departments:[...depts], nonApplicable};
}

/* ---------------- Helpers ---------------- */
const $ = sel => document.querySelector(sel);
const root = () => document.getElementById('root');
const esc = s => String(s==null?'':s).replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
function nav(hash){ location.hash = hash; }
function vendor(id){ return state.vendors.find(v=>v.id===id); }
function toast(msg){ const t=$('#toast'); t.textContent=msg; t.classList.add('show'); clearTimeout(t._h); t._h=setTimeout(()=>t.classList.remove('show'),2200); }
function statusBadge(st){ const s=STATUS[st]; return `<span class="badge"><span class="dot ${s.cls}"></span>${s.lbl}</span>`; }
function glyph(st){ const m={ok:['✓','g-ok'],warn:['⚠','g-warn'],bad:['✕','g-bad'],todo:['○','g-todo']}; const g=m[st]||m.todo; return `<span class="glyph ${g[1]}">${g[0]}</span>`; }
function logActivity(vendorName, ev){ const now=new Date(); const when='Now'; state.activity.unshift({when, who:state.currentUser.name, vendor:vendorName, ev}); }
function googleG(){ return `<svg viewBox="0 0 48 48" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
  <path fill="#EA4335" d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"/>
  <path fill="#4285F4" d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"/>
  <path fill="#FBBC05" d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"/>
  <path fill="#34A853" d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"/></svg>`; }

/* ---------------- App shell ---------------- */
function shell(active, body){
  return `
  <header class="topbar">
    <div class="brand">VMS</div>
    <nav class="nav">
      <a href="#/vendors" class="${active==='vendors'?'active':''}">Vendors</a>
      <a href="#/request" class="${active==='requests'?'active':''}">Requests</a>
      <a href="#/activity" class="${active==='activity'?'active':''}">Activity</a>
    </nav>
    <div class="spacer"></div>
    <div class="userchip" onclick="toast('Role switcher is a deferred screen (multi-role users).')">
      <span class="avatar">${esc(state.currentUser.initials)}</span>
      <span style="font-weight:600">${esc(state.currentUser.name)}</span>
      <span class="meta">▾</span>
    </div>
  </header>
  <main class="container">${body}</main>`;
}

/* ================= SCREENS ================= */

/* ---- Stage 0: Landing (VendorIQ vendor-registration page) ---- */
function landing(){
  const steps=[
    ['1','Register','Create an account with your company legal name and email.'],
    ['2','Complete Profile','Provide tax, business details, and country of operation.'],
    ['3','Upload Documents','Submit verification certificates and bank letters.'],
    ['4','Get Approved','Our internal compliance team reviews and approves in 5–7 days.']
  ].map(s=>`<div class="lp-step"><div class="lp-step-badge">${s[0]}</div>
      <div><h3>${s[1]}</h3><p style="margin-top:6px">${s[2]}</p></div></div>`).join('');
  const checklist=[
    'Business registration certificate (e.g., CoC, LLC Filing)',
    'Tax registration / GST / VAT certificate (as applicable locally)',
    'Bank account details + cancelled cheque or official bank letter',
    'Insurance certificate (Comprehensive General Liability, if applicable)',
    'ISO / ESG / Compliance certificates (optional but recommended)'
  ].map(c=>`<div class="lp-check-item"><span class="ck">✓</span><span>${c}</span></div>`).join('');
  const features=[
    ['Automated Document Tracking','Receive alerts before your insurance or tax certificates expire, avoiding vendor pauses.'],
    ['Secure Bank Verification','Direct verification pathways ensure secure payment processes and reduce B2B fraud.'],
    ['Universal Standards Compliance','Compliance parameters automatically adjust dynamically to match local country requirements.']
  ].map(f=>`<div class="ft"><b>${f[0]}</b><span>${f[1]}</span></div>`).join('');
  const faqs=[
    ['How long does the verification process take?','Typically, our internal compliance team completes the full audit within 5–7 business days after all mandatory documents are submitted successfully.'],
    ['Can I edit my submission once it is under review?','No, once submitted, files are locked during review. If our audit team finds any errors, you will receive a notification to revise that specific document.'],
    ['Is our banking and corporate data secure?','Absolutely. All banking information and official filings are encrypted in transit and at rest using enterprise-grade bank-level security measures.']
  ].map((q,i)=>`<details class="lp-faq" ${i===0?'open':''}><summary>${q[0]}</summary><div class="ans">${q[1]}</div></details>`).join('');

  return `<div class="lp-page">
    <nav class="lp-nav">
      <div class="lp-nav-left">
        <div class="lp-logo"><span class="lp-logo-mark">n̄.Ai+ve Tribe</span><span class="lp-logo-text">VendorWorld</span></div>
        <div class="lp-links">
          <a href="javascript:void(0)" class="active" onclick="nav('#/')">Home</a>
          <a href="javascript:void(0)" onclick="lpScroll('lp-about')">About</a>
          <a href="javascript:void(0)" onclick="lpScroll('lp-features')">Features</a>
          <a href="javascript:void(0)" onclick="lpScroll('lp-faq')">FAQ</a>
        </div>
      </div>
      <div class="lp-nav-right">
        <span class="lp-login" onclick="nav('#/login')">Login</span>
        <button class="btn" onclick="openDemo()">Request a Demo</button>
      </div>
    </nav>

    <section class="lp-hero">
      <span class="lp-badge">VENDOR ONBOARDING</span>
      <h1>Vendor onboarding in days, not weeks</h1>
      <p class="sub">Collect details, documents, and bank data from vendors — verified automatically and approved in days.</p>
      <div class="lp-ctas">
        <button class="btn" onclick="openDemo()">Request a Demo</button>
      </div>
    </section>

    <section class="lp-section">
      <div class="lp-wrap">
        <div class="lp-sec-head"><h2>How It Works</h2><p>Four simple steps to join our global vendor network</p></div>
        <div class="lp-steps">${steps}</div>
      </div>
    </section>

    <section class="lp-section" style="padding-top:0">
      <div class="lp-wrap">
        <div class="lp-checklist">
          <div class="lp-check-info">
            <h2>Before You Start</h2>
            <p>Preparing these documents ahead of time guarantees a seamless registration. Incorrect or expired certificates may delay approval.</p>
          </div>
          <div class="lp-check-card">${checklist}</div>
        </div>
      </div>
    </section>

    <section class="lp-section" id="lp-about">
      <div class="lp-wrap lp-about">
        <div class="lp-about-col">
          <div class="lp-eyebrow">About the platform</div>
          <h2>Streamlined vendor qualification</h2>
          <p>VendorWorld handles digital identity verification, audit trails, and automatic notification systems to keep corporate standards clean, transparent, and always up-to-date.</p>
        </div>
        <div class="lp-about-col" id="lp-features">
          <h3>What the Portal Does</h3>
          <div class="lp-flist">${features}</div>
        </div>
      </div>
    </section>

    <section class="lp-section alt" id="lp-faq">
      <div class="lp-wrap">
        <div class="lp-sec-head"><h2>Frequently Asked Questions</h2><p>Got questions? We have answers.</p></div>
        <div class="lp-faq-list">${faqs}</div>
      </div>
    </section>

    <footer class="lp-footer">
      <span>Support: vendors@company.com</span>
      <div class="lp-footer-links">
        <a href="javascript:void(0)" onclick="toast('Terms of Service')">Terms of Service</a>
        <a href="javascript:void(0)" onclick="toast('Privacy Policy')">Privacy Policy</a>
      </div>
    </footer>
    ${demoModalHTML()}
  </div>`;
}
window.lpScroll=(secId)=>{ const el=document.getElementById(secId); if(el) el.scrollIntoView({behavior:'smooth', block:'start'}); };
function demoModalHTML(){
  return `<div class="modal-overlay" id="demoModal" style="display:none" onclick="if(event.target===this)closeDemo()">
    <div class="modal" role="dialog" aria-modal="true" aria-labelledby="demoTitle">
      <div id="demoBody">
        <h3 id="demoTitle">Request a demo</h3>
        <p class="modal-sub">See VendorWorld in action. Tell us where to reach you.</p>
        <div class="field"><label>Name</label><input type="text" id="demoName" placeholder="Your name"></div>
        <div class="field"><label>Work email</label><input type="email" id="demoEmail" placeholder="you@company.com"></div>
        <div class="field"><label>Phone number</label><input type="text" id="demoPhone" placeholder="+91 …"></div>
        <div class="err" id="demoErr" style="display:none"></div>
        <div class="modal-actions">
          <button class="btn btn-secondary" onclick="closeDemo()">Cancel</button>
          <button class="btn" onclick="submitDemo()">Book a demo</button>
        </div>
        <p class="consent">By submitting, you agree to be contacted about VendorWorld. See our Privacy Policy.</p>
      </div>
    </div>
  </div>`;
}
window.openDemo=()=>{ const m=document.getElementById('demoModal'); if(m){ m.style.display='flex'; setTimeout(()=>{const n=document.getElementById('demoName'); if(n)n.focus();},0); } };
window.closeDemo=()=>{ const m=document.getElementById('demoModal'); if(m) m.style.display='none'; };
window.submitDemo=()=>{
  const val=id=>((document.getElementById(id)||{}).value||'').trim();
  const name=val('demoName'), email=val('demoEmail'), phone=val('demoPhone');
  const err=document.getElementById('demoErr');
  const emailOk=/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(email);
  if(!name||!emailOk||!phone){
    err.textContent = !name?'Please enter your name.' : !emailOk?'Please enter a valid work email.' : 'Please enter a phone number.';
    err.style.display='block'; return;
  }
  document.getElementById('demoBody').innerHTML = `<div class="modal-success">
    <div style="font-size:42px;color:var(--ok)">✓</div>
    <h3 style="margin:10px 0 8px">Demo requested</h3>
    <p class="muted">Thanks for requesting a demo. Our sales team will contact you shortly to schedule it.</p>
    <div style="margin-top:20px"><button class="btn" onclick="closeDemo()">Done</button></div></div>`;
};

/* ---- Stage 1: Login chooser — Vendor · Client · Admin ---- */
function loginChooser(){
  return `<div class="auth-wrap"><div class="auth-card">
    <div class="center-brand"><h2>Welcome to VendorWorld</h2><p class="meta">How would you like to sign in?</p></div>
    <div class="card card-pad">
      <button class="btn btn-block" onclick="nav('#/login/vendor')">Login as Vendor</button>
      <p class="meta center" style="margin:8px 0 16px">Upload and manage your documents</p>
      <button class="btn btn-secondary btn-block" onclick="nav('#/login/client')">Login as Client</button>
      <p class="meta center" style="margin:8px 0 18px">Manage vendors in the VMS tool</p>
      <div class="divider"></div>
      <div class="section-label">Admin</div>
      <button class="btn-google" onclick="nav('#/login/google')">${googleG()} Continue with Google</button>
    </div>
    <p class="meta center" style="margin-top:16px"><a href="#/">← Back to home</a></p>
  </div></div>`;
}

/* ---- Stage 1: Google SSO (simulated) — internal staff ---- */
function googleAuth(){
  return `<div class="auth-wrap"><div class="g-card">
    <div style="text-align:center;margin-bottom:24px">
      <span style="display:inline-block;width:40px;height:40px">${googleG()}</span>
      <h2 style="font-weight:400;font-size:22px;margin:14px 0 6px">Choose an account</h2>
      <p class="meta">to continue to <b style="color:var(--ink)">VendorWorld</b></p>
    </div>
    <button class="g-acct" onclick="googleSignIn('sb')">
      <span class="avatar">SB</span>
      <span><b>S. Borse</b><br><span class="meta">s.borse@company.com</span></span>
    </button>
    <button class="g-acct" onclick="googleSignIn('other')">
      <span class="g-acct-ic">+</span>
      <span>Use another account</span>
    </button>
    <p class="meta" style="margin-top:20px;font-size:12px;line-height:1.5">To continue, Google will share your name and email address with VendorWorld. Before using this app, review its <a href="#/">privacy policy</a> and <a href="#/">terms of service</a>.</p>
  </div>
  <p class="meta center" style="margin-top:16px;position:absolute;bottom:24px"><a href="#/login">← Back to login options</a></p>
  </div>`;
}
window.googleSignIn=(acct)=>{
  state.currentUser={name:'S. Borse', initials:'SB', role:'Admin', dept:'Procurement', cc:'CC-4471', type:'admin'};
  toast(acct==='other'?'Signed in with Google':'Signed in as S. Borse · Admin');
  nav('#/home');
};

/* ---- Stage 1: Client login (email + password) ---- */
function loginClient(){
  return `<div class="auth-wrap"><div class="auth-card">
    <div class="center-brand"><h2>Client login</h2><p class="meta">VMS tool users</p></div>
    <div class="card card-pad">
      <div class="field"><label>Work email</label><input type="email" id="clientEmail" placeholder="you@company.com"></div>
      <div class="field"><label>Password</label><input type="password" id="clientPass" placeholder="••••••••"></div>
      <div class="err" id="clientErr" style="display:none;color:var(--danger);font-size:13px;margin-bottom:12px"></div>
      <button class="btn btn-block" onclick="clientLogin()">Continue</button>
      <p class="meta center" style="margin-top:12px"><a href="javascript:void(0)" onclick="nav('#/login/client/reset')">Forgot password?</a></p>
    </div>
    <p class="meta center" style="margin-top:16px"><a href="#/login">← Other login options</a></p>
  </div></div>`;
}
window.clientLogin=()=>{
  const email=((document.getElementById('clientEmail')||{}).value||'').trim()||'user@company.com';
  state.currentUser={name:email.split('@')[0], initials:email.slice(0,2).toUpperCase(), role:'Client', dept:'Procurement', cc:'—', type:'client'};
  toast('Signed in as Client');
  nav('#/home');
};

/* ---- Stage 1: Client password reset ---- */
function clientReset(){
  return `<div class="auth-wrap"><div class="auth-card">
    <div class="center-brand"><h2>Reset your password</h2></div>
    <div class="card card-pad" id="resetBody">
      <p class="muted" style="margin-bottom:14px">Enter your work email and we'll send a reset link.</p>
      <div class="field"><label>Work email</label><input type="email" id="resetEmail" placeholder="you@company.com"></div>
      <div class="err" id="resetErr" style="display:none;color:var(--danger);font-size:13px;margin-bottom:12px"></div>
      <button class="btn btn-block" onclick="clientResetSend()">Send reset link</button>
    </div>
    <p class="meta center" style="margin-top:16px"><a href="#/login/client">← Back to client login</a></p>
  </div></div>`;
}
window.clientResetSend=()=>{
  const email=((document.getElementById('resetEmail')||{}).value||'').trim()||'your email';
  document.getElementById('resetBody').innerHTML=`<div class="center" style="padding:8px 0">
    <div style="font-size:40px;color:var(--ok)">✓</div>
    <h3 style="margin:10px 0 8px">Check your inbox</h3>
    <p class="muted">A password reset link is on its way to ${esc(email)}.</p>
    <div style="margin-top:18px"><button class="btn" onclick="nav('#/login/client')">Back to login</button></div></div>`;
};

/* ---- Stage 1: Vendor login (OTP) ---- */
function loginVendor(){
  const vl=state.vendorLogin;
  const body = !vl.otpSent ? `
      <div class="field"><label>Work email</label><input type="email" id="vendorEmail" value="${esc(vl.email)}" placeholder="you@vendor.com"></div>
      <div class="err" id="vendorErr" style="display:none;color:var(--danger);font-size:13px;margin-bottom:12px"></div>
      <button class="btn btn-block" onclick="vendorSendOtp()">Send OTP</button>`
    : `
      <p class="muted" style="margin-bottom:14px">We sent a 6-digit code to <b>${esc(vl.email)}</b>.</p>
      <div class="field"><label>Enter OTP</label><input type="text" id="vendorOtp" placeholder="6-digit code" inputmode="numeric"></div>
      <div class="err" id="vendorErr" style="display:none;color:var(--danger);font-size:13px;margin-bottom:12px"></div>
      <button class="btn btn-block" onclick="vendorVerify()">Verify &amp; continue</button>
      <p class="meta center" style="margin-top:12px"><a href="javascript:void(0)" onclick="vendorResetOtp()">Change email</a> · <a href="javascript:void(0)" onclick="toast('OTP resent')">Resend code</a></p>`;
  return `<div class="auth-wrap"><div class="auth-card">
    <div class="center-brand"><h2>Vendor login</h2><p class="meta">Sign in with a one-time code</p></div>
    <div class="card card-pad">${body}</div>
    <p class="meta center" style="margin-top:16px"><a href="#/login">← Other login options</a></p>
  </div></div>`;
}
window.vendorSendOtp=()=>{
  const email=((document.getElementById('vendorEmail')||{}).value||'').trim()||'vendor@company.com';
  state.vendorLogin.email=email; state.vendorLogin.otpSent=true;
  toast('OTP sent to '+email); render();
};
window.vendorResetOtp=()=>{ state.vendorLogin.otpSent=false; render(); };
window.vendorVerify=()=>{
  const email=state.vendorLogin.email;
  const vend=state.vendors.find(v=>(v.email||'').toLowerCase()===email.toLowerCase()) || vendor('acme');
  state.currentUser={name:vend.contact!=='—'?vend.contact:email.split('@')[0], initials:(vend.contact!=='—'?vend.contact:email).slice(0,2).toUpperCase(), role:'Vendor', dept:vend.name, cc:'—', type:'vendor'};
  state.vendorLogin={email:'',otpSent:false}; state.onbStep=0;
  toast('Signed in as Vendor');
  nav('#/onboard/'+vend.id);
};

/* ---- Stage 1: Admin signup wizard (4 steps) ---- */
function signup(){
  const step = state.signupStep;
  const dots = [1,2,3,4].map(n=>`<i class="${n<=step?'on':''}"></i>`).join('');
  let inner='';
  if(step===1) inner=`
    <h3>What's your work email?</h3>
    <div class="field" style="margin-top:14px"><input type="email" placeholder="you@company.com"></div>
    <button class="btn btn-block" onclick="signupNext()">Continue</button>
    <p class="hint" style="margin-top:12px">ⓘ We check if your organisation already exists.</p>`;
  else if(step===2) inner=`
    <h3>How do you want to verify?</h3>
    <div style="display:grid;gap:10px;margin-top:14px">
      <button class="btn btn-secondary btn-block" onclick="signupNext()">Password</button>
      <button class="btn btn-secondary btn-block" onclick="signupNext()">Google SSO</button>
      <button class="btn btn-secondary btn-block" onclick="signupNext()">Microsoft SSO</button>
      <button class="btn btn-secondary btn-block" onclick="signupNext()">Mobile + OTP</button>
    </div>`;
  else if(step===3) inner=`
    <h3>Set a password</h3>
    <div class="field" style="margin-top:14px"><label>Password</label><input type="password"></div>
    <div class="field"><label>Confirm</label><input type="password"></div>
    <button class="btn btn-block" onclick="signupNext()">Continue</button>`;
  else inner=`
    <h3>Set up MFA</h3>
    <div class="field" style="margin-top:14px"><label>Name</label><input type="text" value="S. Borse"></div>
    <div class="field"><label>Mobile</label><input type="text" placeholder="+91 …"></div>
    <button class="btn btn-block" onclick="state.signupStep=1; nav('#/org-setup')">Finish</button>`;
  return `<div class="auth-wrap"><div class="auth-card">
    <div class="card card-pad">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:18px">
        <span class="meta">Step ${step} of 4</span><span class="dots-progress">${dots}</span>
      </div>
      ${inner}
      ${step>1?`<p class="meta center" style="margin-top:14px"><a href="javascript:void(0)" onclick="signupBack()">← Back</a></p>`:''}
    </div>
  </div></div>`;
}
window.signupNext = ()=>{ state.signupStep=Math.min(4,state.signupStep+1); render(); };
window.signupBack = ()=>{ state.signupStep=Math.max(1,state.signupStep-1); render(); };

/* ---- Stage 1: Admin first-time org setup dashboard ---- */
function orgSetup(){
  const o = state.orgSetup;
  const items = [
    {done:true, lbl:'Confirm company details', sub:'name / address / size'},
    {done:true, lbl:'Add authorised contact', sub:'one additional contact'},
    {done:true, lbl:'Upload required documents', sub:''},
    {done:o.risk, lbl:'Complete risk assessment', sub:'', key:'risk', primary:true},
    {done:o.security, lbl:'Complete security questionnaire', sub:'', key:'security'},
    {done:o.decl, lbl:'Review declarations', sub:'consents + declarations', key:'decl'},
    {done:false, lbl:'Submit for approval', sub:'approved by Super-Admin'}
  ];
  const doneCount = items.filter(i=>i.done).length;
  const pct = Math.round(doneCount/items.length*100);
  const rows = items.map(i=>`
    <div class="doc-line">
      ${glyph(i.done?'ok':'todo')}
      <div class="name">${esc(i.lbl)} ${i.sub?`<span class="meta"> · ${esc(i.sub)}</span>`:''}</div>
      ${!i.done && i.key?`<button class="btn ${i.primary?'':'btn-secondary'} btn-sm" onclick="orgDo('${i.key}')">Start</button>`:''}
      ${!i.done && !i.key && i.lbl.startsWith('Submit')?`<button class="btn btn-sm" onclick="toast('Submitted for Super-Admin approval.')">Submit</button>`:''}
    </div>`).join('');
  return shell('', `
    <div class="page-head"><div><h1>Finish setting up your organisation</h1>
      <p class="meta">Onboard your own org before adding vendors.</p></div>
      <span class="badge accent">${doneCount} of ${items.length} done</span></div>
    <div class="card card-pad">
      <div style="height:8px;background:var(--surface-2);border-radius:999px;overflow:hidden;margin-bottom:20px">
        <div style="height:100%;width:${pct}%;background:var(--accent);border-radius:999px;transition:.3s"></div></div>
      ${rows}
    </div>
    <p class="meta center" style="margin-top:16px">Skip ahead to the product → <a href="#/vendors">Vendor directory</a></p>`);
}
window.orgDo = (k)=>{ state.orgSetup[k]=true; toast('Marked complete.'); render(); };

/* ---- Stage 2/9: Vendor directory ---- */
function filteredVendors(){
  const f=state.filters;
  return state.vendors.filter(v=>{
    const q=f.search.toLowerCase();
    if(q && !(v.name.toLowerCase().includes(q)||v.contact.toLowerCase().includes(q)||v.category.toLowerCase().includes(q))) return false;
    if(f.status && v.status!==f.status) return false;
    if(f.dept && v.dept!==f.dept) return false;
    if(f.risk && riskLabel(computeRisk(v).level)!==f.risk) return false;
    return true;
  });
}
function riskLabel(level){ return level==='Elevated' ? 'High' : 'Low'; }
function riskLevelBadge(level){ return level==='Elevated' ? `<span class="badge danger">High</span>` : `<span class="badge ok">Low</span>`; }
function directoryResults(){
  const list=filteredVendors();
  const rows = list.map(v=>`
    <tr class="clickable" onclick="if(!event.target.closest('.rowmenu'))nav('#/vendor/${v.id}')">
      <td><b>${esc(v.name)}</b></td>
      <td>${esc(v.contact)}</td>
      <td>${esc(v.raisedBy||'—')}</td>
      <td>${esc(v.category)}</td>
      <td>${riskLevelBadge(computeRisk(v).level)}</td>
      <td>${statusBadge(v.status)}</td>
      <td class="rowmenu" style="text-align:right">
        <span class="dots" onclick="toggleMenu(event,'${v.id}')">⋮</span>
        <div class="menu" id="menu-${v.id}" style="display:none">
          <button onclick="toast('Edit ${esc(v.name)}')">Edit</button>
          <button onclick="toast('Message sent to ${esc(v.contact)}')">Message</button>
          <button class="danger" onclick="deactivate('${v.id}')">Deactivate</button>
        </div>
      </td>
    </tr>`).join('');

  return list.length ? `<div class="table-wrap"><table>
      <thead><tr><th>Vendor Name</th><th>Vendor Contact</th><th>Client Contact</th><th>Category</th><th>Risk Level</th><th>Status</th><th></th></tr></thead>
      <tbody>${rows}</tbody></table></div>`
    : `<div class="empty-state"><div class="em">📭</div><h3>No vendors match</h3>
        <p class="muted">Try clearing filters, or add your first vendor to begin tracking risk and compliance.</p>
        <button class="btn" style="margin-top:14px" onclick="nav('#/request')">Add vendor</button></div>`;
}
// Existing Search component — search box + filters + results table. Shared by the directory and the client dashboard.
function vendorSearchPanel(){
  const f=state.filters;
  const chip=(key,val,lbl)=>`<span class="chip ${f[key]===val?'active':''}" onclick="setFilter('${key}','${f[key]===val?'':val}')">${lbl}</span>`;
  const flbl=(on,txt)=>`<span class="flabel ${on?'flabel-on':''}" style="margin-right:4px">${txt}</span>`;
  const sel=(key,opts)=>`<select onchange="setFilter('${key}',this.value)" style="width:auto;min-width:150px">`
    + opts.map(([v,l])=>`<option value="${v}" ${f[key]===v?'selected':''}>${l}</option>`).join('')
    + `</select>`;
  const depts=[...new Set(state.vendors.map(v=>v.dept).filter(d=>d&&d!=='—'))];
  const more = state.showMoreFilters ? `
      <div class="chips" style="margin-top:12px;padding-top:12px;border-top:1px solid var(--border);align-items:center">
        ${flbl(f.dept,'Department')}
        ${sel('dept',[['','All departments'],...depts.map(d=>[d,d])])}
      </div>` : '';
  return `
    <div class="card card-pad" style="margin-bottom:16px">
      <div class="search" style="margin-bottom:14px">
        <span class="ico">⚲</span>
        <input type="text" placeholder="Search by name, contact, or category…" value="${esc(f.search)}" oninput="setSearch(this.value)">
      </div>
      <div class="chips" style="align-items:center">
        ${flbl(f.status,'Status')}
        ${sel('status',[['','All statuses'],['active','Active'],['onboarding','Onboarding'],['flagged','Flagged']])}
        <span style="width:12px"></span>
        ${flbl(f.risk,'Risk')}
        ${sel('risk',[['','All risk'],['Low','Low'],['High','High']])}
        <span style="width:12px"></span>
        <span class="chip" onclick="toggleMoreFilters()">${state.showMoreFilters?'− Less filters':'+ More filters'}</span>
        ${(f.search||f.status||f.dept||f.risk)?`<span class="chip" onclick="clearFilters()">✕ Clear</span>`:''}
      </div>
      ${more}
    </div>
    <div class="card" id="dirResults">${directoryResults()}</div>`;
}
window.toggleMoreFilters=()=>{ state.showMoreFilters=!state.showMoreFilters; render(); };
function directory(){
  return shell('vendors', `
    <div class="page-head"><div><h1>Vendor List</h1><p class="meta">Search everything · one list for all statuses</p></div>
      <button class="btn" onclick="nav('#/request')">+ Add vendor</button></div>
    ${vendorSearchPanel()}`);
}
/* ---- Client dashboard (homepage) ---- */
function clientShell(active, body){
  const u=state.currentUser;
  const item=(key,label,route)=>`<a class="${active===key?'active':''}" onclick="nav('${route}')">${label}</a>`;
  return `<div class="cshell">
    <aside class="csidebar">
      <div>
        <div class="cbrand"><span class="mk">🛡</span><div><b>VMS Enterprise</b><span>Compliance &amp; Risk</span></div></div>
        <nav class="cnav">
          ${item('home','▚&nbsp;&nbsp;Home','#/home')}
          ${item('admin','◨&nbsp;&nbsp;Admin Console','#/admin')}
          ${item('search','⚲&nbsp;&nbsp;Search Vendors','#/vendors')}
          ${item('records','▤&nbsp;&nbsp;Vendor Records','#/vendors')}
          ${item('activity','◷&nbsp;&nbsp;Activity Log','#/activity')}
        </nav>
      </div>
      <div class="cbottom">
        <div class="cprofile"><span class="av">${esc(u.initials)}</span><div><b>${esc(u.name)}</b><span>${esc(u.role)}</span></div></div>
        <button class="csignout" onclick="signOut()">⎋&nbsp; Sign Out</button>
      </div>
    </aside>
    <main class="cmain">${body}</main>
  </div>`;
}
function clientHome(){
  const u=state.currentUser;
  const first=(u.name||'').split(' ')[0]||u.name;
  const elevated=state.vendors.filter(v=>computeRisk(v).level==='Elevated').length;
  const inprog=state.vendors.filter(v=>v.status==='onboarding').length;
  const registered=state.vendors.filter(v=>v.status==='active').length;
  return clientShell('home', `
    <div class="page-head">
      <div><h1 style="font-size:28px">Welcome back, ${esc(first)}</h1>
        <p class="meta">Here's an overview of active onboardings and risk triage for today.</p></div>
      <button class="btn" onclick="nav('#/request')">+ Raise new onboarding request</button>
    </div>
    <div class="kpi-row">
      <div class="kpi"><span class="ic" style="background:var(--danger-soft);color:var(--danger)">⚠</span><div><b>${elevated} High risk</b><span>Action required</span></div></div>
      <div class="kpi"><span class="ic" style="background:var(--accent-soft);color:var(--accent)">◐</span><div><b>${inprog} In progress</b><span>Active onboardings</span></div></div>
      <div class="kpi"><span class="ic" style="background:var(--ok-soft);color:var(--ok)">✓</span><div><b>${registered} Active</b><span>Onboarded &amp; compliant vendors</span></div></div>
    </div>
    <div class="page-head" style="margin-bottom:12px"><h2 style="font-size:18px">Vendor List</h2>
      <button class="btn btn-secondary btn-sm" onclick="nav('#/vendors')">Show Full list →</button></div>
    ${vendorSearchPanel()}`);
}
/* ---- Admin Console (analytics dashboard — from vendor-dashboard-handoff.md) ---- */
function adminConsole(){
  const total=state.vendors.length;
  const pending=state.vendors.filter(v=>v.status==='onboarding').length;
  const highrisk=state.vendors.filter(v=>computeRisk(v).level==='Elevated').length;
  const kpi=(bg,col,ic,num,lbl)=>`<div class="kpi"><span class="ic" style="background:${bg};color:${col}">${ic}</span><div><b>${num}</b><span>${lbl}</span></div></div>`;

  // Onboarding funnel (ends at Approved) — sample cohort, wire to API later
  const funnel=[{k:'Invite sent',v:120},{k:'Details submitted',v:96},{k:'Documents uploaded',v:78},{k:'Bank verified',v:61},{k:'Approved',v:52}];
  const fmax=funnel[0].v;
  const funnelRows=funnel.map((s,i)=>{
    const w=Math.max(6,Math.round(s.v/fmax*100));
    const drop=i===0?'—':`−${Math.round((funnel[i-1].v-s.v)/funnel[i-1].v*100)}%`;
    return `<div class="afrow"><div class="flab">${esc(s.k)}</div>
      <div class="abar-wrap"><div class="abar" style="width:${w}%">${s.v}</div></div>
      <div class="fmeta">${drop}</div></div>`;
  }).join('');

  // TAT (hours, last 4 weeks) — bars narrow as TAT improves; target marker at 48h
  const tat=[{w:'4 wks ago',v:72},{w:'3 wks ago',v:64},{w:'2 wks ago',v:58},{w:'This week',v:50}];
  const target=48, tmax=Math.max.apply(null,tat.map(t=>t.v));
  const tpos=Math.round(target/tmax*100);
  const tatRows=tat.map(t=>{
    const w=Math.max(6,Math.round(t.v/tmax*100)), met=t.v<=target;
    return `<div class="afrow"><div class="flab">${esc(t.w)}</div>
      <div class="abar-wrap"><div class="abar ${met?'good':''}" style="width:${w}%">${t.v}h</div><div class="atarget" style="left:${tpos}%"></div></div>
      <div class="fmeta">${met?'✓ met':''}</div></div>`;
  }).join('');
  const tatNow=tat[tat.length-1].v, tatDelta=tat[0].v-tatNow;

  const acts=state.activity.slice(0,6).map(a=>`<li class="${a.system?'system':''}"><div class="t-when">${esc(a.when)}</div>${esc(a.ev)} <span class="meta">· ${esc(a.vendor)}${a.who?' · '+esc(a.who):''}</span></li>`).join('');

  return clientShell('admin', `
    <div class="page-head"><div><h1 style="font-size:28px">Admin Console</h1>
      <p class="meta">Overview &amp; analytics across all vendors and onboarding.</p></div>
      <button class="btn btn-secondary" onclick="exportVendorsCSV()">⤓&nbsp; Export CSV</button></div>
    <div class="kpi-row six">
      ${kpi('var(--accent-soft)','var(--accent)','▤',total,'My vendors')}
      ${kpi('var(--warn-soft)','var(--warn)','◔',pending,'Pending approvals')}
      ${kpi('var(--danger-soft)','var(--danger)','⚠',highrisk,'High-risk vendors')}
      ${kpi('var(--danger-soft)','var(--danger)','⏱',12,'Overdue vendor tasks')}
      ${kpi('var(--warn-soft)','var(--warn)','⧗',9,'Expiring documents')}
      ${kpi('var(--accent-soft)','var(--accent)','✚',6,'New vendor requests')}
    </div>
    <div class="acols">
      <div class="card card-pad">
        <div class="page-head" style="margin-bottom:16px"><h2 style="font-size:18px">Onboarding funnel</h2><span class="badge accent">Cohort: ${fmax}</span></div>
        <div class="afunnel">${funnelRows}</div>
      </div>
      <div class="card card-pad">
        <div class="page-head" style="margin-bottom:2px"><h2 style="font-size:18px">Turnaround time</h2><span class="badge accent">Target ${target} hrs</span></div>
        <p class="meta" style="margin-bottom:16px"><b style="font-size:22px;color:var(--ink)">${tatNow} hrs</b> &nbsp;·&nbsp; ↓ ${tatDelta} hrs vs 4 wks ago</p>
        <div class="afunnel">${tatRows}</div>
      </div>
    </div>
    <div class="page-head" style="margin:24px 0 12px"><h2 style="font-size:18px">Vendor tracking</h2>
      <button class="btn btn-secondary btn-sm" onclick="exportVendorsCSV()">⤓&nbsp; Export CSV</button></div>
    ${vendorSearchPanel()}
    <div class="card card-pad" style="margin-top:16px">
      <div class="section-label">Recent activity</div>
      <ul class="timeline">${acts}</ul>
    </div>`);
}
window.exportVendorsCSV=()=>{
  const q=s=>'"'+String(s==null?'':s).replace(/"/g,'""')+'"';
  const head=['Vendor','Vendor ID','Category','Status','Risk','Last updated'];
  const lines=state.vendors.map(v=>[v.name,v.id,v.category,(STATUS[v.status]||{}).lbl||v.status,riskLabel(computeRisk(v).level),v.onboarded].map(q).join(','));
  const csv=[head.map(q).join(','),...lines].join('\r\n');
  try{
    const url=URL.createObjectURL(new Blob([csv],{type:'text/csv'}));
    const a=document.createElement('a'); a.href=url; a.download='vendor-tracking.csv';
    document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
    toast('Exported vendor-tracking.csv');
  }catch(e){ toast('Export ready · vendor-tracking.csv'); }
};
window.signOut=()=>{ state.currentUser={name:'S. Borse',initials:'SB',role:'Admin',dept:'Product',cc:'—',type:'admin'}; toast('Signed out'); nav('#/login'); };
window.setSearch=(v)=>{ state.filters.search=v; const el=document.getElementById('dirResults'); if(el) el.innerHTML=directoryResults(); };
window.setFilter=(k,v)=>{ state.filters[k]=v; render(); };
window.clearFilters=()=>{ state.filters={search:'',status:'',dept:'',risk:''}; render(); };
window.toggleMenu=(e,id)=>{ e.stopPropagation(); document.querySelectorAll('.menu').forEach(m=>{if(m.id!=='menu-'+id)m.style.display='none';}); const m=document.getElementById('menu-'+id); m.style.display=m.style.display==='none'?'block':'none'; };
window.deactivate=(id)=>{ const v=vendor(id); v.status='suspended'; logActivity(v.name,'Deactivated'); toast(v.name+' deactivated'); render(); };
document.addEventListener('click',()=>document.querySelectorAll('.menu').forEach(m=>m.style.display='none'));
document.addEventListener('keydown',e=>{ if(e.key==='Escape' && typeof closeDemo==='function') closeDemo(); });

/* ---- Stage 2: Vendor profile ---- */
function profile(id){
  const v=vendor(id); if(!v) return notFound();
  const docs = v.docs.length ? v.docs.map(d=>`<div class="doc-line">${glyph(d.st)}<div class="name">${esc(d.name)}</div><span class="meta">${esc(d.note)}</span></div>`).join('')
    : `<p class="muted">No documents collected yet.</p>`;
  const acts = v.activity.length ? `<ul class="timeline">${v.activity.map(a=>`<li class="${a.system?'system':''}"><div class="t-when">${esc(a.when)}</div>${esc(a.ev)}${a.who?` <span class="meta">· ${esc(a.who)}</span>`:''}</li>`).join('')}</ul>`
    : `<p class="muted">No activity for this vendor yet.</p>`;
  const bank = v.bank ? `
    <div class="kv"><span class="k">Account</span><span>${esc(v.bank.acct)}</span></div>
    <div class="kv"><span class="k">IFSC</span><span>${esc(v.bank.ifsc)}</span></div>
    <div class="kv"><span class="k">Rails</span><span>${esc(v.bank.rails)}</span></div>
    <div class="kv"><span class="k">Terms</span><span>${esc(v.bank.terms)}</span></div>` : `<p class="muted">Bank details not submitted.</p>`;
  return shell('vendors', `
    <div class="breadcrumb" onclick="nav('#/vendors')">‹ Back to vendors</div>
    <div class="page-head">
      <div><h1>${esc(v.name)}</h1><p class="meta">${esc(v.category)} services · onboarded ${esc(v.onboarded)} · Risk: ${riskLabel(computeRisk(v).level)}</p></div>
      ${statusBadge(v.status)}</div>
    <div class="pill-nav">
      <a class="active" href="#/vendor/${v.id}">Overview</a>
      <a href="#/verify/${v.id}">Verification</a>
      <a href="#/risk/${v.id}">Risk review</a>
      <a href="#/approve/${v.id}">Approvals</a>
    </div>
    <div class="split">
      <div>
        <div class="card card-pad"><div class="section-label">Compliance &amp; documents</div>${docs}</div>
        <div class="card card-pad"><div class="section-label">Activity (this vendor)</div>${acts}
          <p class="meta" style="margin-top:8px">Org-wide log is on the <a href="#/activity">Activity</a> screen.</p></div>
      </div>
      <div>
        <div class="card card-pad"><div class="section-label">Contact</div>
          <b>${esc(v.contact)}</b><p class="muted" style="margin:4px 0">${esc(v.email)}</p><p class="muted">${esc(v.phone)}</p></div>
        <div class="card card-pad"><div class="section-label">Financial details</div>${bank}</div>
      </div>
    </div>`);
}

/* ---- Stage 3: Raise a request ---- */
function request(){
  return shell('requests', `
    <div class="page-head"><div><h1>New vendor request</h1><p class="meta">Internal only — the vendor knows nothing yet.</p></div>
      <span class="badge">Step 1 of 1</span></div>
    <div class="card card-pad" style="max-width:720px">
      <div class="section-label">Raised by (auto-filled)</div>
      <p style="margin-bottom:18px">${esc(state.currentUser.name)} · ${esc(state.currentUser.dept)} · ${esc(state.currentUser.cc)}</p>
      <div class="divider"></div>
      <div class="field"><label>Vendor name</label>
        <input type="text" id="reqName" placeholder="e.g. Nimbus Cloud" oninput="dupCheck(this.value)">
        <div id="dupWarn"></div>
      </div>
      <div class="field"><label>Vendor contact email</label><input type="email" id="reqEmail" placeholder="contact@vendor.com">
        <p class="hint">Gets the invite to step 2.</p></div>
      <div class="grid-2">
        <div class="field"><label>Country</label><select disabled><option>India</option></select><p class="hint">locked for now</p></div>
        <div class="field"><label>Expected spend</label><select><option>Select range ▾</option><option>&lt; ₹5L</option><option>₹5L–25L</option><option>₹25L–1Cr</option><option>&gt; ₹1Cr</option></select></div>
      </div>
      <div class="field"><label>What we are buying</label><textarea placeholder="Describe the goods or services"></textarea></div>
      <div class="grid-2">
        <div class="field"><label>Project / cost centre</label><input type="text" placeholder="Not a PO number"></div>
        <div class="field"><label>Urgency</label><select><option>Select ▾</option><option>Standard</option><option>High</option><option>Critical</option></select><p class="hint">drives SLA</p></div>
      </div>
      <div class="field"><label>Project end date</label><input type="date">
        <p class="hint">Vendor auto-deactivates on this date.</p></div>
      <div class="divider"></div>
      <div class="field"><label>Who completes step 2</label>
        <div class="radio-row">
          <label class="radio-opt sel" onclick="pickCompleter(this,true)"><input type="radio" name="completer" checked> Vendor <span class="meta">(default)</span></label>
          <label class="radio-opt" onclick="pickCompleter(this,false)"><input type="radio" name="completer"> Our staff <span class="meta">(needs reason)</span></label>
        </div>
        <div id="staffReason" style="display:none;margin-top:10px"><input type="text" placeholder="Reason for completing on behalf of the vendor"></div>
      </div>
      <div style="display:flex;gap:10px;justify-content:flex-end;margin-top:8px">
        <button class="btn btn-secondary" onclick="toast('Draft saved')">Save draft</button>
        <button class="btn" onclick="submitRequest()">Submit &amp; invite</button>
      </div>
    </div>`);
}
window.dupCheck=(val)=>{
  const box=document.getElementById('dupWarn'); if(!box) return;
  const hit=state.vendors.find(v=>val.length>2 && v.name.toLowerCase().startsWith(val.toLowerCase().slice(0,3)));
  box.innerHTML = hit ? `<div class="card" style="border-color:var(--warn);background:var(--warn-soft);padding:12px 14px;margin-top:10px">
      <b>⚠ Similar vendor found</b><br><span class="muted">${esc(hit.name)} · PAN AAxxx1234A · ${STATUS[hit.status].lbl}</span><br>
      <div style="margin-top:8px;display:flex;gap:8px"><button class="btn btn-secondary btn-sm" onclick="nav('#/vendor/${hit.id}')">View existing</button>
      <button class="btn-ghost btn btn-sm" onclick="document.getElementById('dupWarn').innerHTML=''">Not the same — continue</button></div></div>` : '';
};
window.pickCompleter=(el,isVendor)=>{ document.querySelectorAll('.radio-opt').forEach(r=>r.classList.remove('sel')); el.classList.add('sel'); document.getElementById('staffReason').style.display=isVendor?'none':'block'; };
window.submitRequest=()=>{
  const name=(document.getElementById('reqName')||{}).value||'New Vendor';
  const id=name.toLowerCase().replace(/[^a-z0-9]/g,'').slice(0,12)||'vendor'+Date.now();
  if(!vendor(id)) state.vendors.unshift({id,name,contact:'—',email:(document.getElementById('reqEmail')||{}).value||'',phone:'—',
    category:'Pending',dept:'—',status:'onboarding',risk:'—',riskScore:null,onboarded:'—',raisedBy:state.currentUser.name,
    buying:'',docs:[],bank:null,activity:[{when:'Now',who:state.currentUser.name,ev:'Request raised & invite sent'}],
    drivers:{dataAccess:false,poshApplies:false,pmlaApplies:false,clientBFSI:false,bankMismatch:false}, riskAnswers:{}, declAnswers:{}});
  state.activity.unshift({when:'Now',who:state.currentUser.name,vendor:name,ev:'Request raised — invite sent'});
  toast('Invite sent to vendor');
  state.onbStep=0;
  nav('#/invited/'+id);
};

/* ---- Stage 3.5: Request submitted — hand-off to the Vendor persona flow ---- */
function inviteSent(id){
  const v=vendor(id); if(!v) return notFound();
  return shell('requests', `
    <div class="container-sm" style="margin:0 auto">
    <div class="card card-pad center" style="padding:44px 30px">
      <div style="font-size:42px;color:var(--ok)">✓</div>
      <h1 style="margin:12px 0 6px">Request submitted &amp; invite sent</h1>
      <p class="muted">An onboarding invite has been emailed to <b>${esc(v.email||'the vendor')}</b>.</p>
      <div class="card" style="background:var(--surface-2);text-align:left;padding:16px 18px;margin:22px 0">
        <div class="section-label">What happens next</div>
        <p class="muted" style="margin:0">The next step runs in the <b>Vendor persona</b> flow: the vendor signs in with a one-time code to complete their business details, upload documents, and submit bank data for verification. To preview that experience, continue as the vendor.</p>
      </div>
      <button class="btn btn-block" onclick="nav('#/login/vendor')">Login as Vendor →</button>
      <p class="meta center" style="margin-top:12px"><a href="#/home">Back to home</a> · <a href="#/vendors">Vendor list</a></p>
    </div></div>`);
}

/* ---- Stage 4: Vendor onboarding 2A/2B/2C ---- */
function onboarding(id){
  const v=vendor(id); if(!v) return notFound();
  const step=state.onbStep; // 0=details,1=docs,2=declaration,3=bank,4=pennydrop
  const steps=['Details','Documents','Declaration','Bank Details'];
  const stepper=`<div class="stepper">${steps.map((s,i)=>`
    <div class="step ${i<step?'done':''} ${i===step?'current':''}">
      <div class="bub">${i<step?'✓':i+1}</div><div class="lbl">${s}</div><div class="bar"></div></div>`).join('')}</div>`;
  let panel='';
  if(step===0) panel=onbDetails(v);
  else if(step===1) panel=onbDocs(v);
  else if(step===2) panel=onbDeclaration(v);
  else if(step===3) panel=onbBank(v);
  else panel=onbPenny(v);
  return `
    <header class="topbar"><div class="brand">VMS</div>
      <div class="spacer"></div><span class="badge">Vendor portal · invite link</span></header>
    <main class="container container-sm" style="max-width:760px">
      <div class="card card-pad" style="margin-bottom:16px">
        <h2 id="onbTitle" style="margin-bottom:4px">Onboarding for ${esc(v.name)}</h2>
        ${step<4?`<p class="meta" style="margin-bottom:14px">Step ${step+1} of 4</p>`:''}
        ${step<4?stepper:''}
      </div>
      ${panel}
    </main>`;
}
function onbDetails(v){
  return `<div class="card card-pad">
    <div class="section-label">Your business details</div>
    <div class="field"><label>Business type</label>
      <select><option>Select ▾</option><option>Private Limited Company</option><option>LLP</option><option>Partnership</option><option>Proprietorship</option></select>
      <p class="hint">ⓘ This decides which documents we ask you for next.</p></div>
    <div class="grid-2">
      <div class="field"><label>Legal name (must match PAN)</label><input type="text" id="onbLegalName" value="${esc(v.name)}" oninput="onbRename('${v.id}',this.value)"></div>
      <div class="field"><label>Trading name (if different)</label><input type="text"></div>
    </div>
    <div class="field"><label>Registered address</label><input type="text"></div>
    <div class="grid-2">
      <div class="field"><label>Contact person</label><input type="text" value="${esc(v.contact)}"></div>
      <div class="field"><label>Email / phone</label><div class="field-inline"><input type="text" value="${esc(v.email)}"><input type="text" placeholder="phone"></div></div>
    </div>
    <div class="field"><label>PAN</label><input type="text" value="AAxxx1234A"> <span class="badge ok" style="margin-top:6px">✓ verified — ${esc(v.name)}</span></div>
    <div class="grid-2">
      <div class="field"><label>GST registered?</label><div class="radio-row"><label class="radio-opt sel"><input type="radio" name="gst" checked> Yes</label><label class="radio-opt"><input type="radio" name="gst"> No</label></div></div>
      <div class="field"><label>GSTIN</label><input type="text" value="27AAxxx1234A1Z5"> <span class="badge ok" style="margin-top:6px">✓ active</span></div>
    </div>
    <label class="check" style="margin:6px 0 16px"><input type="checkbox" checked> <span>I undertake to notify any change in MSME status.</span></label>

    <div class="card" style="background:var(--surface-2);padding:14px;margin-bottom:16px">
      <div class="section-label" style="margin-bottom:8px">Automated verification</div>
      ${[['PAN','Income Tax — name + validity','ok'],['GSTIN','GST portal — active · filings current','ok'],['Udyam','Udyam portal — valid · Small','ok'],['CIN','MCA — name + status','ok'],['Name match','PAN vs GST — PARTIAL, review needed','warn'],['Duplicate','Vendor master — no match','ok']].map(r=>`<div class="doc-line" style="padding:8px 0">${glyph(r[2])}<div class="name">${r[0]}</div><span class="meta">${r[1]}</span></div>`).join('')}
    </div>
    <div style="display:flex;justify-content:flex-end"><button class="btn" onclick="onbGo(1)">Continue</button></div>
  </div>`;
}
function onbDocs(v){
  const docs=[
    {k:'pan',lbl:'PAN card',fixed:true},{k:'gst',lbl:'GST certificate',fixed:true},
    {k:'bank',lbl:'Cancelled cheque / bank letter'},{k:'addr',lbl:'Address proof'},
    {k:'udyam',lbl:'Udyam certificate',fixed:true},{k:'decl',lbl:'Signed vendor declaration'}
  ];
  const need=docs.filter(d=>!d.fixed);
  const allDone=need.every(d=>state.docChecks[d.k]);
  const rows=docs.map(d=>{
    const stored=state.docChecks[d.k];
    const shown=stored ? stored : (stored===undefined && d.fixed ? d.k+'.pdf' : null);
    const picker=`<input type="file" id="df_${d.k}" style="display:none" onchange="onDocFile('${d.k}',this)">`;
    return shown
      ? `<div class="doc-line">${glyph('ok')}<div class="name">${d.lbl}</div><span class="meta">${esc(shown)}</span>${picker}<button class="file-x" onclick="removeDoc('${d.k}')" title="Remove file" aria-label="Remove file">✕</button></div>`
      : `<div class="doc-line">${glyph('todo')}<div class="name">${d.lbl}</div>${picker}<button class="btn btn-secondary btn-sm" onclick="document.getElementById('df_${d.k}').click()">Upload</button></div>`;
  }).join('');
  return `<div class="card card-pad">
    <div class="section-label">Upload your documents</div>
    <p class="meta" style="margin:-6px 0 12px">Based on: Private Limited Company · GST + MSME registered</p>
    ${rows}
    <div style="display:flex;justify-content:space-between;margin-top:18px">
      <button class="btn btn-secondary" onclick="onbGo(0)">Back</button>
      <button class="btn" ${allDone?'':'disabled'} onclick="onbGo(2)">Continue</button>
    </div>
    ${allDone?'':`<p class="hint" style="text-align:right;margin-top:8px">Disabled until all required documents are uploaded.</p>`}
  </div>`;
}
window.onDocFile=(k,inp)=>{ const file=inp.files&&inp.files[0]; if(!file) return; state.docChecks[k]=file.name; toast('Uploaded'); render(); };
window.removeDoc=(k)=>{ state.docChecks[k]=false; toast('Removed'); render(); };

/* ---- Onboarding step 3: Vendor self-declaration ----
   Vendor self-selects applicability (→ v.drivers) and self-attests the DOMAINS
   questions (→ v.declAnswers, context only). The reviewer verifies these separately
   on #/risk via the REVIEWER_DOMAINS set, which is what actually scores the risk. */
function declYesno(vid,qid,val){
  return `<span style="display:inline-flex;gap:4px;flex:none">
    <button class="btn btn-sm ${val?'btn-ok':'btn-secondary'}" onclick="setDeclAnswer('${vid}','${qid}',true)">Yes</button>
    <button class="btn btn-sm ${!val?'btn-danger':'btn-secondary'}" onclick="setDeclAnswer('${vid}','${qid}',false)">No</button></span>`;
}
function onbDeclaration(v){
  const da=v.declAnswers||{};
  const drv=(key,lbl)=>`<label class="check" style="margin-bottom:9px"><input type="checkbox" ${v.drivers[key]?'checked':''} onclick="setDriver('${v.id}','${key}',this.checked)"> <span>${lbl}</span></label>`;
  const qrow=q=>`<div class="doc-line"><div class="name" style="font-weight:500">${q.q}</div>${declYesno(v.id,q.id,da[q.id]!==false)}</div>`;
  const applicable=Object.entries(DOMAINS).filter(([,d])=>d.applies(v));
  const domainCards=applicable.map(([,d])=>`
    <div class="card" style="background:var(--surface-2);padding:14px;margin-bottom:12px">
      <div class="section-label" style="margin-bottom:8px">${d.name}</div>
      ${d.critical.concat(d.supporting).map(qrow).join('')}
    </div>`).join('');
  return `<div class="card card-pad">
    <div class="section-label">Your compliance self-declaration</div>
    <p class="meta" style="margin:-6px 0 12px">Answer honestly for your business. Your reviewer verifies these independently before approval.</p>
    <div class="card" style="background:var(--surface-2);padding:14px;margin-bottom:14px">
      <div class="section-label" style="margin-bottom:8px">Which of these apply to you? (decides the questions below)</div>
      ${drv('dataAccess','We access / process / store personal data')}
      ${drv('poshApplies','We deploy on-site staff and have 10+ employees')}
      ${drv('pmlaApplies','High-value spend or government-adjacent work')}
      ${drv('clientBFSI','Our client is a BFSI (bank / NBFC / insurer) entity')}
    </div>
    ${domainCards}
    <label class="check" style="margin:6px 0 16px"><input type="checkbox" id="declAttest" ${state.declAttest?'checked':''} onclick="setDeclAttest(this.checked)"> <span>I declare the above is true and complete to the best of my knowledge.</span></label>
    <div style="display:flex;justify-content:space-between;margin-top:6px">
      <button class="btn btn-secondary" onclick="onbGo(1)">Back</button>
      <button class="btn" ${state.declAttest?'':'disabled'} onclick="onbGo(3)">Continue</button>
    </div>
    ${state.declAttest?'':`<p class="hint" style="text-align:right;margin-top:8px">Tick the declaration to continue.</p>`}
  </div>`;
}
window.setDeclAnswer=(vid,qid,val)=>{ const v=vendor(vid); if(!v) return; v.declAnswers=v.declAnswers||{}; v.declAnswers[qid]=val; render(); };
window.setDeclAttest=(val)=>{ state.declAttest=val; render(); };

function bankUpload(key,label,hint){
  const f=state.bankUploads[key];
  return `<div class="field"><label>${label}</label>
    <input type="file" id="bf_${key}" style="display:none" onchange="onBankFile('${key}',this)">
    ${f ? `<div class="doc-line">${glyph('ok')}<div class="name">${esc(f)}</div>
           <button class="file-x" onclick="removeBankFile('${key}')" title="Remove file" aria-label="Remove file">✕</button></div>`
        : `<button class="btn btn-secondary btn-sm" onclick="document.getElementById('bf_${key}').click()">Upload</button>`}
    <p class="hint">${hint}</p></div>`;
}
window.onBankFile=(key,inp)=>{ const file=inp.files&&inp.files[0]; if(!file) return;
  state.bankUploads[key]=file.name; toast('Uploaded'); render(); };
window.removeBankFile=(key)=>{ delete state.bankUploads[key]; toast('Removed'); render(); };
function onbBank(v){
  return `<div class="card card-pad">
    <div class="section-label">Your bank details</div>
    <div class="field"><label>Account holder name</label><input type="text" id="onbAcctHolder" value="${esc(v.name)}" oninput="onbRename('${v.id}',this.value)"><p class="hint">Exactly as your bank holds it.</p></div>
    <div class="grid-2">
      <div class="field"><label>Account number</label><input type="text"></div>
      <div class="field"><label>Re-enter account no.</label><input type="text"></div>
    </div>
    <div class="grid-2">
      <div class="field"><label>IFSC code</label><input type="text" value="HDFC0001234"> <span class="badge ok" style="margin-top:6px">✓ HDFC Bank, Andheri</span></div>
      <div class="field"><label>Account type</label><select><option>Current</option><option>Savings</option></select></div>
    </div>
    <div class="grid-2">
      <div class="field"><label>Bank / branch</label><input type="text" value="HDFC Bank · Andheri East" disabled></div>
      <div class="field"><label>Currency</label><input type="text" value="INR" disabled></div>
    </div>
    <div class="grid-2">
      ${bankUpload('proof','Bank proof document','Must clearly show name, account number and IFSC.')}
      ${bankUpload('cheque','Cancelled cheque','Upload a clear image/PDF of a cancelled cheque.')}
    </div>
    <div style="display:flex;justify-content:space-between;margin-top:18px">
      <button class="btn btn-secondary" onclick="onbGo(2)">Back</button>
      <button class="btn" onclick="runPennyDrop('${v.id}')">Submit &amp; run penny drop</button>
    </div>
  </div>`;
}
function onbPenny(v){
  if(state.pennyState==='running'){
    return `<div class="card card-pad center">
      <div class="spinner"></div>
      <h2 style="margin:14px 0 8px">Running penny drop…</h2>
      <p class="muted">₹1 sent to the account. Verifying the name returned by the bank…</p>
    </div>`;
  }
  return `<div class="card card-pad center">
    <div style="font-size:30px">✓</div>
    <h2 style="margin:8px 0">Penny drop complete</h2>
    <p class="muted">₹1 sent to the account. Name returned by bank:</p>
    <p style="font-size:18px;font-weight:700;margin:10px 0">${esc(v.name)}</p>
    <div style="max-width:340px;margin:0 auto;text-align:left">
      <div class="doc-line">${glyph('ok')}<div class="name">Matches account holder name</div></div>
      <div class="doc-line">${glyph('ok')}<div class="name">Matches PAN legal name</div></div>
      <div class="doc-line">${glyph('ok')}<div class="name">Three-way match complete</div></div>
    </div>
    <div style="margin-top:20px"><button class="btn" onclick="finishOnboarding('${v.id}')">Submit for verification →</button></div>
  </div>`;
}
window.runPennyDrop=(id)=>{ const v=vendor(id); if(!v) return;
  const ah=document.getElementById('onbAcctHolder');
  if(ah && ah.value.trim()) v.name=ah.value.trim();
  state.pennyState='running'; state.onbStep=4; render();
  setTimeout(()=>{ state.pennyState='done'; render(); }, 1600);
};
window.onbGo=(s)=>{ state.onbStep=s; state.pennyState=''; render(); };
window.onbRename=(id,val)=>{ const v=vendor(id); if(!v) return;
  v.name = val.trim() || 'New Vendor';
  const t=document.getElementById('onbTitle'); if(t) t.textContent='Onboarding for '+v.name; };
window.finishOnboarding=(id)=>{ const v=vendor(id); v.status='onboarding'; state.pennyState=''; state.activity.unshift({when:'Now',who:'vendor',vendor:v.name,ev:'Onboarding submitted — bank verified'}); toast('Submitted for verification'); nav('#/verify/'+id); };

/* ---- Stage 5: Due diligence / verification ---- */
function verify(id){
  const v=vendor(id); if(!v) return notFound();
  return shell('vendors', `
    <div class="breadcrumb" onclick="nav('#/vendor/${v.id}')">‹ Back to ${esc(v.name)}</div>
    <div class="page-head"><div><h1>${esc(v.name)} — verification</h1><p class="meta">Human review on top of the automated checks.</p></div>
      <span class="badge ${computeRisk(v).level==='Elevated'?'danger':'ok'}">● ${riskLabel(computeRisk(v).level)} (preliminary)</span></div>
    <div class="split">
      <div>
        <div class="card card-pad"><div class="section-label">Document review</div>
          <div class="grid-2">
            <div class="card" style="background:var(--surface-2);padding:26px;text-align:center;color:var(--ink-3)">📄<br>document preview<br><span class="meta">(scrollable)</span></div>
            <div><div class="kv"><span class="k">Name</span><span>${esc(v.name)}</span></div>
              <div class="kv"><span class="k">PAN</span><span>AAxxx1..</span></div>
              <div class="kv"><span class="k">Date</span><span>04/2026</span></div>
              <div class="kv"><span class="k">Scan</span><span class="g-warn">⚠ blurry</span></div></div>
          </div>
          <p class="meta center" style="margin-top:12px">‹ 1 of 6 documents ›</p>
        </div>
        <div class="card card-pad"><div class="section-label">Bank verification</div>
          <div class="doc-line">${glyph('ok')}<div class="name">Name match</div><span class="meta">${esc(v.name)}</span></div>
          <div class="doc-line">${glyph('ok')}<div class="name">IFSC validated</div></div>
          <div class="doc-line">${glyph('ok')}<div class="name">Penny drop</div><span class="meta">passed</span></div>
        </div>
        <div class="card card-pad"><div class="section-label">Compliance &amp; background</div>
          <div class="doc-line">${glyph('ok')}<div class="name">Watchlist screening</div><span class="meta">clear</span></div>
          <div class="doc-line">${glyph('warn')}<div class="name">Adverse media</div><span class="meta">1 hit</span></div>
          <div class="doc-line">${glyph('ok')}<div class="name">Tax ID validation</div><span class="meta">matched</span></div>
        </div>
      </div>
      <div>
        <div class="card card-pad"><div class="section-label">Decision</div>
          <button class="btn btn-ok btn-block" style="margin-bottom:8px" onclick="verifyDecision('${v.id}','approve')">Approve</button>
          <button class="btn btn-danger btn-block" style="margin-bottom:8px" onclick="verifyDecision('${v.id}','reject')">Reject</button>
          <button class="btn btn-secondary btn-block" onclick="toast('Info request sent to vendor')">Request more info</button>
          <div class="divider"></div>
          <div class="section-label">Internal notes</div>
          <textarea placeholder="Notes for the record…"></textarea>
          <button class="btn btn-ghost btn-sm" style="margin-top:10px" onclick="toast('Email drafted to vendor')">✉ Email vendor what needs fixing</button>
        </div>
      </div>
    </div>`);
}
window.verifyDecision=(id,d)=>{ const v=vendor(id); if(d==='reject'){toast('Vendor rejected');v.status='flagged';render();return;} state.activity.unshift({when:'Now',who:state.currentUser.name,vendor:v.name,ev:'Documents verified'}); toast('Verified — proceed to risk review'); nav('#/risk/'+id); };

/* ---- Stage 6: Risk assessment (Isha's framework) ---- */
function yesno(vid,qid,val){
  return `<span style="display:inline-flex;gap:4px;flex:none">
    <button class="btn btn-sm ${val?'btn-ok':'btn-secondary'}" onclick="setRiskAnswer('${vid}','${qid}',true)">Yes</button>
    <button class="btn btn-sm ${!val?'btn-danger':'btn-secondary'}" onclick="setRiskAnswer('${vid}','${qid}',false)">No</button></span>`;
}
function riskReview(id){
  const v=vendor(id); if(!v) return notFound();
  const r=computeRisk(v);
  const levelBadge = r.level==='Elevated' ? `<span class="badge danger">● High</span>` : `<span class="badge ok">● Low</span>`;
  const drv=(key,lbl)=>`<label class="check" style="margin-bottom:9px"><input type="checkbox" ${v.drivers[key]?'checked':''} onclick="setDriver('${v.id}','${key}',this.checked)"> <span>${lbl}</span></label>`;
  const qrow=(q,crit)=>`<div class="doc-line">
      <span class="badge ${crit?'danger':''}" style="flex:none;padding:2px 8px">${crit?'🔴':'⚪'}</span>
      <div class="name" style="font-weight:500">${q.q}</div>${yesno(v.id,q.id,v.riskAnswers[q.id]!==false)}</div>`;
  const domainCards = r.domains.map(dm=>`
    <div class="card card-pad">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
        <b>${dm.name}</b>
        <span class="badge accent">weight ${Math.round(dm.nw*100)}% · supporting score ${Math.round(dm.score)}%</span>
      </div>
      ${dm.critFails.length?`<p class="meta" style="color:var(--danger);margin-bottom:6px">⚠ ${dm.critFails.length} critical control failed → knockout (K1)</p>`:''}
      ${dm.critical.map(q=>qrow(q,true)).join('')}
      ${dm.supporting.map(q=>qrow(q,false)).join('')}
    </div>`).join('');
  const naCards = r.nonApplicable.length ? `<div class="card card-pad"><div class="section-label">Not applicable (excluded, weight redistributed)</div>
      ${r.nonApplicable.map(d=>`<div class="doc-line" style="opacity:.7">${glyph('todo')}<div class="name">${d.name}</div><span class="meta">${d.reason}</span></div>`).join('')}</div>` : '';
  const koBanner = r.knockouts.length ? `<div class="card card-pad" style="border-color:var(--danger);background:var(--danger-soft);margin-bottom:16px">
      <b class="g-bad">⚠ Knockout${r.knockouts.length>1?'s':''} fired → forced High</b>
      ${r.knockouts.map(k=>`<div class="meta" style="margin-top:5px"><b>${k.code}</b> · ${k.text}</div>`).join('')}</div>` : '';
  // Read-only context: what the vendor self-attested during onboarding (DOMAINS / v.declAnswers).
  const da = v.declAnswers || {};
  const ynBadge = val => val===false ? `<span class="badge danger" style="flex:none">No</span>` : `<span class="badge ok" style="flex:none">Yes</span>`;
  const declApplicable = Object.entries(DOMAINS).filter(([,d])=>d.applies(v));
  const declCard = `<div class="card card-pad"><div class="section-label">Vendor declaration · self-attested (context)</div>
      <p class="meta" style="margin:-4px 0 10px">What the vendor claimed at onboarding — verify against your assessment on the left.</p>
      ${declApplicable.length ? declApplicable.map(([,d])=>`
        <div style="margin-bottom:10px"><b class="meta">${d.name}</b>
        ${d.critical.concat(d.supporting).map(q=>`<div class="doc-line"><div class="name" style="font-weight:400">${q.q}</div>${ynBadge(da[q.id])}</div>`).join('')}</div>`).join('')
        : `<p class="meta">Vendor declared no applicable domains.</p>`}</div>`;
  return shell('vendors', `
    <div class="breadcrumb" onclick="nav('#/vendor/${v.id}')">‹ Back to ${esc(v.name)}</div>
    <div class="page-head">
      <div><h1>${esc(v.name)} — risk assessment</h1>
        <p class="meta">Reviewer assessment · applicability → weighted scoring → knockouts</p></div>
      <div style="text-align:right">${levelBadge}<div class="meta" style="margin-top:6px">Overall ${r.overall}% · Low ≥ 75%</div></div>
    </div>
    <div class="split">
      <div>
        ${koBanner}
        ${domainCards}
        ${naCards}
      </div>
      <div>
        <div class="card card-pad"><div class="section-label">Stage 1 · Applicability drivers</div>
          <p class="meta" style="margin:-4px 0 10px">Vendor-declared — adjust if your review differs.</p>
          ${drv('dataAccess','Accesses / processes personal data → DPDP')}
          ${drv('poshApplies','On-site staff & 10+ employees → POSH')}
          ${drv('pmlaApplies','High-value or government-adjacent → PMLA')}
          ${drv('clientBFSI','Client is a BFSI entity → RBI KYV')}
          ${drv('bankMismatch','Bank country ≠ registered country → K2')}
        </div>
        ${declCard}
        <div class="card card-pad"><div class="section-label">Applicable domains &amp; weights</div>
          ${r.domains.map(d=>`<div class="kv"><span class="k">${d.name}</span><span>${Math.round(d.nw*100)}%</span></div>`).join('')}
          <div class="divider"></div>
          <div class="section-label">Reviewing departments</div>
          <div class="chips">${r.departments.map(d=>`<span class="badge accent">${d}</span>`).join('')}</div>
          <p class="meta" style="margin-top:8px">${riskLabel(r.level)} → ${SENIORITY[r.level].join(' → ')} each · ${r.departments.length*SENIORITY[r.level].length} approvals</p>
        </div>
        <div class="card card-pad"><div class="section-label">Decision</div>
          <textarea id="riskNotes" placeholder="Notes * required — especially when overriding a warning"></textarea>
          <div style="display:flex;gap:8px;margin-top:12px">
            <button class="btn btn-ok" onclick="riskDecision('${v.id}','approve')">Approve &amp; route</button>
            <button class="btn btn-danger" onclick="riskDecision('${v.id}','reject')">Reject</button>
          </div>
        </div>
      </div>
    </div>`);
}
window.setRiskAnswer=(vid,qid,val)=>{ const v=vendor(vid); v.riskAnswers[qid]=val; render(); };
window.setDriver=(vid,key,val)=>{ const v=vendor(vid); v.drivers[key]=val; render(); };
window.riskDecision=(id,d)=>{ const n=(document.getElementById('riskNotes')||{}).value; if(!n){toast('Notes are required.');return;} const v=vendor(id);
  if(d==='reject'){v.status='flagged';toast('Rejected');render();return;}
  const r=computeRisk(v);
  state.approvals[id]={};
  state.activity.unshift({when:'Now',who:state.currentUser.name,vendor:v.name,ev:`Risk assessed — ${riskLabel(r.level)} (${r.overall}%)`});
  toast(`${riskLabel(r.level)} · routed to ${r.departments.join(', ')}`);
  nav('#/approve/'+id); };

/* ---- Stage 7: Approval flow — risk-routed (departments × seniority) ---- */
function approval(id){
  const v=vendor(id); if(!v) return notFound();
  const r=computeRisk(v);
  const chain=SENIORITY[r.level];
  const appr = state.approvals[id] || (state.approvals[id]={});
  r.departments.forEach(d=>{ if(appr[d]==null) appr[d]=0; });
  const totalNeeded = r.departments.length*chain.length;
  const totalDone = r.departments.reduce((s,d)=>s+Math.min(appr[d]||0,chain.length),0);
  const allDone = totalDone>=totalNeeded;
  const lanes = r.departments.map(d=>{
    const done=appr[d]||0;
    const steps=chain.map((role,i)=>{
      const st = i<done?'done':(i===done?'current':'todo');
      const dot = st==='done'?'✓':(st==='current'?'◐':'○');
      return `<div class="doc-line">
        <span class="glyph ${st==='done'?'g-ok':(st==='current'?'g-warn':'g-todo')}">${dot}</span>
        <div class="name">${role}</div>
        ${st==='current'?`<button class="btn btn-ok btn-sm" onclick="deptApprove('${v.id}','${d}')">Approve</button>
          <button class="btn btn-danger btn-sm" onclick="toast('Sent back — '+'${d}')">Reject</button>`:''}
        ${st==='done'?`<span class="badge ok">approved</span>`:''}</div>`;
    }).join('');
    return `<div class="card card-pad">
      <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:6px">
        <b>${d}</b><span class="meta">${Math.min(done,chain.length)}/${chain.length}</span></div>
      ${steps}</div>`;
  }).join('');
  return shell('vendors', `
    <div class="breadcrumb" onclick="nav('#/vendor/${v.id}')">‹ Back to ${esc(v.name)}</div>
    <div class="page-head">
      <div><h1>${esc(v.name)} — approvals</h1>
        <p class="meta">Routed from risk · ${r.departments.length} department(s) × ${chain.join(' → ')}</p></div>
      <span class="badge ${r.level==='Elevated'?'danger':'ok'}">● ${riskLabel(r.level)} · ${totalDone}/${totalNeeded} approvals</span>
    </div>
    <div class="split">
      <div>${lanes}</div>
      <div>
        <div class="card card-pad"><div class="section-label">Why this routing</div>
          <p class="meta">Depth (seniority) comes from the risk level; breadth (departments) from the applicable domains.</p>
          <div class="divider"></div>
          <div class="kv"><span class="k">Risk level</span><span>${riskLabel(r.level)}</span></div>
          <div class="kv"><span class="k">Chain / dept</span><span>${chain.join(' → ')}</span></div>
          <div class="kv"><span class="k">Departments</span><span>${r.departments.join(', ')}</span></div>
          <div class="kv"><span class="k">Total approvals</span><span>${totalNeeded}</span></div>
        </div>
        <div class="card card-pad"><div class="section-label">Project end date</div>
          <div class="kv"><span class="k">Ends</span><span>31 Dec 2026 <button class="btn btn-ghost btn-sm" onclick="toast('Adjust end date')">Adjust</button></span></div>
          <p class="meta" style="margin-top:8px">ⓘ You raised this request, so you cannot approve it.</p>
          <p class="meta">ⓘ Bank details cannot be approved by whoever entered them.</p>
        </div>
        ${allDone?`<button class="btn btn-block" onclick="nav('#/activate/${v.id}')">Continue to activation →</button>`
          :`<div class="card card-pad center"><span class="meta">${totalNeeded-totalDone} approval(s) remaining</span></div>`}
      </div>
    </div>`);
}
window.deptApprove=(id,dept)=>{ const appr=state.approvals[id]||(state.approvals[id]={}); appr[dept]=(appr[dept]||0)+1; const v=vendor(id); state.activity.unshift({when:'Now',who:state.currentUser.name,vendor:v.name,ev:`Approved — ${dept}`}); render(); };

/* ---- Stage 8: Activate ---- */
function activate(id){
  const v=vendor(id); if(!v) return notFound();
  const done=v.status==='active';
  return shell('vendors', `
    <div class="container-sm" style="margin:0 auto">
    <div class="card card-pad center" style="padding:44px 30px">
      <div style="font-size:40px">✓</div>
      <h1 style="margin:12px 0 6px">${esc(v.name)} is approved</h1>
      <p class="muted">All approvals complete · all checks verified</p>
      ${done?`<div style="margin-top:22px"><span class="badge ok"><span class="dot active"></span>Active</span>
        <p class="muted" style="margin-top:14px">Now live in your vendor directory.</p>
        <button class="btn" style="margin-top:14px" onclick="nav('#/vendor/${v.id}')">View vendor →</button></div>`
      : `<button class="btn" style="margin-top:22px" onclick="doActivate('${v.id}')">Activate vendor</button>
        <p class="meta" style="margin-top:14px">Activating adds them to your vendor list and lets you raise POs and payments against them. This is a separate, explicit act.</p>`}
    </div></div>`);
}
window.doActivate=(id)=>{ const v=vendor(id); v.status='active'; v.onboarded='Today'; v.risk=v.risk==='—'?'Medium':v.risk; state.approvalFinanceDone=false; state.activity.unshift({when:'Now',who:state.currentUser.name,vendor:v.name,ev:'Activated'}); toast(v.name+' is now Active'); render(); };

/* ---- Stage 10: Activity log ---- */
function activityLog(){
  const rows=state.activity.map(a=>`<tr>
    <td class="meta">${esc(a.when)}</td><td>${esc(a.who)}</td><td><b>${esc(a.vendor)}</b></td>
    <td>${esc(a.ev)}${a.sub?`<br><span class="meta">└ ${esc(a.sub)}</span>`:''}</td></tr>`).join('');
  return shell('activity', `
    <div class="page-head"><div><h1>Activity log</h1><p class="meta">Org-wide — records what did <i>not</i> become a vendor as much as what did.</p></div>
      <button class="btn btn-secondary" onclick="toast('Exported CSV')">Export</button></div>
    <div class="card card-pad" style="margin-bottom:16px">
      <div class="chips">
        <div class="search"><span class="ico">⚲</span><input type="text" placeholder="Search…"></div>
        <span class="chip">Vendor ▾</span><span class="chip">User ▾</span><span class="chip">Event type ▾</span><span class="chip">Date range ▾</span>
      </div>
    </div>
    <div class="card"><div class="table-wrap"><table>
      <thead><tr><th>When</th><th>Who</th><th>Vendor</th><th>Event</th></tr></thead>
      <tbody>${rows}</tbody></table></div></div>`);
}

/* ---- Stage 11: Analytics (parked) ---- */
function analytics(){
  return shell('', `
    <div class="page-head"><h1>Analytics</h1></div>
    <div class="placeholder">
      <div style="font-size:26px;margin-bottom:10px">📊</div>
      <b>Parked — not yet specified</b>
      <p class="muted" style="margin-top:8px;max-width:480px;margin-left:auto;margin-right:auto">When picked up, the activity log is the data source: full funnel including drop-outs, timestamps for cycle time, and urgency for SLA.</p>
    </div>`);
}

function notFound(){ return shell('', `<div class="empty-state"><div class="em">🔍</div><h3>Screen not found</h3><p class="muted">This flow may be deferred.</p><button class="btn" style="margin-top:12px" onclick="nav('#/vendors')">Go to vendors</button></div>`); }

/* ================= Router ================= */
const routes = [
  [/^#?\/?$/, ()=>landing()],
  [/^#\/login$/, loginChooser],
  [/^#\/login\/vendor$/, loginVendor],
  [/^#\/login\/client$/, loginClient],
  [/^#\/login\/client\/reset$/, clientReset],
  [/^#\/login\/google$/, googleAuth],
  [/^#\/signup$/, signup],
  [/^#\/org-setup$/, orgSetup],
  [/^#\/home$/, clientHome],
  [/^#\/admin$/, adminConsole],
  [/^#\/vendors$/, directory],
  [/^#\/vendor\/(.+)$/, m=>profile(m[1])],
  [/^#\/request$/, request],
  [/^#\/invited\/(.+)$/, m=>inviteSent(m[1])],
  [/^#\/onboard\/(.+)$/, m=>onboarding(m[1])],
  [/^#\/verify\/(.+)$/, m=>verify(m[1])],
  [/^#\/risk\/(.+)$/, m=>riskReview(m[1])],
  [/^#\/approve\/(.+)$/, m=>approval(m[1])],
  [/^#\/activate\/(.+)$/, m=>activate(m[1])],
  [/^#\/activity$/, activityLog],
  [/^#\/analytics$/, analytics],
];
function render(){
  const h=location.hash||'#/';
  // In-page anchors (e.g. #lp-about) are not routes — let the browser scroll, don't re-render.
  if(h.startsWith('#') && !h.startsWith('#/') && h!=='#') return;
  for(const [re,fn] of routes){ const m=h.match(re); if(m){ root().innerHTML=fn(m); window.scrollTo(0,0); return; } }
  root().innerHTML=notFound();
}
window.addEventListener('hashchange', render);
render();
</script>
</body>
</html>
````

**⟶ END `vms_prototype.html` ⟵**

---

_This handoff embeds the source verbatim. If you change the prototype, re-generate this file so the embedded copy stays in sync._
