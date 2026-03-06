# Timers & Animation Reference

## Timers (time-based, milliseconds)

```lua
import "CoreLibs/timer"

-- MUST call every frame:
playdate.timer.updateTimers()
```

### Basic timer
```lua
-- Fire callback after delay ms
local t = playdate.timer.new(1000, function()
    print("1 second passed!")
end)

-- Repeating
local t = playdate.timer.new(500, callback)
t.repeats = true
```

### Value timer (interpolation)
```lua
-- Animate a value from startVal to endVal over duration ms
local t = playdate.timer.new(500, 0, 100)
-- Read t.value in update to get current interpolated value

t.timerEndedCallback = function(timer)
    print("finished, final value:", timer.value)
end
```

### Easing
```lua
import "CoreLibs/easing"
-- Pass easing function as 4th arg to value timer:
local t = playdate.timer.new(500, 0, 100, playdate.easingFunctions.outBounce)
local t = playdate.timer.new(500, 0, 100, playdate.easingFunctions.inOutCubic)

-- Available easing functions (all in playdate.easingFunctions):
-- linear, inQuad, outQuad, inOutQuad
-- inCubic, outCubic, inOutCubic
-- inQuart, outQuart, inOutQuart
-- inSine, outSine, inOutSine
-- inExpo, outExpo, inOutExpo
-- inCirc, outCirc, inOutCirc
-- inBounce, outBounce, inOutBounce
-- inBack, outBack, inOutBack
-- inElastic, outElastic, inOutElastic
```

### Key-repeat timer (held button navigation)
```lua
-- Fires immediately, then after 300ms, then every 100ms
local t = playdate.timer.keyRepeatTimer(function()
    moveMenuSelection()
end)

-- Custom delays
local t = playdate.timer.keyRepeatTimerWithDelay(300, 100, callback)

-- Typical usage with button:
local keyTimer
function playdate.downButtonDown()
    keyTimer = playdate.timer.keyRepeatTimer(function()
        selectedIndex += 1
    end)
end
function playdate.downButtonUp()
    keyTimer:remove()
end
```

### Timer control
```lua
t:pause()
t:start()       -- resume; not needed for new timers (auto-start)
t:remove()      -- delete timer
t:reset()       -- restart from beginning

-- Properties
t.duration      -- total duration in ms
t.value         -- current value (value timers)
t.timeLeft      -- ms remaining
t.repeats       -- bool
t.reverses      -- bool: reverses direction after each cycle
t.timerEndedCallback = function(timer) end
t.timerEndedArgs = { arg1, arg2 }  -- passed to callback
```

---

## Frame Timers (frame-based)

Use when timing should be tied to frames rather than real time (e.g., game-speed-dependent effects).

```lua
import "CoreLibs/frameTimer"

-- MUST call every frame:
playdate.frameTimer.updateTimers()

-- Fire after N frames
local ft = playdate.frameTimer.new(30, callback)  -- 30 frames ≈ 1s at 30fps

-- Value frame timer
local ft = playdate.frameTimer.new(60, 0, 100)    -- interpolate over 60 frames
-- ft.value holds current value

-- Same control methods as timer: :pause(), :start(), :remove(), :reset()
-- Same properties: .repeats, .reverses, .timerEndedCallback
```

---

## Animation Loop (sprite frame cycling)

```lua
import "CoreLibs/animation"

-- File: "walk-table-32-32.png" (grid of 32×32 frames)
local imgTable = playdate.graphics.imagetable.new("images/walk")

-- new(msPerFrame, imageTable, shouldLoop)
local anim = playdate.graphics.animation.loop.new(100, imgTable, true)

-- Get current frame image (call in sprite:update or playdate.update)
local img = anim:image()
img:draw(x, y)

-- Or set directly on sprite
sprite:setImage(anim:image())

-- Properties
anim.frame         -- current frame index (1-based)
anim.isValid       -- false when non-looping anim has ended
anim.shouldLoop    -- get/set
anim.paused        -- get/set

-- Jump to frame
anim.frame = 1
```

---

## Animator (value/point interpolation along path)

```lua
import "CoreLibs/animator"

-- Animate scalar value
local a = playdate.graphics.animator.new(1000, 0, 100)
-- a:currentValue() → interpolated number

-- Animate point
local p1 = playdate.geometry.point.new(0, 0)
local p2 = playdate.geometry.point.new(200, 120)
local a = playdate.graphics.animator.new(1000, p1, p2)
-- a:currentValue() → playdate.geometry.point

-- With easing
import "CoreLibs/easing"
local a = playdate.graphics.animator.new(500, 0, 100,
    playdate.easingFunctions.outBounce)

-- Along a line segment
local line = playdate.geometry.lineSegment.new(x1, y1, x2, y2)
local a = playdate.graphics.animator.new(1000, line)

-- Along a polygon / arc / etc.
local a = playdate.graphics.animator.new(1000, arc)

-- Check state
a:currentValue()   -- current interpolated value
a:ended()          -- true when animation is complete
a:progress()       -- 0.0–1.0

-- Reset/reverse
a:reset()
a:reset(duration)  -- reset with new duration
```

---

## Blinker (on/off flashing)

```lua
import "CoreLibs/graphics"

local b = playdate.graphics.animation.blinker.new()
b:startLoop()                    -- blink forever
b:start(onMs, offMs, looping, times)

-- In update:
if b.on then
    sprite:setVisible(true)
else
    sprite:setVisible(false)
end
-- or: sprite:setVisible(b.on)

b:stop()
b:remove()
b.on        -- current on/off state
b.running   -- is it running?
```
