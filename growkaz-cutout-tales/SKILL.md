---
name: "growkaz-cutout-tales"
description: "Make narrated 2D cutout/puppet-style story scenes (flat vector adult characters, layered dusk sets, slow camera push-ins, pop-in name tags, stamps, star ratings, speech bubbles) rendered to MP4 for Grow With Kaz."
---

# GrowKaz Cutout Tales

A Grow With Kaz video style: "What if you were born as…" storybook explainers in flat-vector **cutout / puppet animation**. Characters are layered rigs that barely move, and the camera and overlays carry the energy. Everything is drawn in code (Python + pycairo) and piped to ffmpeg, so it renders locally with no assets.

Use it when the user asks for a GrowKaz cutout scene, a "born as" style story, a western, crime or historical life story in this look, or to recreate the 20-second Silas scene.

## Bundled files (use these instead of writing from scratch)
- `scripts/engine.py`: the core kit. Drawing helpers, the parametric character rig (`CAST`, `POSES`, `EXPR`, `character()`), modern sets (apartment, office, street, dinner, stage, rooftop), western accessories (stetson, bandana, holster, sheriff star), icons and every overlay (tag, stamp, bubble, caption, card, counter, bigword, arrow_label).
- `scripts/scene.py`: the original 20s Silas western scene, plus the western buildings, mountains and silhouette horse rider.
- `scripts/align.py`: `python3 align.py script.txt part1.mp3 [part2.mp3 ...]` makes `timing.json` (sentence start and end times found from pauses in the audio). Use it when Whisper is unavailable.
- `scripts/sfx.py`: builds `sfx.wav` automatically by scanning a shot list for overlay events (pop, thud, whoosh, typing ticks, dings, coins, boom) and adds a quiet room tone.
- `examples/money_explainer.py`: a full narrated explainer (two voiceover parts, about 45 shots), with a `Shot` class driven by `timing.json`, `C()` character placement, gestures, talking mouths and `speech()` bubbles. Run `TIMING=timing.json python3 money_explainer.py 1 qc|full`.
- `examples/dustwater_western.py`: a 30s multi-character western with sound effects built in. Run `python3 dustwater_western.py qc|full`.
- Requirements: `pip install pycairo numpy pillow` and ffmpeg. Default font is Poppins (any bold sans works).

Pipeline for a narrated video: align the voiceover → write a shot list modeled on `money_explainer.py` → QC contact sheet → full render → `sfx.py` → mix (SFX at about 0.45 under the voice, then `loudnorm=I=-14`).

## Rules
- **Original characters only.** Never draw a game, film or TV character (no Arthur Morgan, CJ and so on). Invent a character with the same archetype (drifter cowboy, street kid, gang boss) and an original name.
- Default output: 1920x1080, 30fps, H.264 CRF 18 `-preset veryfast`, yuv420p. For 9:16, set W=1080, H=1920 and re-frame the character at about 60% height.
- Always render QC stills (a 2x2 contact sheet) and look at them before the full render. Fix overlaps and off-screen elements, then render.

## Style bible
**Characters (adults)**
- About 7 heads tall, with the head slightly oversized. Mitten hands (circles).
- Outline is dark brown `(0.17,0.11,0.09)` at about 3.2px, never pure black. Round joins.
- Each part is a base fill plus ONE hard-edged shadow tone, clipped to the shape on the side away from the light. No gradients and no textures on characters.
- Face: jaw curve, almond eyes with dark pupils, HEAVY brows (thick round-cap strokes, which carry the emotion), nose as a shadow wedge, a translucent stubble patch, a mustache stroke.
- Detail comes from costume construction: vest, satchel strap, belt with a gold buckle, holster, bandana, a hat as its own layer with a band.
- Every character needs a unique silhouette (hat shape, build, signature color).
- Kids are optional chibi versions: big round head, dot eyes, tiny body.

**Rig layers (draw order):** legs and boots → back arm (shoulder, elbow) → torso shirt → vest panels → strap and belt and holster → neck and bandana → head (ears, face, stubble, nose, eyes, brows, mouth, hair) → hat → front arm. Arms are two segments: `rotate(a1)` at the shoulder, then translate by the upper-arm length and `rotate(a2)` at the elbow. Poses are dicts of angles keyframed with smoothstep.

**Idle motion (always on):** breathing via torso scale `1 + sin(t*2.1)*0.012`, head bob of about 1.6px, a slow head sway of about 0.025 rad, blinks of 0.14s every 3–4s, and a short glance with a brow raise when an overlay appears. Gestures (hat tip, point, wave) take 0.45s to ease in, hold, then 0.45s to ease out.

**Sets:** a sky gradient (3 stops), a sun or moon with a big radial bloom, soft clouds made from circle clusters, 2 mountain ridges, a midground set (false-front western buildings with lit yellow windows, porches and signs), a gradient ground with grass tufts, and a prop or two (barrel, crate).
Mood palettes:
- Dusk: `#3d2a5c → #c4566a → #f6a65a`, mountains `#7d4a72` / `#a65a6a`.
- Night: navy `#141a3a → #2b3566`, a moon, yellow windows, campfire glow.
- Day: `#7fb6e6 → #cfe6f5`, white clouds, green or tan ground.
- Interior: warm browns with a lamp bloom.
Always add drifting dust particles (about 90, alpha pulsing) and a vignette (radial, 0.55–0.7).

**Camera:** every shot has a slow push-in (zoom 1.0→1.13 with smoothstep across the shot) plus a slight drift. Parallax: wrap each layer in `layer(depth, fn)`, which scales by `1+(z-1)*depth` around a focus point and offsets x by `t*6*depth`. Use depth 0.05 for stars, 0.1 for sun, 0.15 for clouds, 0.25 and 0.4 for mountains, 0.6 for town, 0.85 for ground, 1.0 for the character.

**Overlays (the explainer layer):** all pop in with back-out easing (`c=2.2`, about 0.28s), slightly rotated (±0.03–0.08 rad), with a soft drop shadow, in Poppins Bold.
- **Name tag:** a white rounded rect, a 4px outline, a bold 38px title and a grey 20px subtitle ("SILAS / the drifter"). Place it beside the character, not on the face.
- **Stamp:** red double-border box on a cream fill ("BOUNTY $2,500", "SAVED", "ROBBED").
- **Stat rating:** white label plus 5 gold stars that pop in one by one 0.12s apart ("GRIT RATING").
- **Speech bubble:** a white pill with a tail pointing at the mouth and typewriter text over 1s. It shrinks out at the end.
- **Year or age tag:** "1899", "ABOUT 14".
- Also available: hand-drawn arrows with labels, chalkboards, parchment notes, maps, polaroids, a viewer comment box.
- Make sure overlays never overlap each other or the character's head; check this in QC.

**Transitions:** 0.4s fade through black between shots, a 0.5s fade-in at the start and 0.6s fade-out at the end. Silhouette shots (a near-black `(0.08,0.05,0.1)` figure against a giant sun) work for dramatic beats and endings.

## Core helpers (implement these first)
```python
def ease(t): t=clamp(t); return t*t*(3-2*t)
def back_out(t): t=clamp(t); c=2.2; return 1+(c+1)*(t-1)**3+c*(t-1)**2
# scale guard: cairo errors on scale 0 -> c.scale(max(s,0.001), max(s,0.001))
def shade(c, path_fn, shadow_fn, base, dark):
    path_fn(); c.set_source_rgba(*base); c.fill_preserve()
    c.save(); c.clip(); shadow_fn(); c.set_source_rgba(*dark); c.fill(); c.restore()
    path_fn(); c.set_source_rgb(*LINE); c.set_line_width(3.2); c.stroke()
```
Colors are RGBA tuples from a `hexc()` helper. Always use `set_source_rgba(*col)` for them (`set_source_rgb` with 4 values throws).

Frame loop: build a `cairo.ImageSurface(FORMAT_RGB24, W, H)` per frame and write `surf.get_data()` to an ffmpeg stdin pipe using `-f rawvideo -pix_fmt bgra -s WxH -r 30 -i -`. Add a stills mode (`python3 scene.py x 3 8 12.5 17`) that writes PNGs for QC. A 20s 1080p render takes about 50s.

## Workflow
1. **Intake:** get the character concept, setting, length (default 20s), aspect ratio, and any narration script or voiceover file. If a voiceover is given, transcribe it for timestamps and cut the shots to the sentences.
2. **Shot list:** 3–8s per shot. For each shot write the set, mood palette, character pose and gesture, overlays with their pop times, and the transition. Classic arc: epic silhouette open → name tag intro → stat or stamp jokes → speech bubble line → silhouette ride-off.
3. **Build** `scene.py` in the working directory from the style bible and helpers above: palette, `character(c,t,blink,pose)`, set functions, overlay functions, `pose_at(t)`, one `shotN(c,t)` per shot, and `frame(i)` with fades.
4. **QC:** render 4 stills into a 2x2 contact sheet, view it, and fix scale, placement and overlaps.
5. **Render** the MP4. If there's a voiceover or music, mux it with `ffmpeg -i video.mp4 -i audio.m4a -c:v copy -c:a aac -shortest`, with music ducked about 14dB under the voice.
6. **Deliver** with SendUserFile and a one-line caption (resolution, fps, length, audio or no audio).

## Reference scene: "Silas" (20s)
- **0–15s, dusk western town.** Hotel, Saloon, Bank and General buildings. Silas is centered at scale 1.45 with his feet at y=1045. He keeps hands on hips (back arm 0.38/-1.25, front arm -0.38/1.25).
  - 1.2s: name tag "SILAS / the drifter" at (560,380).
  - 4.0s: stamp "BOUNTY $2,500" at (1430,300), with a glance and brow raise from 3.8 to 5.6s.
  - 5.3s: "GRIT RATING" and 5 stars at (1500,160).
  - 7.2–9.3s: hat tip (front arm to -2.5/0.55, hat rotates -0.13, smile).
  - 10.6–14.4s: speech bubble "This town ain't big enough." at (560,150), with stern brows.
- **15–20s:** silhouette rider on a walking horse (sine-driven legs, body bob) crossing in front of a giant setting sun, with fence posts scrolling past, a "1899" tag, and a fade-out.