# Unity Essentials

This module is part of the Unity Essentials ecosystem and follows the same lightweight, editor-first approach.
Unity Essentials is a lightweight, modular set of editor utilities and helpers that streamline Unity development. It focuses on clean, dependency-free tools that work well together.

All utilities are under the `UnityEssentials` namespace.

```csharp
using UnityEssentials;
```

## Installation

Install the Unity Essentials entry package via Unity's Package Manager, then install modules from the Tools menu.

- Add the entry package (via Git URL)
    - Window → Package Manager
    - "+" → "Add package from git URL…"
    - Paste: `https://github.com/CanTalat-Yakan/UnityEssentials.git`

- Install or update Unity Essentials packages
    - Tools → Install & Update UnityEssentials
    - Install all or select individual modules; run again anytime to update

---

# Audio Mixer

> Quick overview: Drop‑in AudioMixer asset with common groups and exposed volume parameters: `master`, `music`, `effects`, `voice`, and `environment`. Load from Resources, route AudioSources to groups, and drive volumes via `SetFloat` in decibels.

A ready‑to‑use `AudioMixer` shipped under `Resources/`. It defines a Master bus with Music, Effects, Voice, and Environment child groups, and exposes volume parameters for each (plus master). Use Unity’s built‑in audio APIs to assign mixer groups to your `AudioSource`s and adjust volumes at runtime in dB.

![screenshot](Documentation/Screenshot.png)

## Features
- Preconfigured mixer layout
  - Groups: Master → { Music, Effects, Voice, Environment }
- Exposed volume parameters (case‑sensitive)
  - `master`, `music`, `effects`, `voice`, `environment`
- Snapshot
  - Default snapshot named `Snapshot` set as the start target
- Suspend behavior
  - Suspension enabled with threshold −80 dB to reduce CPU when silent
- Lightweight
  - Asset‑only module; no runtime scripts

## Requirements
- Unity Editor 6000.0+
- Built‑in Audio system (`UnityEngine.Audio`)
- Asset location: `Resources/UnityEssentials_AudioMixer.mixer`
- Optional helpers
  - Utilities – Extensions: `ToDecibelLevel()` to map UI sliders (0–200) to decibels
  - Utilities – ResourceLoader: typed `LoadResource<AudioMixer>(...)`

Tip: Volume parameters are in decibels (dB). Typical ranges: −80 dB (mute) to +20 dB (boost). 0 dB is “unity gain”.

## Usage
Load the mixer (Resources)
```csharp
using UnityEngine;
using UnityEngine.Audio;

// Directly via Resources
AudioMixer mixer = Resources.Load<AudioMixer>("UnityEssentials_AudioMixer");

// Or via UnityEssentials.ResourceLoader (optional)
// var mixer = ResourceLoader.TryGet<AudioMixer>("UnityEssentials_AudioMixer");
```

Route an AudioSource to a group
```csharp
var groups = mixer.FindMatchingGroups("Music");
if (groups.Length > 0)
    GetComponent<AudioSource>().outputAudioMixerGroup = groups[0];
```

Set group volumes (dB)
```csharp
// Set music volume to -10 dB
mixer.SetFloat("music", -10f);

// Mute SFX (effects)
mixer.SetFloat("effects", -80f);

// Reset to 0 dB (unity)
mixer.SetFloat("master", 0f);

// Read back a value
if (mixer.GetFloat("voice", out float voiceDb))
    Debug.Log($"Voice: {voiceDb} dB");
```

Map UI slider to decibels (optional extension)
```csharp
// Using UnityEssentials.UtilityExtensions.ToDecibelLevel(): int 0..200 -> dB [-80..+20]
int uiVolume = 100; // 100 => 0 dB
float db = uiVolume.ToDecibelLevel();
mixer.SetFloat("environment", db);
```

Snapshots (optional)
- The asset includes a single snapshot `Snapshot`. You can add your own snapshots in the Mixer window and switch at runtime:
```csharp
var snaps = mixer.snapshots;           // define custom snapshots in the asset first
var weights = new float[snaps.Length];
// e.g., fully to snapshot 0 over 0.5 seconds
weights[0] = 1f;
mixer.TransitionToSnapshots(snaps, weights, 0.5f);
```

## How It Works
- The mixer asset exposes each group’s volume parameter by name so you can drive it via `AudioMixer.SetFloat(name, dB)`
- Groups are organized under a single Master; each child has a built‑in Attenuation effect for volume control
- Suspension is enabled at −80 dB so processing can pause when fully silent
- A default snapshot is provided; you can create and blend additional snapshots as needed in the Unity Mixer window

## Notes and Limitations
- Exposed parameter names are lowercase and case‑sensitive: `master`, `music`, `effects`, `voice`, `environment`
- Resource path for `Resources.Load` is `"UnityEssentials_AudioMixer"` (omit extension and `Resources/`)
- Assigning groups: `FindMatchingGroups("Music")` returns an array; pick index 0 unless you’ve duplicated names
- Loudness vs volume: UI sliders should convert to dB; linear 0..1 sliders feel non‑linear - use a mapping (e.g., `ToDecibelLevel()` or your own curve)
- No runtime scripts are included; this package supplies the mixer asset only
- Addressables: not integrated; if you move the asset out of `Resources`, adjust your loading approach

## Files in This Package
- `Resources/UnityEssentials_AudioMixer.mixer` – AudioMixer asset with groups and exposed parameters
- `package.json` – Package manifest metadata

## Tags
unity, audio, mixer, audiomixer, music, sfx, voice, environment, volume, decibel, snapshot, resources
