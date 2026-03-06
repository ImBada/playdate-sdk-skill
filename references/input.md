# Input Reference

## Buttons

Constants: `playdate.kButtonA`, `kButtonB`, `kButtonUp`, `kButtonDown`, `kButtonLeft`, `kButtonRight`

### Polling (inside `playdate.update`)
```lua
if playdate.buttonIsPressed(playdate.kButtonA) then end    -- held down
if playdate.buttonJustPressed(playdate.kButtonA) then end  -- first frame only
if playdate.buttonJustReleased(playdate.kButtonA) then end -- release frame only

-- All states at once (bitmask)
local current, pressed, released = playdate.getButtonState()
if current & playdate.kButtonA ~= 0 then ... end
if pressed & playdate.kButtonA ~= 0 then ... end
```

### Callbacks (global functions)
```lua
function playdate.AButtonDown()  end   -- pressed
function playdate.AButtonUp()    end   -- released
function playdate.AButtonHeld()  end   -- held ≥1 second (good for "alt action")

function playdate.BButtonDown()  end
function playdate.BButtonUp()    end
function playdate.BButtonHeld()  end

function playdate.upButtonDown()    end
function playdate.upButtonUp()      end
function playdate.downButtonDown()  end
function playdate.downButtonUp()    end
function playdate.leftButtonDown()  end
function playdate.leftButtonUp()    end
function playdate.rightButtonDown() end
function playdate.rightButtonUp()   end
```

### Input Handlers (OOP-friendly alternative to global callbacks)
```lua
local myHandler = {
    AButtonDown = function() print("A pressed") end,
    AButtonUp   = function() print("A released") end,
    cranked     = function(change, accel) print("crank:", change) end,
}
playdate.inputHandlers.push(myHandler)
playdate.inputHandlers.pop()  -- remove when done (e.g. on scene change)
```

---

## Crank

The crank is Playdate's most unique input — a physical wheel on the right side.

```lua
-- Change since last frame (degrees, positive = clockwise)
local change = playdate.getCrankChange()

-- Absolute position (0–360 degrees, clockwise from 12 o'clock)
local angle = playdate.getCrankPosition()

-- Is crank tucked away?
if playdate.isCrankDocked() then
    -- show "please extend crank" UI, or use buttons as fallback
end

-- Callbacks
function playdate.cranked(change, acceleratedChange)
    -- change: degrees this frame
    -- acceleratedChange: amplified when spinning fast (good for menus)
end
function playdate.crankDocked()    end  -- crank put away
function playdate.crankUndocked()  end  -- crank pulled out

-- Suppress system crank sounds
playdate.setCrankSoundsDisabled(true)
```

### Common crank patterns

**Crank as dial / pointer:**
```lua
local angle = 0
function playdate.update()
    angle = (angle + playdate.getCrankChange()) % 360
    -- convert to radians for sin/cos:
    local rad = math.rad(angle)
    local dx = math.sin(rad)
    local dy = -math.cos(rad)  -- -y because screen y increases downward
end
```

**Crank as scroll / speed control:**
```lua
local scrollY = 0
function playdate.update()
    scrollY = scrollY + playdate.getCrankChange() * 2
    playdate.graphics.setDrawOffset(0, -scrollY)
end
```

**Crank indicator UI** (tell player to use crank):
```lua
import "CoreLibs/ui"
playdate.ui.crankIndicator:start()
-- in update:
playdate.ui.crankIndicator:update()  -- draws animated crank hint
```

---

## Accelerometer

```lua
playdate.startAccelerometer()     -- must call before reading

local x, y, z = playdate.readAccelerometer()
-- x: left(-) / right(+) tilt
-- y: up(-) / down(+) tilt (positive toward bottom of screen)
-- z: front(+) = screen facing up; back(-) = screen facing down
-- Device held upright: (0, 1, 0)
-- Device flat on back: (0, 0, 1)

playdate.stopAccelerometer()      -- save battery when not needed
playdate.accelerometerIsRunning() -- true/false
```

**Tilt-to-move example:**
```lua
playdate.startAccelerometer()
function playdate.update()
    local gx, gy, _ = playdate.readAccelerometer()
    x = math.max(0, math.min(400, x + gx * 8))
    y = math.max(0, math.min(240, y + gy * 8))
end
```
