# GateCaptcha

A self-hosted, multi-tenant puzzle-slider CAPTCHA — the GeeTest/hCaptcha-style
"drag the piece into the notch" challenge, run on our own infrastructure
instead of a third-party vendor.

**[Try the live demo →](https://lee3010.github.io/gatecaptcha-demo/)**

<!-- Add a screenshot or GIF of the slider here once you have one, e.g.: -->
<!-- ![GateCaptcha demo](docs/demo.gif) -->

## What it is

Most "slider captcha" packages are either thin (no real server-side puzzle
generation, trivially bypassed) or pull in a third-party SaaS. GateCaptcha is
a real backend service: it generates a unique jigsaw-notch puzzle image per
challenge, verifies the solve server-side (position tolerance + drag-gesture
bot-heuristics), and issues a single-use token your own backend re-validates
before trusting the result — the same two-step model GeeTest/hCaptcha use
under the hood.

It's multi-tenant: any app can be issued its own `CaptchaId`/`CaptchaSecret`
pair and point its client at the shared service, the same way you'd consume
any hosted captcha provider.

## How it works

1. **Challenge** — the client requests a puzzle (public `CaptchaId` only).
   The server cuts a jigsaw-shaped notch out of a background image at a
   random position and returns the background + loose piece. The correct
   position is never sent to the client.
2. **Verify** — the client reports where it dropped the piece, plus its drag
   trail (position samples over time). The server checks the position
   against a tolerance *and* scores the drag trail for bot-like behavior
   (too few points, no timing variance, teleporting instead of dragging).
   One attempt per challenge. On success, it returns a single-use token —
   never the pass/fail reason, never the correct position.
3. **Confirm** — the client app's *own backend* calls a separate,
   secret-gated endpoint, server-to-server, to confirm that token is genuine
   before trusting "the user passed the captcha." The client itself never
   holds the secret.

## Stack

ASP.NET Core 8, EF Core, server-side image generation (SkiaSharp), a
sliding-window per-tenant rate limiter, and a Flutter widget SDK for the
client side.

## Source

The puzzle-generation engine, bot-detection scoring, and multi-tenant
backend are closed-source. This repo is a live, unmodified demo client —
everything it does, any integrator's own client could do by calling the
same public API. Interested in using GateCaptcha for your own app, or in
licensing details? Reach out.
