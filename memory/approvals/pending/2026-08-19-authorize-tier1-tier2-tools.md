## What

Authorize and set up the Tier 1/2 tools recommended in `TOOL_STACK.md`:

1. **Buffer** — free account, then OAuth-link whichever platform account
   comes out of issue #1. Unlocks me actually publishing via API/MCP
   instead of manual copy-paste, permanently.
2. **A Google AI Studio / Gemini API key** — free to create, pay-per-image
   billing (~$0.02/image) for actual visual content.
3. Optional, not urgent: **Shotstack** and **ElevenLabs** free-tier
   signups, only once there's a specific video/voice piece to make.

## Why

Full research in `TOOL_STACK.md` and
`memory/decisions/0004-tool-stack-strategy.md`. Short version: these are
the cheapest tools that unlock real capability — Buffer solves the
publishing bottleneck for $0, and pay-as-you-go image generation is a
fraction of a cent per output instead of a $15-40/mo subscription.

## Cost

$0 to set up. Actual usage cost is pay-as-you-go and tiny at this scale —
generating a handful of test images would cost cents, not dollars. I'm not
proposing an actual spend yet, just the account/key setup; real generation
spend would go through `update-ledger` and, above threshold, a separate
approval.

## Expected upside

Removes the two biggest bottlenecks blocking autonomous operation:
publishing (Buffer) and visual content (image API). Both fit comfortably
inside the $100 budget instead of consuming a third of it in month one.

## Potential downside

Low. Free signups, no commitment. The only real downside is the OAuth-link
step needs a live platform account to link to (still gated on issue #1),
so Buffer setup can happen in parallel but won't be *useful* until that
account exists.

## What happens if we do nothing

I keep operating exactly as I am now — text drafts, human posts them
manually. Not broken, just slower and more manual than it needs to be.

## Exact steps (updated — the integration code is already built and waiting)

**Buffer:**
1. Create a free account at https://buffer.com
2. Once issue #1's platform account exists, OAuth-connect it to Buffer
   from Buffer's dashboard
3. Generate an API token at https://developers.buffer.com
4. Add it as an environment variable named `BUFFER_API_KEY` on this
   Claude Code environment — not by pasting it into chat. Environment
   settings: https://code.claude.com/docs/en/cloud-environments#set-environment-variables

**Gemini/Imagen:**
1. Go to https://aistudio.google.com/apikey, sign in, create an API key
   (may require enabling billing on the associated Google Cloud project
   past a small free quota)
2. Add it as an environment variable named `GEMINI_API_KEY` the same way

Once either variable is set, the `publish-buffer` and `generate-image`
skills (already built, `.claude/skills/`) go from inert to usable on the
next wake — no further engineering needed on my end.

## GitHub issue

https://github.com/jacksonbgunther/Claudetherobot/issues/2
