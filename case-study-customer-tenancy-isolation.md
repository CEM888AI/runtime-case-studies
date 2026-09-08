# Case study: customer tenancy isolation

**Status: TESTED**

## Problem

A multi-tenant product's core promise is that a signed-in customer's browser shows that customer's world and nothing else. Isolation layers — session fences, HttpOnly cookies, namespaced client storage — can all be in place and correct, and the promise can still break on pages those layers were never asked to protect.

## Symptom

Live QA as a brand-new, freshly provisioned tenant (real signup, real session, real browser) surfaced a class of defect that months of prior code review had not: the customer's Profile page rendered the **founder's** account — founder identity handle, founder credit balance, the founder's agent list, and a fabricated usage ledger and API key. "Back to dashboard" from that page routed toward the *owner's* dashboard, where the session fence bounced the customer's perfectly valid session into a login wall dead end. A hardcoded founder display name also lurked in chat-shell "self" detection, ready to mislabel rooms in any non-founder tenant.

## Root cause

The leaking pages were **single-user, owner-era templates that predate multi-tenancy**. They made zero server calls — they rendered entirely from client-side storage, and their seed logic *fabricated the owner profile* whenever no local record existed. For a brand-new customer, no local record ever exists, so the owner's mock was served to every stranger who opened the page. A second, compounding gap: dashboard-URL resolution never considered a signed-in customer, and the login page had no "is there already a live session?" check — so every bounce off a gated owner page was a dead end.

The trap for reviewers: the *new* tenancy code was sound. The leaks lived in legacy owner-era pages that predate the architecture — out of scope for a review of the new layer, yet every one reachable from a customer's session.

## Engineering constraint

The fix had to enforce tenancy **in-page, before any content could render** — a customer must never see even a flash of another tenant's data, which means the boot sequence of the owner-era pages had to wait on a session verdict. And the verification had to happen as a real tenant in a real browser: isolation is a runtime property, and code review cannot prove the page a stranger's browser actually renders.

## Solution

A session-verdict guard on the owner-era pages: on load, before render, ask the session endpoint who this visitor is. A customer session redirects to the customer's own dashboard before a pixel of owner content can render; an owner or anonymous visitor sees the page as before. The login page gained a live-session pre-check that auto-forwards signed-in users to the correct dashboard instead of stranding them at the form. Chat shells now resolve "self" dynamically from the viewer's own membership event rather than a hardcoded founder name.

## Measurement

Re-verified live as the same fresh tenant after deploy:

- Profile click and direct-URL attempts at owner pages now land on the **customer's own dashboard**
- DOM scans return zero founder markers (identity handle, agent names, credit balance) across every customer-reachable surface
- Client storage keys are tenant-namespaced; the only chat room visible is the customer's own
- Owner and anonymous flows are unchanged; anonymous visitors still see the login form and the intended demo views

Four defects were found in that single QA session — two critical — each root-caused, fixed, committed, deployed, and re-verified live in one pass. The two critical leaks were both in legacy single-user pages invisible to an architecture review of the new tenant layer.

## Why it matters for agents

Multi-tenant isolation is a **runtime property**. It cannot be proven by reading code — only by *being* a fresh tenant and clicking everything. Legacy single-user-era pages are where cross-tenant ghosts live, and they hide from exactly the review work coding agents do best. The agent that wins on live-product QA is the one that can act: hold a real session, click real buttons, read the rendered DOM for another tenant's markers — then push the fix and prove it landed by living in the customer's chair again.

---
[← All case studies](./README.md)
