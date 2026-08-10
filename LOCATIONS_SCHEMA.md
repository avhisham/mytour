# locations.json — Schema Documentation

**File**: `d:\git\tour_data\locations.json`  
**Synced to**: `c:\flutterprojects\mytrip\assets\data\locations.json` (via nLite update)  
**Last standardized**: 2026-08-10

---

## Purpose

Universal source of truth for all custom locations used by the MyTrip app:
- **Lat/lng coordinates** — every location pin on the map
- **Multilingual names & descriptions** — 4 languages (EN / MS / ML / TA)
- **Display images** — `imageUrls[]`
- **AI prompt images** — `promptImages[]` (with embedded DINOv2+ORB vectors)
- **Historical sentences** — `sentences[]` for narration/subtitle use

---

## ID Convention

IDs must be:
- **Human-readable** and **lowercase snake_case**
- **Unique** across the whole file
- **Descriptive** — prefer `masjid_nabawi` over `l6`

### Naming Patterns by Type

| Pattern | Example |
|:---|:---|
| Mosque | `masjid_<name>` |
| Miqat | `miqat_<name>` |
| Mountain/Hill | `jabal_<name>` |
| Gate | `gate_<name>` |
| Jamarah | `jamarah_<ula\|wusta\|aqabah>` |
| Battlefield/Cemetery | `battlefield_<name>`, `shuhada_<name>` |
| Historical house/site | `<person>_house`, `<person>_birth_place` |
| City | bare name: `makkah`, `madinah` |

---

## Full Field Reference

```json
{
  "id": "rasul_birth_place",

  "name": {
    "en": "Birthplace of Rasulullah ﷺ (Makkah Library)",
    "ms": "Tempat Lahir Rasulullah ﷺ (Perpustakaan Makkah)",
    "ml": "മുഹമ്മദ് നബി ﷺ ജനിച്ച സ്ഥലം (മക്ക ലൈബ്രറി)",
    "ta": "நபிகள் நாயகம் ﷺ பிறந்த இடம் (மக்கா நூலகம்)"
  },

  "description": {
    "en": "The site where the Prophet Muhammad ﷺ was born...",
    "ms": "...",
    "ml": "...",
    "ta": "..."
  },

  "latitude": 21.424979,
  "longitude": 39.829946,

  "altitude": 350.0,

  "type": "Historical",

  "chronologyDate": "0570-04-12",

  "osmId": "node/6844620719",

  "imageUrls": [
    {
      "url": "image/tempat/birth.jpg",
      "caption": {
        "en": "The site of the Prophet's ﷺ birth.",
        "ms": "...",
        "ml": "...",
        "ta": ""
      }
    }
  ],

  "promptImages": [
    {
      "url": "image/prompt/lahir.jpg",
      "role": "prompt",
      "embeddedData": {
        "format": "DINOV2_ORB_V1",
        "dinov2Dims": 384,
        "orbKeypointCount": 100,
        "timestamp": "2026-08-10T22:42:30.062923",
        "_note": "Feature vector appended to JPEG tail. Read using FeaturePayload.parseFromMetadataString()."
      }
    }
  ],

  "videoUrls": [],
  "socialLinks": [],

  "sentences": [
    {
      "text": {
        "en": "...",
        "ms": "...",
        "ml": "...",
        "ta": "..."
      }
    }
  ]
}
```

---

## Field Descriptions

| Field | Type | Required | Notes |
|:---|:---|:---|:---|
| `id` | string | ✅ | Snake_case, semantic, unique |
| `name` | i18n object | ✅ | Must have at least `en` |
| `description` | i18n object | ✅ | Must have at least `en` |
| `latitude` | float | ✅ | WGS84 |
| `longitude` | float | ✅ | WGS84 |
| `altitude` | float | ❌ | Metres above sea level |
| `type` | string | ✅ | See **Types** below |
| `chronologyDate` | string\|null | ❌ | ISO 8601 date `YYYY-MM-DD` |
| `osmId` | string | ❌ | e.g. `node/6844620719` |
| `imageUrls` | array | ✅ | Display images; always use object format |
| `promptImages` | array | ❌ | AI/AR prompt images (see below) |
| `videoUrls` | array | ✅ | Can be empty `[]` |
| `socialLinks` | array | ✅ | Can be empty `[]` |
| `sentences` | array | ❌ | For narration/subtitle text |

---

## Location Types

| Value | Used for |
|:---|:---|
| `Holy Site` | Kaabah, Hajar Aswad, Jamarah |
| `Mosque` | All mosques |
| `Historical` | Historical sites, mountains, battlefields |
| `Miqat` | Ihram boundary points |

---

## Prompt Images (`promptImages`)

Stored in: `image/prompt/` folder (in `tour_data` git repo)

The images contain embedded AI feature data appended to the JPEG tail:
- **Format**: `DINOV2_ORB_V1::{json_payload}` appended after the JPEG end-of-image marker
- **DINOv2 embedding**: 384-dimensional float vector (from `dinov2_small.onnx`)
- **ORB keypoints**: up to 100 keypoints, each with x/y/size/angle/response + 32-byte BRIEF descriptor (base64)
- **Parsing**: Use `FeaturePayload.parseFromMetadataString()` from `myembedder` project

The `embeddedData` object in `locations.json` is **metadata only** — it describes what is embedded in the image, it does NOT duplicate the actual vectors.

Future: The `embeddedData` field may evolve to include a compact fingerprint hash to detect stale embeddings.

---

## ID Map Reference (old → new)

| Old ID | New ID |
|:---|:---|
| `l1` | `masjid_haram` |
| `l2` | `jabal_rahmah` |
| `l3` | `miqat_dhul_hulaifah` |
| `l4` | `masjid_qiblatayn` |
| `l5` | `jabal_nour_hira` |
| `l6` | `masjid_nabawi` |
| `l7` | `safa_marwah` |
| `l8` | **DELETED** (test data) |
| `l9` | **DELETED** (test data) |
| `l10` | `jabal_uhud` |
| `l11` | **DELETED** (duplicate of `jabal_rumat`) |
| `l12` | `masjid_quba` |
| `l13` | `masjid_jumuah` |
| `l14` | `masjid_shaikhain` |
| `l15` | `masjid_mustarahah` |
| `l16` | `masjid_fatah` |
| `l17` | `masjid_fasah` |

---

## Adding New Locations

1. Add entry to `d:\git\tour_data\locations.json`
2. Use a semantic human-readable ID
3. Include all 4 language keys for `name` and `description`
4. If you have an AI prompt image, run `myembedder` to generate the embedding and save to `image/prompt/`
5. Add `promptImages` entry with `embeddedData` describing the format
6. Transfer to app via nLite update (do NOT edit `assets/data/locations.json` directly)
