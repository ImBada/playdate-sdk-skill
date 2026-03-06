# Sound Reference

## Quick Guide

| Use case | API |
|----------|-----|
| Short SFX (loads into RAM) | `sampleplayer` |
| Music / long audio (streams) | `fileplayer` |
| Procedural / synth tones | `synth` |

Recommended format: **ADPCM .wav** for SFX (small, fast). `.mp3` works only with fileplayer.

---

## SamplePlayer

```lua
-- Load sample into memory
local sfx = playdate.sound.sampleplayer.new("sounds/jump")
-- returns nil + error string if file not found

-- Play
sfx:play()           -- play once
sfx:play(0)          -- loop forever
sfx:play(3)          -- play 3 times
sfx:playAt(when)     -- schedule at device time (from getCurrentTime())

-- Control
sfx:stop()
sfx:pause()
sfx:isPlaying()      -- true/false

-- Volume (0.0–1.0)
sfx:setVolume(0.8)
sfx:setVolume(left, right)  -- stereo
local l, r = sfx:getVolume()

-- Pitch
sfx:setRate(1.0)     -- 1.0 = normal, 2.0 = double speed / octave up
local rate = sfx:getRate()

-- Callback when done
sfx.finishCallback = function(player) end

-- Clone (share same sample data, independent playback)
local sfx2 = sfx:copy()
```

---

## FilePlayer

Streams from disk — good for music and long audio (no RAM spike).

```lua
local music = playdate.sound.fileplayer.new("sounds/bgm")
-- or preload:
local music = playdate.sound.fileplayer.new("sounds/bgm", bufferSize)

-- Play
music:play()         -- play once
music:play(0)        -- loop forever
music:play(2)        -- loop 2 times

-- Control
music:stop()
music:pause()
music:isPlaying()
music:setOffset(seconds)   -- seek to position
local pos = music:getOffset()
local len = music:getLength()

-- Volume
music:setVolume(0.8)
music:setVolume(left, right)

-- Loop region
music:setLoopRange(startSec, endSec)
music:setLoopCallback(function(player) end)

-- Finish callback
music.finishCallback = function(player) end

-- Fade
music:setVolume(0, 1000)  -- fade to 0 over 1 second
```

---

## Synth (Procedural Audio)

No import needed; synth is built-in.

```lua
local synth = playdate.sound.synth.new(playdate.sound.kWaveSine)
-- Other waveforms:
-- kWaveSquare, kWaveSawtooth, kWaveTriangle, kWaveNoise
-- kWavePOVosim, kWaveRenoise, kWavePhasemod

synth:setVolume(0.5)

-- Play note by name or MIDI number
synth:playNote("C4")                   -- play indefinitely
synth:playNote("C4", 0.5)             -- at volume 0.5
synth:playNote("C4", 0.5, 0.25)       -- for 0.25 seconds
synth:playNote(60, 0.5, 0.25)         -- MIDI 60 = C4

-- Play by frequency
synth:playFrequency(440, 0.5, 0.25)   -- 440Hz = A4

synth:noteOff()   -- release note (lets envelope decay)
synth:stop()      -- hard stop

-- ADSR envelope
synth:setAttackTime(0.01)   -- seconds
synth:setDecayTime(0.1)
synth:setSustainLevel(0.7)  -- 0.0–1.0
synth:setReleaseTime(0.3)

-- FM synthesis
synth:setFrequencyModulator(lfo)
synth:setAmplitudeModulator(lfo)
```

---

## LFO (Low Frequency Oscillator)

```lua
local lfo = playdate.sound.lfo.new(playdate.sound.kLFOSine)
lfo:setRate(4)       -- cycles per second
lfo:setDepth(0.5)    -- modulation amount
lfo:setCenter(0.0)   -- center value

-- Attach to synth parameter
synth:setFrequencyModulator(lfo)
```

---

## Channels & Mixing

```lua
-- Default channel
local ch = playdate.sound.getDefaultChannel()

-- Custom channel
local ch = playdate.sound.channel.new()
ch:setVolume(0.8)
ch:setPan(0.0)      -- -1.0 left, 0 center, 1.0 right
source:addToChannel(ch)
source:removeFromChannel(ch)

-- Effects
local reverb = playdate.sound.effect.ringmodulator.new()
ch:addEffect(reverb)
```

---

## Microphone

```lua
playdate.sound.micinput.startListening()
local level = playdate.sound.micinput.getLevel()  -- 0.0–1.0
playdate.sound.micinput.stopListening()
```

---

## Audio Time

```lua
local t = playdate.sound.getCurrentTime()  -- device audio clock
-- Use with sampleplayer:playAt() for precise scheduling
```
