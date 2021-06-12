[Back to main](/index.md)

# User Interface

The UI functionality consists of multiple scripts that work together to create a fluent User experience

## AudioMaster
### Sound.cs

| Variable | Explanation |

| :--- | :--- |
| `string Name` | The name of the clip. |
| `float Volume` | The volume the clip will be played at. |
| `float Pitch` | The pitch the clip will be played at. |
| `bool Loop` | Whether or not the clip should be played in a loop. |
| `Type SoundType` | The type of clip (can be Music or SFX). |
| `AudioSource Source` | The actual sound file. |

### AudioManager.cs
#### Variables
| Variable | Explanation |

| :--- | :--- |
| `Sound[] Sounds` | An array of sounds used in the game. |
| `AudioMixer MainMixer` | The mixer used for managing sound. |
| `List<Sound> Soundtrack` | A list of background songs that have been unlocked. |
| `int CurrentlyPlaying` | The index of which sound is currently playing. |

#### Methods
##### Awake
In the `Awake()` method everything is initialised:
```csharp
        if (Globals.MainAudioManager == null)
        {
            Globals.MainAudioManager = this;
        }
        else
        {
            Destroy(gameObject);
            return;
        }
        DontDestroyOnLoad(gameObject);
        
        foreach (Sound Clip in Sounds)
        {
            Clip.Source = gameObject.AddComponent<AudioSource>();
            Clip.Source.clip = Clip.Clip;
            Clip.Source.volume = Clip.Volume;
            Clip.Source.pitch = Clip.Pitch;
            Clip.Source.loop = Clip.Loop;

            if ((int)Clip.SoundType == 0)
            {
                Clip.Source.outputAudioMixerGroup = MainMixer.FindMatchingGroups("Music")[0];
            }
            else if ((int)Clip.SoundType == 1)
            {
                Clip.Source.outputAudioMixerGroup = MainMixer.FindMatchingGroups("SFX")[0];
            }
            else
            {
                Clip.Source.outputAudioMixerGroup = MainMixer.FindMatchingGroups("Master")[0];
            }
        }
```
