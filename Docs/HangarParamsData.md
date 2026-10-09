# WOTB/TB — Hangar Params Data

Reference for the four hangar config files used by World of Tanks Blitz/Tanks Blitz to describe tank stats, their units, their layout, and their balance bounds.

---

## Files Overview

| File | Purpose | Contains |
|------|---------|----------|
| `hangarParamsConfig.yaml` | Per-parameter display settings | Units, decimal digits, biggerIsBetter flag |
| `hangarParamsStructureConfig.yaml` | Layout of the hangar UI | Sections, subsections, tables, order of params |
| `hangarBaseParamsBalance.yaml` | Weights and global min/max | Used to compute the 0–100% progress bars |
| `hangarBaseParamsPrecomputedData_generated.yaml` | Precomputed per-tank values | One string per tank with 7 base stats + per-level min/max |

---

## `hangarParamsConfig.yaml` — Parameter Definitions

Defines how each stat is displayed: units, precision, and whether higher is better.

| Key | Units | Digits | biggerIsBetter |
|-----|-------|--------|----------------|
| `maxHealth` | — | — | true |
| `hullAverageThickness` | — | 1 | true |
| `turretArmorFront / Side / Rear` | — | — | true |
| `primaryArmorFront / Side / Rear` | — | — | true |
| `engineFireStartingChance` | PERCENT | — | false |
| `circularVisionRadius` | — | 1 | true |
| `stillInvisibility` | PERCENT | — | true |
| `movingInvisibility` | PERCENT | — | true |
| `shotInvisibility` | PERCENT | — | true |
| `gunShotsPerMinute` | — | 1 | true |
| `gunCommonDamagePerMin` | — | — | true |
| `gunShellReloadTime` | — | 2 | false |
| `gunClipReloadTime` | — | 2 | false |
| `gunClipShellReloadTime` | — | 2 | false |
| `gunClipBurstReloadTime` | — | 2 | false |
| `gunDupletReloadTime` | — | 2 | false |
| `timeBetweenPumpShots` | — | 2 | false |
| `reloadingShellTime` | — | 2 | false |
| `gunAimingTime` | — | 1 | false |
| `gunShotDispersionRadius` | — | 3 | false |
| `gunPitchUpLimit` | DEGREE | — | true |
| `gunPitchDownLimit` | DEGREE | — | false |
| `turretYawLeftLimit / RightLimit` | DEGREE | — | true |
| `baseShellAvgDamage` | — | — | true |
| `premShellAvgDamage` | — | — | true |
| `altShellAvgDamage` | — | — | true |
| `baseShellAvgPiercingPower` | — | — | true |
| `premShellAvgPiercingPower` | — | — | true |
| `altShellAvgPiercingPower` | — | — | true |
| `baseShellStartSpeed` | — | — | true |
| `premShellStartSpeed` | — | — | true |
| `altShellStartSpeed` | — | — | true |
| `baseShellMaxSpeed` | — | — | true |
| `baseShellAccelerationTime` | — | 2 | false |
| `hasTurret` | — | — | — |
| `hasMissileShell` | — | — | — |
| `pumpGunMode` | — | — | — |
| `enginePower` | HORSE_POWER | 1 | true |
| `weight` | TON | 2 | false |
| `chassiMaxLoad` | TON | — | true |
| `chassiRotationSpeed` | DEGREE_PER_SECOND | 2 | true |
| `turretRotationSpeed` | DEGREE_PER_SECOND | 2 | true |
| `maxFrontSpeed` | KILOMETER_PER_HOUR | — | true |
| `maxBackSpeed` | KILOMETER_PER_HOUR | — | true |
| `avgSpeed` | KILOMETER_PER_HOUR | — | true |
| `thrustToWeightRatio` | HORSE_POWER_PER_TON | 1 | true |
| `terrainPermeabilityFirm` | PERCENT | — | true |
| `terrainPermeabilityMedium` | PERCENT | — | true |
| `terrainPermeabilitySoft` | PERCENT | — | true |
| `healthBurnPerSec` | — | 1 | false |

---

## `hangarParamsStructureConfig.yaml` — Layout

Defines where each param appears in the hangar UI.

### Main sections

- `text`
- `protectability`
- `gun`
- `mobility`

### Subsections

| Section | Content |
|---------|---------|
| `base` | `maxHealth`, `damage`, `firingRate`, `penetration`, `armor`, `speed`, `rotation` |
| `rawbase` | Same as `base`, but uses raw values |
| `protectability` | `maxHealth`, armor table (turret/hull × front/side/rear), `engineFireStartingChance`, `circularVisionRadius`, invisibility |
| `gun` | `gunCommonDamagePerMin`, reload times, clip params, shell params, `gunAimingTime`, dispersion, pitch/yaw limits |
| `mobility` | `maxFrontSpeed`, `maxBackSpeed`, `avgSpeed`, `thrustToWeightRatio`, `weight`, `turretRotationSpeed`, `chassiRotationSpeed`, terrain, `enginePower` |
| `vehicleChassis` | `weight`, `chassiRotationSpeed`, `gunAimingTime`, terrain |
| `vehicleEngine` | `weight`, `enginePower`, `engineFireStartingChance`, `maxFrontSpeed`, `maxBackSpeed`, `avgSpeed`, `thrustToWeightRatio`, `chassiRotationSpeed` |
| `vehicleTurret` | `weight`, `maxHealth`, turret armor, `turretRotationSpeed`, yaw limits, `circularVisionRadius`, pitch limits |
| `vehicleGun` | `weight`, `gunAimingTime`, dispersion, `damage/min`, reloads, clip params, shells, pitch limits |
| `crew` | `gunShotsPerMinute`, `circularVisionRadius`, `gunAimingTime`, `gunShotDispersionRadius`, `turretRotationSpeed`, `chassiRotationSpeed`, terrain |

---

## `hangarBaseParamsBalance.yaml` — Weights & Bounds

Used to compute the 0–100% progress bars in the hangar.

### Weights

| Parameter | Weight |
|-----------|--------|
| `averageThicknessWeight` | 0.6 |
| `chassisRotationSpeedWeight` | 0.35 |
| `enginePowerPerTonWeight` | 0.5 |
| `maxHealthWeight` | 0.4 |
| `maxSpeedWeight` | 0.15 |
| `piercingPowerWeight` | 0.6 |
| `shotAccuracyWeight` | 0.4 |

### Global min/max (with reference tank)

| Parameter | Min | Tank | Max | Tank |
|-----------|-----|------|-----|------|
| `averageThickness` | 5.756888 | GB39_Universal_CarrierQF2 | 324.970734 | Oth23_Werewolf |
| `chassisRotationSpeed` | 0.209440 rad/s | G110_Mauschen | 1.047198 rad/s | Ke_Ho |
| `enginePowerPerTon` | 3687.346892 | T95 | 26464.767616 | AMX_dracula |
| `maxHealth` | 270 | Ha_Go | 3300 | Maus |
| `maxSpeed` | 3.333360 | GB44_Archer_Custom | 22.222400 | G103_RU_251 |
| `piercingPower` | 20 | T82 | 310 | G181_StuG_Maus |
| `shotDamage` | 10 | PzII | 930 | GB48_FV215b_183 |
| `shotDispersionAngle` | 0.002800 | G121_Grille_15_L63 | 0.005700 | KV2 |
| `shotsPerMinute` | 2.222222 | R171_IS_3_II | 73.469388 | PzII_J |
| `firePower` | 294.117647 | GB69_Cruiser_Mk_II | 4675.324675 | Oth001_Protheus |
| `mobility` | 1.764706 | T95 | 91.760616 | Bat_Chatillon25t |
| `protectability` | 1.056106 | GB39_Universal_CarrierQF2 | 90.084988 | Maus |
| `shotEfficiency` | 2.068966 | KV2 | 97.931034 | G121_Grille_15_L63 |

---

## `hangarBaseParamsPrecomputedData_generated.yaml` — Per-Tank Values

Each tank has a single string with 7 base values, in this order:
