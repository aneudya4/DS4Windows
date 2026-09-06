# Injected stick wobble

## What "injected input" means

Everything in the `Axis Config` tab (deadzone, curve, sensitivity, stick
calibration) reshapes movement the player actually made. When the thumb rests,
the virtual pad reports the stick at rest and the game sees it inside its own
deadzone. Rotational aim assist in most shooters is only engaged while the
stick is outside that deadzone, so it switches off whenever the player stops
moving. That is normal behaviour and it is what the calibrator works with.

Injected wobble is different. DS4Windows adds a small sine wave to the stick's
X axis *after* all of the above processing, right before the value is handed
to the virtual pad. The game never sees the stick sitting inside its deadzone,
believes the player is constantly strafing by a tiny amount, and keeps
rotational assist engaged while the thumb is still. The player did not make
that movement; the software did. This is the same trick sold by third party
"aim assist" adapters.

Pipeline:

```
DualSense  ->  DS4Windows                          ->  virtual pad  ->  game
 (thumb)       deadzone . curve . sensitivity . +wobble   (what Windows sees)
```

## Settings

Per profile, per stick, under `Axis Config` -> `Left Stick` / `Right Stick`
-> **Injected Wobble**:

| Setting   | Default | Range        | Meaning                                              |
|-----------|---------|--------------|------------------------------------------------------|
| Enabled   | off     |              | Adds the wobble to that stick's output               |
| Amplitude | 8 %     | 0 - 100 %    | Peak offset as a percentage of full axis travel      |
| Rate      | 20 Hz   | 0.1 - 100 Hz | Full sine cycles per second                          |

Amplitude should sit a little above the game's own deadzone (8 % for a 5 %
deadzone in the illustration this was built from). Profile XML elements are
`LSWobble`, `LSWobbleAmplitude`, `LSWobbleRate` and the `RS` equivalents.

## Implementation notes

- `StickWobbleInfo` (`ProfilePropGroups.cs`) holds the three values; the
  backing store keeps one per profile slot and `ResetProfile` restores defaults.
- `Mapping.SetCurveAndDeadzone` applies the wobble as its last step so it is
  never eaten by the deadzone or bent by the output curve. The offset is
  `sin(phase) * amplitude/100 * 127` axis units on X; Y is untouched.
- The phase is accumulated from `Stopwatch` time per stick and profile slot, so
  changing the rate while playing does not jump the waveform. Gaps of a second
  or more (reconnect, profile reload) do not spin the phase forward.
- Because it runs inside `SetCurveAndDeadzone`, the profile editor's controller
  readings preview shows the wobbled output as well, which is what the game
  would receive.
