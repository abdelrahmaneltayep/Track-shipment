# Cashback box animations (options 56–65)

Exports of the ten looping cashback carton animations from the design handoff.
All have a transparent background and loop forever over a 5-second cycle.

| # | Name | GIF | Lottie JSON |
|---|---|---|---|
| 56 | Bounce hops | `gifs/cashback-56-bounce-hops.gif` | `lottie/cashback-56-bounce-hops.json` |
| 57 | Squash & stretch | `gifs/cashback-57-squash-stretch.gif` | `lottie/cashback-57-squash-stretch.json` |
| 58 | Horizontal shiver | `gifs/cashback-58-horizontal-shiver.gif` | `lottie/cashback-58-horizontal-shiver.json` |
| 59 | Rocking | `gifs/cashback-59-rocking.gif` | `lottie/cashback-59-rocking.json` |
| 60 | Jelly wobble | `gifs/cashback-60-jelly-wobble.gif` | `lottie/cashback-60-jelly-wobble.json` |
| 61 | Shiver then hop | `gifs/cashback-61-shiver-then-hop.gif` | `lottie/cashback-61-shiver-then-hop.json` |
| 62 | Spin hop | `gifs/cashback-62-spin-hop.gif` | `lottie/cashback-62-spin-hop.json` |
| 63 | Pulse | `gifs/cashback-63-pulse.gif` | `lottie/cashback-63-pulse.json` |
| 64 | Drop & bounce | `gifs/cashback-64-drop-bounce.gif` | `lottie/cashback-64-drop-bounce.json` |
| 65 | Skew wobble | `gifs/cashback-65-skew-wobble.gif` | `lottie/cashback-65-skew-wobble.json` |

## GIF
- 512×512, 25 fps, 125 frames.
- GIF transparency is on/off only, so edges are not anti-aliased against the
  page and the bills pop in and out instead of fading. Use the Lottie files
  where smooth edges matter.

## Lottie JSON
- Fully vector (shape layers only, no embedded images), 1024×1024, 50 fps,
  250 frames. Plays in lottie-web, lottie-ios, lottie-android and dotLottie.
- Carton shapes are converted from the handoff SVGs. Bill transforms and
  opacity are sampled from the reference CSS keyframes every frame and
  reduced to linear keyframes (within 0.25px / 0.1°), so the motion matches
  the reference to sub-pixel accuracy.
