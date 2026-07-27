# Gargantua

A Schwarzschild black hole, raytraced in real time in the browser.

**Live:** https://omkarbhoite25.github.io/gargantua/

One self-contained HTML file. No build step, no dependencies, no framework.

## What it actually does

Every frame, a WebGL2 fragment shader fires one ray per pixel and integrates it
backwards along a null geodesic:

```
d²x/dλ² = -3/2 · h² · x / r⁵      h = x × v, conserved
```

That single line is where all of it comes from. The lensing, the photon ring,
the halo arcing over the top of the shadow, the second image folded underneath:
none of it is drawn or faked. It falls out of light paths bending in curved
spacetime.

On top of that:

- **Accretion disk** as a plane at `y = 0`. A ray crosses it several times, and
  each crossing is accumulated front to back, which is why you see the far side
  of the disk *above* the shadow and its underside *below*.
- **Novikov-Thorne flux profile** with the innermost stable circular orbit at
  3 R<sub>S</sub>, setting how temperature falls off with radius.
- **Relativistic beaming**, `I = I₀ · δ³` where `δ = 1/(γ(1 − β·d̂))`. This is
  what makes one limb burn white and drops the other into the dark. It is not a
  gradient painted on.
- **Gravitational redshift**, `g = √(1 − R_S/r)`.
- **Keplerian shear.** The turbulence is sampled in a frame co-rotating at the
  local orbital rate, so the disk's filaments wind themselves into spirals
  instead of turning rigidly like a printed disc.
- Procedural starfield, which lenses correctly because the rays that reach it
  have already been bent.
- Two-scale anamorphic bloom, ACES tonemap, vignette, grain floor.

## Performance

Quality is **measured on the visitor's machine, never assumed from it.** Everyone
starts mid-ladder, and the page then climbs as high as the device actually
sustains and stops. There is no user-agent sniffing anywhere in the file.

| Tier | Scale | Steps | FPS |
| ---- | ----- | ----- | --- |
| 0    | 1.60x | 300   | 30  |
| 1    | 1.30x | 275   | 30  |
| 2    | 1.00x | 240   | 30  |
| 3    | 1.00x | 240   | 20  |
| 4    | 0.82x | 205   | 20  |
| 5    | 0.66x | 175   | 20  |
| 6    | 0.52x | 145   | 20  |
| 7    | 0.40x | 110   | 20  |

Scale multiplies *device* pixels, so 1.00 is already native on a retina panel
and 1.60 is real supersampling on top of that.

Two things in that table are deliberate:

- **Tier 3 holds native resolution and spends frame rate instead.** For a
  background whose camera drifts at 0.02 rad/s, 20fps is nearly impossible to
  notice and a soft upscale is impossible to miss. Resolution is surrendered
  last, not first.
- **Every frame rate is a divisor of 60.** The frame gate can only release on a
  vsync boundary, so asking a 60Hz panel for 24fps actually delivers 20 and
  leaves the ladder judging itself against a target it can never reach.

Throughput is measured over a full second against the wall clock, not as the gap
between two frames. That gap is quantised to the vsync interval, so it will
happily read a healthy 33.3ms while the GPU is only completing 16 frames a
second.

Also: the render pauses when the tab is hidden, survives WebGL context loss, and
widens its field of view as the frame narrows so the whole system stays in shot
on a portrait phone.

## Fallback

`gargantua.webp` is shown immediately on load, so the first paint is never black,
and it stays put if WebGL2 is unavailable.

## Tuning

`window.__gq` is exposed for profiling:

```js
__gq.tier          // current tier
__gq.w, __gq.h     // render resolution
__gq.set(0)        // pin a tier
```

## Licence

MIT.
