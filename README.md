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

It renders at 30fps on purpose. The camera drifts at 0.02 rad/s, so nothing on
screen moves fast enough to need 60, and holding half the frame rate halves the
GPU cost for no visible difference.

Quality is **measured on the visitor's machine, never assumed from it.** Everyone
starts mid-ladder, and the page then climbs as high as the device actually
sustains and stops:

| Tier | Scale | Steps |
| ---- | ----- | ----- |
| 0    | 1.30× | 260   |
| 1    | 1.00× | 230   |
| 2    | 0.82× | 195   |
| 3    | 0.66× | 165   |
| 4    | 0.52× | 135   |
| 5    | 0.40× | 105   |

A workstation settles at tier 0 with supersampling. A weak laptop settles near
the bottom. Neither is detected or special-cased, and there is no user-agent
sniffing anywhere in the file.

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
