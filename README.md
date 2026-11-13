## Strudel Techniques Cookbook (with Drum & Bass Focus)

This guide assumes you already have Strudel running in the browser and you know the basics of writing patterns.

---

## Core Concepts Refresher

- **Pattern**: A time-varying list of values, e.g. notes, samples, or functions.
- **Structure**: Built with combinators like `fast`, `slow`, `stack`, `every`, `rev`, `jux`, `chunk`, etc.
- **Orbit / Channel**: A separate lane of sound (e.g. drums on `orbit:0`, bass on `orbit:1`).

Basic drum pattern:

```js
note("<bd sd bd sd>", { orbit: 0 })
```

---

## Song Sections & Arrangement

### Defining Sections as Functions

Make named functions for each section and switch between them:

```js
const intro = () => stack([
  note("~ bd ~ bd", { orbit: 0, gain: 0.7 }),
  note("~ ~ ~ ~", { orbit: 1 }) // no bass yet
])

const drop = () => stack([
  note("bd*2 [sd sd] bd [sd ~]", { orbit: 0 }),
  note("0 0 ~ 0", { orbit: 1, s: "bass", cps: 2 })
])

const breakdown = () => stack([
  note("[bd ~] ~ ~ ~", { orbit: 0, gain: 0.4 }),
  note("~ ~ ~ ~", { orbit: 1 }),
  note("~ ~ ~ ~", { orbit: 2, s: "pad" })
])

intro()
```

Then manually call `drop()` / `breakdown()` live as you perform.

### Time-Based Section Switching (Simple)

Use `when`-style logic via `every` on a higher-level pattern (if your environment exposes the current cycle):

```js
// high-level: alternate intro and drop every 8 cycles
const song = (cycle) =>
  cycle % 16 < 8 ? intro() : drop()

song
```

If not, just manually change which function you call during performance.

---

## Drum & Bass Drum Patterns

### Classic DnB Kick & Snare

Standard 2-step (kicks on 1 and “and of 3”, snare on 2 and 4):

```js
note("bd ~ ~ ~ bd ~ ~ ~ ~ sd ~ ~ bd ~ ~ sd", { orbit: 0 })
```

Shorter and more readable using grouping:

```js
note("<bd ~ ~ ~ [bd ~] ~ ~> <~ sd ~ ~ ~ sd ~ ~>", { orbit: 0 })
```

Ghost notes and shuffle using random removal:

```js
every(4, (p) => degradeBy(0.4, p),
  note("bd ~ ~ [~ bd] ~ sd ~ ~ bd ~ ~ [sd ~] ~ sd ~", { orbit: 0 })
)
```

### Hi-Hats & Shuffles

16th-note hats with random accents:

```js
note("hh*16", { orbit: 0, gain: 0.7 })
  .map(ev => ev.with({
    gain: ev.gain * choose([0.4, 0.6, 1.0]),
    pan: rand2()
  }))
```

For shuffle:

```js
fast(1.5,
  note("hh*8", { orbit: 0, gain: 0.8 })
)
```

---

## Basslines for Drum & Bass

### Simple Reese-Style Pattern

Assume `s: "bass"` refers to a reese-style synth or sample:

```js
note("<0 0 ~ 0> <~ 0 ~ 0>", {
  orbit: 1,
  s: "bass",
  cps: 2
})
```

Use scale or transposition:

```js
const bassPattern = note("0 [0 3] ~ ~ [5 3] 0 ~ ~", {
  orbit: 1,
  s: "bass",
  cps: 2,
  scale: "minor"
})

bassPattern
```

### Rhythmic Bass Chops

Use `chunk` to rearrange chunks of a bass pattern:

```js
chunk(4, rev,
  note("0 ~ [0 3] ~ [5 3] 0 ~ ~", {
    orbit: 1,
    s: "bass",
    cps: 2
  })
)
```

Or `jux` to flip variants:

```js
jux(rev,
  note("0 ~ [0 3] ~ [5 3] 0 ~ ~", {
    orbit: 1,
    s: "bass"
  })
)
```

---

## Building Full DnB Grooves (Stacking)

### Minimal Groove

```js
stack([
  // drums
  note("bd ~ ~ [~ bd] ~ sd ~ ~ bd ~ ~ [sd ~] ~ sd ~", { orbit: 0 }),

  // hats
  note("hh*16", { orbit: 0, gain: 0.6 })
    .map(ev => ev.with({ gain: ev.gain * choose([0.4, 1]) })),

  // bass
  note("0 [0 3] ~ ~ [5 3] 0 ~ ~", {
    orbit: 1,
    s: "bass",
    cps: 2,
    scale: "minor"
  })
])
```

### Add FX / Atmos

```js
stack([
  note("bd ~ ~ [~ bd] ~ sd ~ ~ bd ~ ~ [sd ~] ~ sd ~", { orbit: 0 }),
  note("hh*16", { orbit: 0, gain: 0.6 })
    .map(ev => ev.with({ gain: ev.gain * choose([0.4, 1]) })),
  note("0 [0 3] ~ ~ [5 3] 0 ~ ~", {
    orbit: 1,
    s: "bass",
    cps: 2,
    scale: "minor"
  }),

  slow(2,
    note("0 ~ 5 ~ 3 ~ 7 ~", {
      orbit: 2,
      s: "pad",
      gain: 0.4
    })
  )
])
```

---

## Arrangement Tricks & Live Performance

### Gradual Build-Ups

Speed up drums every 8 cycles:

```js
every(8, fast(2),
  note("bd ~ ~ [~ bd] ~ sd ~ ~ bd ~ ~ [sd ~] ~ sd ~", { orbit: 0 })
)
```

Start with heavily degraded hats, then tighten:

```js
every(16, (p) => degradeBy(0.5, p),
  note("hh*16", { orbit: 0, gain: 0.9 })
)
```

### Filters / Automation via Param Patterns

If your synth responds to `cutoff` / `res`:

```js
note("0 [0 3] ~ ~ [5 3] 0 ~ ~", {
  orbit: 1,
  s: "bass"
})
.map(ev => ev.with({
  cutoff: range(0.2, 0.9, saw(8)),
  res: 0.7
}))
```

---

## Useful General Strudel Techniques

### Humanization

Timing humanization (if supported as `nudge`):

```js
note("hh*16", { orbit: 0 })
  .map(ev => ev.with({
    nudge: rand2() * 0.01
  }))
```

Velocity / gain humanization:

```js
note("bd ~ bd [~ bd] sd ~", { orbit: 0 })
  .map(ev => ev.with({
    gain: ev.gain * choose([0.8, 1, 1.1])
  }))
```

### Polyrhythms / Polymeter

```js
stack([
  fast(1, note("bd ~ ~ [~ bd] ~ sd ~ ~", { orbit: 0 })),
  fast(3/2, note("hh*8", { orbit: 0, gain: 0.6 })),
  fast(5/4, note("0 ~ 0 ~ 3 ~ 0 ~", { orbit: 1, s: "bass" }))
])
```

### Call & Response

```js
const bassA = note("0 0 ~ 3 ~ 0 ~ ~", { orbit: 1, s: "bass" })
const bassB = note("5 ~ 3 ~ 0 0 ~ ~", { orbit: 1, s: "bass" })

every(4, (p) => stack([p, fast(1, bassB)]), bassA)
```

---

## Simple Song Template (DnB)

```js
const drumsIntro = () =>
  stack([
    slow(2, note("bd ~ ~ [~ bd] ~ sd ~ ~", { orbit: 0 })),
    degradeBy(0.7, note("hh*16", { orbit: 0, gain: 0.6 }))
  ])

const drumsFull = () =>
  stack([
    note("bd ~ ~ [~ bd] ~ sd ~ ~ bd ~ ~ [sd ~] ~ sd ~", { orbit: 0 }),
    note("hh*16", { orbit: 0, gain: 0.8 })
  ])

const bassMain = () =>
  note("0 [0 3] ~ ~ [5 3] 0 ~ ~", {
    orbit: 1,
    s: "bass",
    cps: 2,
    scale: "minor"
  })

const pads = () =>
  slow(2, note("0 ~ 5 ~ 3 ~ 7 ~", { orbit: 2, s: "pad", gain: 0.4 }))

const intro = () => stack([drumsIntro(), pads()])
const drop = () => stack([drumsFull(), bassMain(), pads()])

// start with intro()
intro()
```
