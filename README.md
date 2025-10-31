# Unity Essentials

**Unity Essentials** is a lightweight, modular utility namespace designed to streamline development in Unity. 
It provides a collection of foundational tools, extensions, and helpers to enhance productivity and maintain clean code architecture.

## 📦 This Package

This package is part of the **Unity Essentials** ecosystem.  
It integrates seamlessly with other Unity Essentials modules and follows the same lightweight, dependency-free philosophy.

## 🌐 Namespace

All utilities are under the `UnityEssentials` namespace. This keeps your project clean, consistent, and conflict-free.

```csharp
using UnityEssentials;
```

---

# Unity Essentials – Core AudioMixer

UnityEssentials.Core.AudioMixer is a tiny, ready‑to‑use Audio Mixer setup you can drop into any project. It ships with a preconfigured mixer, common groups, and exposed volume parameters so you can hook up audio quickly in code and in the Editor.
~~~~
- Mixer asset: `Assets/Unity/Unity.Core.AudioMixer/Resources/UnityEssentials_AudioMixer.mixer`
- Groups: Master, Music, Effects, Voice, Environment
- Exposed parameters: `master`, `music`, `effects`, `voice`, `environment`

## Requirements
- Unity: 6.0 (per `package.json`) or newer
- No external dependencies

## Why use it
- Consistent naming across projects (low friction for scripts/tools)
- Lives in `Resources` so it can be loaded at runtime without scene references
- Sensible defaults and a clean layout for most games/apps

## Getting started

1) Add the folder to your project (it already lives under `Assets/Unity/Unity.Core.AudioMixer`).
2) Open the mixer in the Audio Mixer window to review/modify groups: Window > Audio > Audio Mixer.
3) Route your `AudioSource`s to the right groups and control volumes via exposed parameters.

## Loading the mixer at runtime
The mixer is placed under `Resources`, so you can load it by name:

```csharp
using UnityEngine;
using UnityEngine.Audio;

public static class MixerRef
{
    public const string MixerPath = "UnityEssentials_AudioMixer"; // Resources path without extension

    public static AudioMixer Load()
    {
        return Resources.Load<AudioMixer>(MixerPath);
    }
}
```

## Routing AudioSources to groups
You can assign the correct output group so each source is affected by the right volume control:

```csharp
using UnityEngine;
using UnityEngine.Audio;

public class RouteToMusic : MonoBehaviour
{
    public AudioSource source;

    void Awake()
    {
        var mixer = MixerRef.Load();
        if (!mixer || !source) return;

        // You can search by group name or path (e.g., "Master/Music")
        var groups = mixer.FindMatchingGroups("Music");
        if (groups != null && groups.Length > 0)
            source.outputAudioMixerGroup = groups[0];
    }
}
```

Common group names available out of the box:
- Master
- Music
- Effects
- Voice
- Environment

## Controlling volumes in code (exposed parameters)
The mixer exposes five float parameters representing group volumes in decibels (dB):
- `master`
- `music`
- `effects`
- `voice`
- `environment`

Unity’s AudioMixer parameters are in dB. A handy pattern is to convert 0..1 sliders to dB using `Mathf.Log10(value) * 20`, clamped so `0` maps to something like `-80 dB` (silence):

```csharp
using UnityEngine;
using UnityEngine.Audio;

public static class AudioUtils
{
    public static float Linear01ToDb(float value)
    {
        // Map [0,1] -> [-80, 0] dB (unity mixers treat around -80 as silence)
        if (value <= 0.0001f) return -80f;
        return Mathf.Log10(Mathf.Clamp01(value)) * 20f;
    }
}

public class SetVolumesExample : MonoBehaviour
{
    [Range(0,1)] public float master = 1f;
    [Range(0,1)] public float music = 1f;
    [Range(0,1)] public float sfx = 1f;
    [Range(0,1)] public float voice = 1f;
    [Range(0,1)] public float environment = 1f;

    AudioMixer mixer;

    void Awake()
    {
        mixer = MixerRef.Load();
    }

    void Update()
    {
        if (!mixer) return;
        mixer.SetFloat("master",      AudioUtils.Linear01ToDb(master));
        mixer.SetFloat("music",       AudioUtils.Linear01ToDb(music));
        mixer.SetFloat("effects",     AudioUtils.Linear01ToDb(sfx));
        mixer.SetFloat("voice",       AudioUtils.Linear01ToDb(voice));
        mixer.SetFloat("environment", AudioUtils.Linear01ToDb(environment));
    }
}
```

Reading the current value back (in dB):

```csharp
float db;
if (mixer.GetFloat("music", out db))
{
    // db is the current volume in dB for the music group
}
```

## Snapshots and transitions
A default `Snapshot` is included. You can create more snapshots in the Audio Mixer window and transition at runtime:

```csharp
using UnityEngine;
using UnityEngine.Audio;

public class SnapshotSwitch : MonoBehaviour
{
    public string snapshotName = "Snapshot";
    public float transitionSeconds = 0.5f;

    AudioMixerSnapshot snapshot;

    void Start()
    {
        var mixer = MixerRef.Load();
        if (!mixer) return;
        var snaps = mixer.FindSnapshot(snapshotName);
        if (snaps != null)
        {
            snapshot = snaps;
        }
    }

    public void Apply()
    {
        if (snapshot)
            snapshot.TransitionTo(transitionSeconds);
    }
}
```

Tip: You can also use `AudioMixer.TransitionToSnapshots` for complex blends.

## Editor tips
- To expose a parameter: click a group’s volume/pitch knob in the Audio Mixer and check the “Exposed to script” toggle; name it consistently (lowercase).
- To visualize routing: select an `AudioSource` and ensure its Output is one of the groups above.
- If you rename groups or parameters, update your code and saved snapshots accordingly.

## Asset layout
- `Resources/UnityEssentials_AudioMixer.mixer` – the main mixer asset (auto-included in builds)
- Groups under Master: Music, Effects, Voice, Environment
- Each group has an Attenuation (volume) effect and an exposed volume parameter as listed above

## Troubleshooting
- “Resources.Load returned null”: ensure the asset path is exactly `Resources/UnityEssentials_AudioMixer.mixer` and you’re loading `"UnityEssentials_AudioMixer"` (no extension).
- “Group not found”: verify names in the Audio Mixer window; `FindMatchingGroups("Master/Effects")` also works if there are duplicates.
- “Volume not changing”: confirm you used the parameter name (e.g., `"music"`) not the group name; parameters are case-sensitive.

## License
See repository license. If this folder is embedded in your project, it inherits your project’s license unless stated otherwise.

## Namespace note
You can organize your scripts under the `UnityEssentials` namespace for consistency across Essentials modules:

```csharp
namespace UnityEssentials { /* your scripts */ }
```
