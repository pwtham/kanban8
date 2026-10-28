# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT PMO Kanban demo (`index.html`) for a fictitious bank. There is no build, lint, or test tooling and no package manager. Run it by opening `index.html` in a browser (double-click; no server).

## Hard constraints (from the original brief)

- Vanilla HTML/CSS/JS only, all in `index.html` (one `<style>`, one `<script>`). No frameworks, CDNs, fonts, or image files; icons are Unicode/inline SVG.
- No persistence of any kind: no localStorage/sessionStorage/IndexedDB/cookies. A refresh resets to seeded data by design (the header says so).
- The only network call is the FormSubmit AJAX endpoint in `FORMSUBMIT_ENDPOINT`. The email address must appear nowhere else.
- No `!important`; no native `alert()`/`confirm()` (inline errors and an inline "Delete? Yes / No" instead).
- No real bank logo/branding; text wordmark "IT PMO" and a blue palette only. The `UOB-ITPM-####` ID prefix and `[UOB IT PMO]` email subject were specified verbatim.

## Architecture

Everything flows one way: events mutate `state` (`{ tasks, filters, confirmingId, nextId }`), then `renderBoard()` rebuilds the board from scratch via `innerHTML`.

- Card and column DOM is built only in `renderCard()` / `renderBoard()` / `renderSummary()`. Do not mutate card DOM elsewhere.
- Every user-supplied string must go through `escapeHtml()` before entering an HTML string.
- Board interaction uses event delegation on `#board` (set up in `initBoardEvents()`). Because the board is re-rendered wholesale, listeners must not be attached to individual cards. Focus is restored after re-render with `focusInCard()`.
- Drag and drop is native HTML5; the keyboard fallback is the per-card "Move" `<select>`. Both go through `moveTask()`.
- Delete confirmation is state-driven (`state.confirmingId`), not DOM-driven.
- `addTask()` is optimistic: the task is added and rendered first, then `notifyNewTask()` runs; failure only shows a warning toast and must never break the board. The submit button is disabled with "Sending…" while the request is in flight.
- Seed due dates are relative to today (`offsetDate()`), so the Overdue badges appear whenever the page loads.
