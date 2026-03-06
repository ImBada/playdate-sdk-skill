---
name: playdate-sdk
description: >
  Playdate SDK (v3.0.3) expert for Lua game development. Use this skill whenever
  the user asks anything about Playdate game development: writing game code, using
  the Playdate API, debugging, project structure, graphics, sprites, collisions,
  sound, crank input, timers, file I/O, CoreLibs, performance, or building/running
  games. Trigger for prompts like "Playdate 게임 만들어줘", "how do I use sprites in
  Playdate", "크랭크 입력 처리", "Playdate 충돌 감지", "플레이데이트 SDK", "pdx", "pdc compiler",
  "Playdate Simulator", etc. If the user is working on a Lua game for a small
  handheld console with a crank, always use this skill.
---

# Playdate SDK Expert (v3.0.3)

You are an expert Playdate SDK developer.

**SDK path**: `/Users/bada/Developer/PlaydateSDK/`

In Cowork sessions the SDK is mounted — find the active path with:
```bash
find /sessions -name "Inside Playdate.html" 2>/dev/null | head -1
```
Use that directory as the SDK root for reading docs and examples.

## Reference files (read when relevant to the task)

| Topic | File |
|-------|------|
| Drawing, images, text, tilemaps | [graphics.md](references/graphics.md) |
| Sprites, OOP, collision detection | [sprites.md](references/sprites.md) |
| Buttons, crank, accelerometer | [input.md](references/input.md) |
| Sound effects, music, synth | [sound.md](references/sound.md) |
| Timers, animation loops, easing | [timers-animation.md](references/timers-animation.md) |
| Save data, file I/O, system callbacks, menus, pathfinding | [data-system.md](references/data-system.md) |

For questions spanning multiple areas, read the relevant files. For detailed API lookups, also check the live SDK docs:
- `Inside Playdate.html` — full Lua API (parse with Python html.parser)
- `CoreLibs/*.lua` — source of all CoreLibs
- `Examples/Single File Examples/*.lua` — working code examples

---

## Device Specs

- **Display**: 400×240 px, 1-bit monochrome (black/white only), 30fps default (max 50fps)
- **Memory**: 16MB RAM
- **Controls**: D-pad, A/B buttons, Menu button, Lock button, **Crank** (unique!), Accelerometer
- **Sound**: Speaker, mic, headphone jack
- **Connectivity**: Wi-Fi, Bluetooth

---

## Project Structure

```
MyGame/
  source/
    main.lua          ← required entry point
    pdxinfo           ← game metadata
    images/           ← .png → compiled to .pdi
    sounds/           ← ADPCM .wav or .mp3
  MyGame.pdx/         ← compiled output
```

**pdxinfo:**
```
name=My Game
author=Dev Name
bundleID=com.yourname.mygame
version=1.0
buildNumber=1
```

**Build:**
```bash
export PLAYDATE_SDK_PATH=/Users/bada/Developer/PlaydateSDK
pdc source/ MyGame.pdx
```

---

## Game Loop Skeleton

```lua
import "CoreLibs/graphics"
import "CoreLibs/sprites"
import "CoreLibs/timer"

local gfx = playdate.graphics

-- One-time setup
local player = gfx.sprite.new(gfx.image.new("images/player"))
player:moveTo(200, 120)
player:add()

-- Called every frame (~30fps)
function playdate.update()
    gfx.sprite.update()
    playdate.timer.updateTimers()
end
```

**Key rules:**
- Use `import` not `require` (each file compiled once; second import does nothing)
- Always use `local` for variables (globals are slow in Lua)
- `local gfx = playdate.graphics` alias at top of every file
- Never create tables/objects in the hot `update()` loop (GC pressure)
- Simulator runs much faster than real hardware — test on device often

---

## Performance Tips

- **Prefer sprites** over manual `gfx` drawing — dirty-rect redraws only changed areas
- **Image tables** for animation: `walk-table-32-32.png` = grid of 32×32 frames
- **ADPCM .wav** for SFX: best size/performance balance
- **`setRefreshRate(50)`**: only if you need max fps
- **`local gfx = playdate.graphics`**: local alias avoids global table lookup every frame
- **Avoid GC in hot paths**: reuse tables, don't create new objects each frame
- **`playdate.display.setInverted(true)`**: for white-background games

---

## Asset Formats

| Type | Format | Notes |
|------|--------|-------|
| Images | `.png` (1-bit) | Compiled to `.pdi` by pdc |
| Image tables | `name-table-WxH.png` | Grid of same-size frames, auto-loaded |
| Fonts | `.fnt` + image | `gfx.font.new("fonts/MyFont")` |
| SFX | ADPCM `.wav` | Recommended |
| Music | `.mp3` or `.wav` | Fileplayer only for MP3 |

---

## Quick Patterns

**D-pad movement:**
```lua
local dx, dy = 0, 0
if playdate.buttonIsPressed(playdate.kButtonRight) then dx = 3 end
if playdate.buttonIsPressed(playdate.kButtonLeft)  then dx = -3 end
if playdate.buttonIsPressed(playdate.kButtonDown)  then dy = 3 end
if playdate.buttonIsPressed(playdate.kButtonUp)    then dy = -3 end
player:moveBy(dx, dy)
```

**Crank as direction:**
```lua
local angle = playdate.getCrankPosition()  -- 0–360°
local dx = math.sin(math.rad(angle))
local dy = -math.cos(math.rad(angle))
```

**Save on exit:**
```lua
function playdate.gameWillTerminate()
    playdate.datastore.write(saveData)
end
function playdate.gameWillSleep()
    playdate.datastore.write(saveData)
end
```

**Screen shake:**
```lua
local shake = 0
function playdate.update()
    if shake > 0 then
        gfx.setDrawOffset(math.random(-shake, shake), math.random(-shake, shake))
        shake = math.max(0, shake - 1)
    else
        gfx.setDrawOffset(0, 0)
    end
    gfx.sprite.update()
end
```
