# Graphics API Reference

## Setup
```lua
local gfx = playdate.graphics  -- standard alias
```

## Colors (monochrome only!)
```lua
gfx.setColor(gfx.kColorBlack)
gfx.setColor(gfx.kColorWhite)
gfx.setColor(gfx.kColorClear)
gfx.setColor(gfx.kColorXOR)

gfx.setBackgroundColor(gfx.kColorWhite)
gfx.clear()  -- clear to background color
```

## Shapes
```lua
gfx.fillRect(x, y, w, h)
gfx.drawRect(x, y, w, h)
gfx.drawRoundRect(x, y, w, h, radius)
gfx.fillRoundRect(x, y, w, h, radius)
gfx.fillCircleAtPoint(x, y, radius)
gfx.drawCircleAtPoint(x, y, radius)
gfx.fillCircleInRect(x, y, w, h)
gfx.drawCircleInRect(x, y, w, h)
gfx.fillEllipseInRect(x, y, w, h)
gfx.drawEllipseInRect(x, y, w, h)
gfx.drawLine(x1, y1, x2, y2)
gfx.fillPolygon(p1x, p1y, p2x, p2y, ...)
gfx.drawPolygon(p1x, p1y, p2x, p2y, ...)
gfx.fillTriangle(x1, y1, x2, y2, x3, y3)
gfx.drawArc(x, y, radius, startAngle, endAngle)
gfx.fillArc(x, y, radius, startAngle, endAngle)
```

## Images
```lua
-- Load (no extension needed; pdc compiles .png → .pdi)
local img = gfx.image.new("images/player")
local img = gfx.image.new(width, height)          -- blank image
local img = gfx.image.new(width, height, gfx.kColorWhite)

-- Draw
img:draw(x, y)
img:draw(x, y, gfx.kImageFlippedX)               -- flip H
img:draw(x, y, gfx.kImageFlippedY)               -- flip V
img:draw(x, y, gfx.kImageFlippedXY)              -- flip both
img:drawCentered(x, y)
img:drawScaled(x, y, scale)
img:drawScaled(x, y, xScale, yScale)
img:drawRotated(x, y, angle)
img:drawRotated(x, y, angle, scale)
img:drawWithTransform(transform, x, y)
img:drawTiled(x, y, w, h)

local w, h = img:getSize()

-- Transformations (return new image, don't mutate)
local rotated  = img:rotatedImage(angle)
local scaled   = img:scaledImage(scale)
local scaled   = img:scaledImage(xScale, yScale)
local inverted = img:invertedImage()
local blurred  = img:blurredImage(radius, numPasses, ditherType)
local faded    = img:fadedImage(alpha, ditherType)

-- Mask
img:setMaskImage(maskImg)
local mask = img:getMaskImage()
img:clearMask()
img:addMask()          -- add blank mask (fully opaque)

-- Drawing into an image (offscreen rendering)
local canvas = gfx.image.new(400, 240)
gfx.pushContext(canvas)
    gfx.fillRect(0, 0, 100, 100)
gfx.popContext()
```

## Image Tables (sprite sheets / animation frames)
```lua
-- File naming: "walk-table-32-32.png" = grid of 32×32 px frames
local imgTable = gfx.imagetable.new("images/walk")
local img = imgTable:getImage(n)       -- 1-indexed
local img = imgTable[n]                -- shorthand
local len = imgTable:getLength()       -- or #imgTable
imgTable:drawImage(n, x, y)
```

## Drawing Mode
```lua
gfx.setImageDrawMode(gfx.kDrawModeCopy)        -- normal
gfx.setImageDrawMode(gfx.kDrawModeWhiteTransparent)
gfx.setImageDrawMode(gfx.kDrawModeBlackTransparent)
gfx.setImageDrawMode(gfx.kDrawModeFillWhite)
gfx.setImageDrawMode(gfx.kDrawModeFillBlack)
gfx.setImageDrawMode(gfx.kDrawModeXOR)
gfx.setImageDrawMode(gfx.kDrawModeNXOR)
gfx.setImageDrawMode(gfx.kDrawModeInverted)
```

## Patterns
```lua
-- 8-byte array defines 8×8 px repeating pattern
gfx.setPattern({0xAA, 0x55, 0xAA, 0x55, 0xAA, 0x55, 0xAA, 0x55})
gfx.fillRect(x, y, w, h)   -- fills with pattern
gfx.setDitherPattern(alpha, gfx.image.kDitherTypeBayer8x8)
```

## Drawing Modifiers
```lua
-- Clipping
gfx.setClipRect(x, y, w, h)
gfx.clearClipRect()

-- Stencil
gfx.setStencilImage(maskImage)
gfx.setStencilImage(maskImage, tile)  -- tile=true to repeat
gfx.clearStencil()

-- Draw offset (for scrolling — offsets ALL drawing)
gfx.setDrawOffset(ox, oy)

-- Line width
gfx.setLineWidth(2)
gfx.setLineCap(gfx.kLineCapRound)

-- Context stack
gfx.pushContext(image)   -- draw into image
gfx.popContext()         -- back to screen
```

## Text
```lua
-- Basic
gfx.drawText("Hello!", x, y)
gfx.drawTextAligned("Centered", x, y, kTextAlignment.center)
gfx.drawTextAligned("Right", x, y, kTextAlignment.right)

-- Styled (markup: *bold*, _italic_, etc.)
gfx.drawText("*bold* and _italic_", x, y)

-- Fonts
local font = gfx.font.new("fonts/Roobert-11-Mono")
gfx.setFont(font)
local w, h = gfx.getTextSize("hello")
local w, h = font:getTextWidth("hello"), font:getHeight()

-- Font variants
gfx.setFont(font, gfx.font.kVariantNormal)
gfx.setFont(font, gfx.font.kVariantBold)
gfx.setFont(font, gfx.font.kVariantItalic)
```

## Tilemap
```lua
local imgTable = gfx.imagetable.new("images/tiles")
local tilemap = gfx.tilemap.new()
tilemap:setImageTable(imgTable)
tilemap:setSize(mapW, mapH)       -- in tiles
tilemap:setTileAtPosition(col, row, tileIndex)  -- 1-indexed, 0=empty
local idx = tilemap:getTileAtPosition(col, row)
tilemap:draw(x, y)
local w, h = tilemap:getSize()    -- in tiles
local pw, ph = tilemap:getTileSize()

-- Collision from tilemap
local collisionRects = tilemap:getCollisionRects(tileRect)
```

## Misc
```lua
-- VCR glitch effect
local glitched = gfx.image.vcrPauseFilterImage(img)

-- Perlin noise
local val = gfx.perlin(x, y, z, repeat, octaves, persistence)
-- val is 0.0–1.0

-- QR Code
import "CoreLibs/graphics"
gfx.generateQRCode(text, size, callback)  -- async, callback(img)

-- FPS overlay (debug)
playdate.drawFPS(x, y)

-- Sine wave
gfx.drawSineWave(x1, y1, x2, y2, startAmp, endAmp, period, phaseShift)
```
