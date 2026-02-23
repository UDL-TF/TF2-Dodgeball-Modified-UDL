# TF2 Dodgeball Custom Differences

This file documents behavior/config changes added on top of the base plugin so custom logic is easy to audit.

## 1) Per-class bounce force config is now wired

### What changed
- `bounce force scale` and `bounce force angle` are now parsed per rocket class from `general.cfg`.
- Bounce logic now uses class values, not only global ConVars.

### Behavior
- If class keys exist, those values are used.
- If class keys are missing, fallback defaults come from:
  - `tf_dodgeball_bounce_force_scale`
  - `tf_dodgeball_bounce_force_angle`

### Code
- Parse keys:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_config.inc:430`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_config.inc:431`
- Use values in bounce flow:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:361`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:362`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:379`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:380`

## 2) Added crawl bounce mode

### What changed
- New rocket flag:
  - `RocketFlag_CrawlBounce = 1 << 25`
- New class settings:
  - `crawl bounce` (on/off)
  - `crawl bounce scale`
  - `crawl bounce max up`

### Behavior
- Reflection vector is still computed with standard bounce reflection.
- Normal mode:
  - scales all velocity components with normal bounce logic.
- Crawl mode:
  - scales X/Y by bounce scale
  - scales Z by `bounce scale * crawl bounce scale`
  - clamps positive Z to `crawl bounce max up` when `crawl bounce max up > 0`

### Code
- New flag:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/tfdb.inc:63`
- New class arrays:
  - `TF2Dodgeball/addons/sourcemod/scripting/dodgeball.sp:131`
  - `TF2Dodgeball/addons/sourcemod/scripting/dodgeball.sp:132`
- Parse keys + enable flag:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_config.inc:404`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_config.inc:432`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_config.inc:433`
- Bounce branch:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:354`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:371`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:388`

### Config examples currently present
- `TF2Dodgeball/addons/sourcemod/configs/dodgeball/general.cfg:175`
- `TF2Dodgeball/addons/sourcemod/configs/dodgeball/general.cfg:176`
- `TF2Dodgeball/addons/sourcemod/configs/dodgeball/general.cfg:177`
- `TF2Dodgeball/addons/sourcemod/configs/dodgeball/general.cfg:241`
- `TF2Dodgeball/addons/sourcemod/configs/dodgeball/general.cfg:242`
- `TF2Dodgeball/addons/sourcemod/configs/dodgeball/general.cfg:243`

## Notes
- Keep this file updated whenever gameplay behavior diverges from upstream/default plugin behavior.

## 3) Map-change stability fix for `object_deflected` event hook

### Problem
- Intermittent map-load errors:
  - `Game event "object_deflected" has no active hook`
  - `Invalid hook callback specified for game event "object_deflected"`
- If unhook throws during map end, `EventsHooked` could remain true and block re-hooking on next map, which prevents normal gamemode flow (including rocket spawning).

### What changed
- Added explicit hook-state tracking for `object_deflected`.
- Only unhook `object_deflected` when it was actually hooked.
- Updated callback signature to a standard event-hook form returning `Action`.

### Code
- Hook-state tracking and guarded unhook:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_core.inc:13`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_core.inc:56`
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_core.inc:136`
- Callback signature:
  - `TF2Dodgeball/addons/sourcemod/scripting/include/dodgeball_events.inc:203`
