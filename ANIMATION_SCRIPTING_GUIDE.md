# 3D Army Animation Scripting Guide (`animCommand` DSL)

**Document**: `d:\git\tour_data\ANIMATION_SCRIPTING_GUIDE.md`  
**Target Systems**: Narrative Story Engine (`story_uhud.json`, `stories.json`), Audio-Sync Provider, Native 3D Map Engine  
**Last Updated**: 2026-08-11

---

## 1. Overview & Architecture

The **Animation Scripting Engine** allows historical battle scenes and army movements (e.g., Battle of Uhud) to be scripted directly inside narrative JSON files. 

As the audio narrator reads each sentence, the `audio_sync_provider.dart` executes the sentence's `animCommand` string, resolving semantic location IDs into GPS coordinates and animating 3D unit formations on the terrain map synchronously with the voiceover.

```
+---------------------+     +-----------------------+     +------------------------+
|  story_uhud.json    | --> |  audio_sync_provider  | --> |  3D Map Engine         |
|  (animCommand DSL)  |     |  (resolves tokens)    |     |  (renders 3D units)    |
+---------------------+     +-----------------------+     +------------------------+
```

---

## 2. Defining Army Units (`tacticalUnits`)

At the root of a story JSON file (e.g. `story_uhud.json`), declare the units involved in the scene using the `tacticalUnits` array:

```json
"tacticalUnits": [
  {
    "id": "meccan",
    "count": 30,
    "color": "#FF0000",
    "size": 5.0,
    "spacing": 8.0,
    "discipline": 0.6
  },
  {
    "id": "muslim",
    "count": 10,
    "color": "#43A047",
    "size": 5.0,
    "spacing": 8.0,
    "discipline": 0.8
  },
  {
    "id": "courier",
    "count": 1,
    "color": "#FDD835",
    "size": 5.0,
    "spacing": 0.0,
    "banner": "image/message.png",
    "discipline": 1.0
  }
]
```

### `tacticalUnits` Parameters

| Field | Type | Description |
|:---|:---|:---|
| `id` | string | Unique identifier for the unit group (referenced in commands). |
| `count` | int | Number of 3D unit entities/soldiers to spawn in formation. |
| `color` | string | Hex color code for unit uniforms/markers (e.g. `#FF0000` red, `#43A047` green). |
| `size` | float | 3D visual render scale of each soldier model. |
| `spacing` | float | Grid spacing (in meters) between individual soldiers in formation. |
| `banner` | string | *(Optional)* Icon/banner image path displayed above the unit leader. |
| `discipline` | float | Movement cohesion factor (`0.0` chaotic swarm to `1.0` rigid military march). |

---

## 3. Sentence-Level Scripting (`animCommand` DSL)

Inside each narrative sentence object, attach an `animCommand` string to orchestrate movement:

```json
"sentences": [
  {
    "text": {
      "en": "The Meccans prepared an army of 3,000 men to attack Madinah."
    },
    "animCommand": "meccan[STAY,name[darul_nadwah],3';AWAY,name[darul_nadwah],name[masjid_jinn],40',delay:3']"
  }
]
```

---

## 4. Location Resolvers

Commands locate waypoints on the map using three resolver syntaxes:

### A. Semantic Location Lookup (`name[location_id]`)
References a location ID from `locations.json`. Automatically expanded by Dart to `id|latitude,longitude`.
- **Syntax**: `name[masjid_nabawi]`
- **Resolved**: `masjid_nabawi|24.4672,39.6112`

### B. Raw Coordinates (`latlong[lat, lng]`)
Explicit latitude and longitude for custom map spots.
- **Syntax**: `latlong[24.5028, 39.6118]`
- **Resolved**: `custom|24.5028,39.6118`

### C. Relative Offsets (`offset[base_location, dx, dy]`)
Calculates a coordinate offset relative to a base location.
- **Syntax**: `offset[name[masjid_shaikhain], 0.002, -0.001]`
- **Resolved**: `offset_pos|24.49139,39.60779`

---

## 5. Movement Action Directives

Inside `unit_id[...]`, chain one or more movement actions separated by `;`:

| Action Directive | Syntax | Description |
|:---|:---|:---|
| `STAY` | `STAY,location,duration'` | Holds unit formation at `location` for `duration'` seconds. |
| `AWAY` | `AWAY,start_loc,end_loc,duration'` | Marches unit away from `start_loc` toward `end_loc` over `duration'`. |
| `LERP` | `LERP,start_loc,end_loc,duration'` | Smooth linear position interpolation from `start_loc` to `end_loc`. |
| `MOVE` | `MOVE,start_loc,end_loc,duration'` | Tactical unit march along the terrain path. |
| `delay:` | `delay:seconds'` | Delays execution of the action by the specified seconds. |

*Note: Duration values must end with a single quote `'` (e.g., `40'` = 40 seconds, `3'` = 3 seconds).*

---

## 6. Pronunciation & Display Text Markup

For proper Audio TTS speech generation vs on-screen display text, use bracketed markup:

- **Syntax**: `[Display Text / TTS Pronunciation]`
- **Example**: `"His wife, [Hind/Heend], was upset..."`
  - **Displayed on screen**: `"His wife, Hind, was upset..."`
  - **Spoken by TTS Engine**: `"His wife, Heend, was upset..."`

---

## 7. Complete Real-World Scripting Examples

### Example 1: Army Hold & March Advance
```json
{
  "text": {
    "en": "The Meccans prepared an army of 3,000 men to attack Madinah."
  },
  "animCommand": "meccan[STAY,name[darul_nadwah],3';AWAY,name[darul_nadwah],name[masjid_jinn],40',delay:3']"
}
```

### Example 2: Messenger Courier Express Travel
```json
{
  "text": {
    "en": "Rasulullah's uncle Abbas sent a message warning of the Meccan army."
  },
  "animCommand": "(courier[LERP,name[darul_nadwah],name[masjid_quba],10'])"
}
```

### Example 3: Dual Army Convergence at Mount Uhud
```json
{
  "text": {
    "en": "The Muslim army stationed at Archers' Hill while the Meccan cavalry took position on the plain."
  },
  "animCommand": "muslim[MOVE,name[masjid_shaikhain],name[jabal_rumat],12']; meccan[MOVE,name[masjid_jinn],name[battlefield_uhud],15']"
}
```
