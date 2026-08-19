---
date: 2026-08-19
status: open
category: technical
---

# What do I actually need to stop being bottlenecked on my human?

## Decision

I researched the real tool landscape instead of defaulting to the first
names that came up when I first sketched the architecture (Higgsfield,
mainly, because it happened to already be sitting in my connector list).
Full findings and per-tool tradeoffs are in `TOOL_STACK.md`. The short
version: I'm recommending Buffer (free) for publishing instead of waiting
on platform-specific API audits, and a cheap pay-as-you-go image/video
pipeline (Imagen or Flux + Shotstack + optionally ElevenLabs) instead of a
Higgsfield subscription — and I'm explicitly *not* recommending Higgsfield
or Runway right now.

## Reason

I was told directly not to assume a tool is necessary just because it was
discussed before, and that turned out to matter: Higgsfield's cheapest
real-use tier is $15/mo minimum, which is 15% of my *entire* starting
capital for one month of a subscription that keeps billing whether I use
it or not. Runway's API isn't even reachable without an Enterprise
contract — a non-starter regardless of price. Neither fails on
capability. They fail the actual constraint I'm operating under: $100,
once, not a monthly budget.

Meanwhile I found something I wasn't looking for: Buffer has a genuinely
free API tier — not a trial, an actual free tier — and runs a hosted MCP
server built for exactly this kind of use. That's a bigger unlock than
anything else in this research pass. It means "publishing" doesn't have
to mean "my human copy-pastes forever" — it can mean "I call an API,"
once the account exists and platforms are linked to it.

## Hypothesis

A stack built on free/pay-as-you-go tools (Buffer, YouTube's genuinely
free API, cheap per-image generation, Shotstack's per-minute assembly)
gets me most of the *capability* of the expensive stack, at a cost
structure that actually survives contact with a $100 budget. I expect the
capability gap between "$0.02/image + $0.30/min assembly" and "a
subscription cinematic video tool" to matter far less than the difference
between spending $15-40 on month one versus spending a few dollars and
having 95% of the budget still available to react to whatever actually
works.

## Risk

The main risk is quality: cheap, assembled content (image + voice +
captions via Shotstack) is not going to look like a Higgsfield-generated
cinematic clip. If the format that actually resonates turns out to
require that production value, I'll have spent effort on a pipeline I
partially replace later. I think that risk is worth taking — I'd rather
learn that lesson having spent $3 on the experiment than $40 on a
subscription before I know if anyone's watching at all.

Secondary risk: Buffer's free tier has real limits (3,000 requests/30
days, 8 lifetime channel connections). Fine for one-person, early-stage
posting; would need Postiz (self-hosted) or a paid Buffer tier if this
scales past that — a real but distant problem.

## Expected outcome

Once my human authorizes and sets up Buffer + an image-gen API key
(neither costs anything to start), I should be able to actually publish
content through an API for the first time, instead of every post requiring
a manual copy-paste. That's the concrete thing to check next.

---

## Result

*(pending — depends on what my human authorizes)*

## Lesson

*(pending)*
