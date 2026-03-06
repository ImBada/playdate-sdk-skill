# Data, System & Misc Reference

## Save Data (Datastore)

The easiest way to persist data — serializes a Lua table to JSON automatically.

```lua
-- Write
local saveData = { score = 500, level = 3, unlockedSkins = {"red", "blue"} }
playdate.datastore.write(saveData)

-- Named saves (multiple save slots)
playdate.datastore.write(saveData, "slot1")

-- Read (returns table or nil if no save exists)
local data = playdate.datastore.read()
local data = playdate.datastore.read("slot1")

-- Delete
playdate.datastore.delete()
playdate.datastore.delete("slot1")

-- Always save on exit!
function playdate.gameWillTerminate()
    playdate.datastore.write(saveData)
end
function playdate.gameWillSleep()
    playdate.datastore.write(saveData)
end

-- On startup, load with nil check:
local saveData = playdate.datastore.read()
if saveData then
    score = saveData.score or 0
    level = saveData.level or 1
else
    -- first run defaults
    score = 0
    level = 1
end
```

---

## File API (lower-level)

```lua
-- Write
local f = playdate.file.open("mydata.bin", playdate.file.kFileWrite)
f:write("hello world")
f:close()

-- Read
local f = playdate.file.open("mydata.bin", playdate.file.kFileRead)
local content = f:read(1000)   -- read up to N bytes
f:close()

-- Append
local f = playdate.file.open("log.txt", playdate.file.kFileAppend)
f:write("log entry\n")
f:close()

-- Filesystem operations
playdate.file.exists("path/to/file")
playdate.file.isdir("path/to/dir")
playdate.file.mkdir("newdir")
playdate.file.delete("file.txt")
playdate.file.delete("dir", true)   -- recursive delete
playdate.file.rename("old.txt", "new.txt")
local files = playdate.file.listFiles("dir/")  -- returns array of filenames

-- Read entire file as string
local str, err = playdate.file.getSize("file.txt")
```

---

## JSON

Built-in, no import needed.

```lua
-- Encode table → JSON string
local str = json.encode({ name = "username", score = 100 })

-- Decode JSON string → table
local tbl = json.decode(str)

-- Read JSON file directly
local tbl = json.decodeFile("data/levels.json")
```

---

## System Callbacks

```lua
-- Every frame (required!)
function playdate.update() end

-- Game lifecycle
function playdate.gameWillTerminate() end   -- ← save data here
function playdate.gameWillSleep() end        -- ← save data here too
function playdate.deviceWillLock() end       -- screen locking
function playdate.deviceDidUnlock() end      -- screen unlocked
function playdate.gameWillPause() end        -- system menu opened
function playdate.gameWillResume() end       -- system menu closed
function playdate.deviceWillSleep() end      -- going to sleep

-- Simulator-only debug callback
function playdate.keyPressed(key) end        -- keyboard key pressed in Sim
function playdate.keyReleased(key) end
```

---

## System Menu

```lua
local menu = playdate.getSystemMenu()

-- Simple action item
menu:addMenuItem("Restart", function()
    -- handle tap
end)

-- Options item (like a setting)
menu:addOptionsMenuItem("Speed", {"Slow", "Normal", "Fast"}, "Normal",
    function(value)
        gameSpeed = value
    end)

-- Checkmark item
menu:addCheckmarkMenuItem("Music", true,
    function(checked)
        musicEnabled = checked
    end)

-- Remove all custom items
menu:removeAllMenuItems()

-- Custom image on left side when game is paused
playdate.setMenuImage(myImage)
playdate.setMenuImage(myImage, xOffset)  -- optional x offset
```

---

## Display

```lua
-- Refresh rate (default 30fps, max 50fps)
playdate.display.setRefreshRate(30)
playdate.display.getRefreshRate()

-- Size (always 400×240)
local w = playdate.display.getWidth()   -- 400
local h = playdate.display.getHeight()  -- 240

-- Invert display (white bg games)
playdate.display.setInverted(true)

-- Scale (1×, 2×, 4×, 8×) — reduces effective resolution
playdate.display.setScale(1)

-- Mosaic filter
playdate.display.setMosaic(xAmt, yAmt)

-- Flip
playdate.display.setFlippedX(true)
playdate.display.setFlippedY(true)
```

---

## OOP (CoreLibs/object)

```lua
import "CoreLibs/object"

-- Define class
class("MyClass").extends()            -- base class
class("Child").extends(MyClass)       -- subclass

function MyClass:init(x, y)
    MyClass.super.init(self)          -- call super (if extending something)
    self.x = x
    self.y = y
end

function MyClass:getPosition()
    return self.x, self.y
end

-- Instantiate
local obj = MyClass(10, 20)

-- Type check
if obj:isa(MyClass) then ... end
```

---

## Pathfinding

Built-in A* implementation.

```lua
-- No import needed
local graph = playdate.pathfinder.graph.new()

-- Add node (id, x, y, addToAllNodes)
local node = graph:addNewNode(1, 100, 50, false)
-- or create manually:
local node = playdate.pathfinder.node.new(100, 50)
node.id = 1
graph:addNode(node)

-- Connect nodes (one or both directions)
graph:addConnectionToNodeWithID(fromID, toID, weight, addReverse)
-- addReverse=true → bidirectional connection

-- Find path (returns array of nodes, or nil)
local path = graph:findPath(startNode, endNode)
if path then
    for i, node in ipairs(path) do
        print(node.x, node.y)
    end
end

-- Remove node
graph:removeNodeWithID(id)
graph:removeNode(node)
```

---

## Date & Time

```lua
local time = playdate.getTime()
-- time.year, .month, .day, .hour, .minute, .second, .millisecond, .weekday, .yday

local epoch = playdate.getSecondsSinceEpoch()

local ms = playdate.getCurrentTimeMilliseconds()  -- ms since game start
```

---

## Misc

```lua
-- Pause execution (blocks update)
playdate.wait(ms)

-- Accessibility
if playdate.getReduceFlashing() then
    -- skip flashing visuals
end

-- Garbage collection tuning
playdate.setCollectsGarbage(true)          -- default true
playdate.setGCScaling(minFreePercent, maxFreePercent)  -- e.g. 0.4, 0.7

-- Memory stats
local free, total = playdate.getMemoryUsage()

-- Battery
local pct = playdate.getBatteryPercentage()   -- 0–100
local voltage = playdate.getBatteryVoltage()

-- Serial communication
function playdate.serialMessageReceived(msg) end
playdate.debugDraw()   -- draw debug rects in Simulator
```

---

## Localization

```lua
-- Put en.strings / jp.strings in source root:
-- "greeting" = "Hello"

-- Read localized string
local str = playdate.getLocalizedString("greeting")

-- Draw localized text
playdate.graphics.drawLocalizedText("greeting", x, y)

-- Current language
local lang = playdate.getSystemLanguage()
-- returns playdate.graphics.font.kLanguageEnglish or kLanguageJapanese
```
