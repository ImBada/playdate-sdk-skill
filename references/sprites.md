# Sprites & Collision Reference

Sprites are the primary rendering and game-object system. They use dirty-rect drawing (only redraws changed areas) and provide built-in collision detection.

## Required imports
```lua
import "CoreLibs/graphics"
import "CoreLibs/sprites"
import "CoreLibs/object"   -- for OOP class-based sprites
```

## Basic Usage
```lua
local gfx = playdate.graphics

-- From image
local img = gfx.image.new("images/player")
local s = gfx.sprite.new(img)
s:moveTo(200, 120)   -- center position
s:add()              -- add to scene (won't draw until added)

-- MUST call every frame to draw all sprites:
gfx.sprite.update()
```

## Creating Sprites
```lua
local s = gfx.sprite.new()           -- blank
local s = gfx.sprite.new(image)      -- with image
local s = gfx.sprite.new(image, rect) -- with image cropped to rect

s:setImage(img)
s:setImage(img, gfx.kImageFlippedX)  -- flipped

-- Custom draw (size MUST be set before draw is called)
s:setSize(w, h)
function s:draw(x, y, width, height)
    -- x/y/w/h = dirty rect being updated (NOT sprite dims)
    -- coordinates are relative to sprite's top-left corner
    gfx.fillRect(0, 0, width, height)
end
```

## Movement & Position
```lua
s:moveTo(x, y)            -- set center position
s:moveBy(dx, dy)          -- relative move
local x, y = s:getPosition()
local bounds = s:getBounds()           -- playdate.geometry.rect
local x, y, w, h = s:getBoundsRect()
s:setCenter(cx, cy)       -- pivot point (0–1, default 0.5/0.5)
```

## Lifecycle & Visibility
```lua
s:add()                    -- add to global sprite list
s:remove()                 -- remove from list (doesn't delete object)
s:setVisible(true)
s:isVisible()
s:setZIndex(n)             -- higher = drawn on top
s:getZIndex()
s:setTag(n)                -- arbitrary int tag for identification
s:getTag()
s:setUpdatesEnabled(true)  -- enable/disable :update() callback
s:markDirty()              -- force redraw next frame
```

## Per-Sprite Callbacks
```lua
function s:update()
    -- called each frame before draw
    -- only works if sprite has been :add()ed
end

function s:draw(x, y, width, height)
    -- custom drawing; called only when sprite is dirty
end
```

## Global Sprite Operations
```lua
gfx.sprite.update()                    -- draw all sprites (call each frame!)
gfx.sprite.setAlwaysRedraw(true)       -- disable dirty-rect optimization
gfx.sprite.getAllSprites()             -- returns array of all sprites
gfx.sprite.removeAll()                 -- remove every sprite

-- Background drawing (drawn before sprites, no dirty-rect)
gfx.sprite.setBackgroundDrawingCallback(function(x, y, w, h)
    -- draw background tiles, etc.
end)
```

## Collision Detection
```lua
-- Step 1: set a collide rect (relative to sprite's top-left corner)
s:setCollideRect(0, 0, w, h)
-- Note: getCollideBounds() returns x,y,w,h relative to sprite origin

-- Step 2: move with collision (instead of moveTo/moveBy)
local actualX, actualY, collisions, len =
    s:moveWithCollisions(targetX, targetY)

-- Step 3: handle each collision
for i = 1, len do
    local col = collisions[i]
    local other   = col.other          -- the other sprite
    local normal  = col.normal         -- {x, y} push-out direction
    local touch   = col.touch          -- point where they touched
    local ti      = col.ti             -- time of impact 0–1
    local move    = col.move           -- {x, y} attempted movement
    local response = col.type          -- kCollisionTypeSlide, etc.

    -- normal.x ~= 0 → hit left or right wall
    -- normal.y ~= 0 → hit floor or ceiling
end

-- Collision response (override per-sprite):
function s:collisionResponse(other)
    return gfx.sprite.kCollisionTypeSlide    -- slide along surface
    -- kCollisionTypeBounce                  -- reflect velocity
    -- kCollisionTypeOverlap                 -- pass through, notify
    -- kCollisionTypeFreeze                  -- stop on contact
end

-- Overlap check without movement
local overlaps = s:overlappingSprites()  -- returns array of sprites
local touching, len = s:allOverlappingSprites()

-- Query without moving
local sprites = gfx.sprite.querySpritesInRect(x, y, w, h)
local sprites, len = gfx.sprite.querySpritesAtPoint(x, y)
local sprites, len = gfx.sprite.querySpritesAlongLine(x1, y1, x2, y2)
local infos,  len  = gfx.sprite.querySpriteInfoAlongLine(lineSegment)
-- infos[i] has: .sprite, .entryPoint, .exitPoint, .ti1, .ti2
```

## OOP Pattern (recommended for complex games)
```lua
import "CoreLibs/object"
import "CoreLibs/sprites"

class("Player").extends(playdate.graphics.sprite)

function Player:init(x, y)
    Player.super.init(self)          -- ALWAYS call this first!

    local img = playdate.graphics.image.new("images/player")
    self:setImage(img)
    self:moveTo(x, y)
    self:setCollideRect(0, 0, img:getSize())
    self:setTag(1)                   -- optional: tag for collision filtering
    self:add()
end

function Player:update()
    local dx, dy = 0, 0
    if playdate.buttonIsPressed(playdate.kButtonRight) then dx = 3 end
    if playdate.buttonIsPressed(playdate.kButtonLeft)  then dx = -3 end
    if playdate.buttonIsPressed(playdate.kButtonDown)  then dy = 3 end
    if playdate.buttonIsPressed(playdate.kButtonUp)    then dy = -3 end

    if dx ~= 0 or dy ~= 0 then
        local _, _, cols, len = self:moveWithCollisions(self.x + dx, self.y + dy)
        for i = 1, len do
            -- handle cols[i]
        end
    end
end

function Player:collisionResponse(other)
    return playdate.graphics.sprite.kCollisionTypeSlide
end

-- Type checking:
if sprite:isa(Player) then ... end

-- Usage:
local player = Player(200, 120)
```

## Animated Sprites
```lua
import "CoreLibs/animation"

-- Image table file: "walk-table-32-32.png" (grid of 32×32 frames)
local imgTable = playdate.graphics.imagetable.new("images/walk")

-- Create animation loop (ms per frame, imageTable, shouldLoop)
local walkAnim = playdate.graphics.animation.loop.new(100, imgTable, true)
local idleAnim = playdate.graphics.animation.loop.new(200, idleTable, true)

function Player:update()
    local isMoving = (dx ~= 0 or dy ~= 0)
    local currentAnim = isMoving and walkAnim or idleAnim
    self:setImage(currentAnim:image())
end

-- Animation loop properties:
anim.frame          -- current frame number
anim.isValid        -- false when non-looping anim has finished
anim:image()        -- returns current frame image
```

## Tilemap Collision
```lua
-- Use tilemap as static collision geometry
local tilemapSprite = gfx.sprite.new()
tilemapSprite:setTilemap(tilemap)
tilemapSprite:setCollideRect(0, 0, tilemap:getSize())
tilemapSprite:moveTo(0, 0)
tilemapSprite:add()

-- Player then uses moveWithCollisions normally and will hit tile edges
```
