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

Firstly a check happens whether or not an audiomanager already exists; if not it will keep this one otherwise it will destroy the gameObject. Next it gives the audiomanager a don't destroy on load tag. This makes sure that audio will always be managed by the same manager no matter where in the code. Lastly every soundclip is instantiated and linked to the proper resources for playing later on. 

##### Start
In the `Start()` method some necessary methods are called:
```csharp
        UpdateSoundtrack();
        PlaySoundtrack();
        Invoke("NextSong", Soundtrack[CurrentlyPlaying].Source.clip.length);
```

The only thing worth noting here is the Invoke function. This starts an asynchronous timer afterwhich the "NextSong" function will be called.

##### Play
The `Play()` method is the main component of playing sounds:
```csharp
        Sound Clip = Array.Find(Sounds, Sound => Sound.Name == name);

        if (Clip == null)
        {
            Debug.LogWarning("Sound: " + name + "does not exist in soundlist");
        }

        Clip.Source.Play();
```

Firstly it will check whether or not a sound exists with the given parameter as name. If not it will send a warning otherwise it will play the sounds

##### UpdateSoundtrack
The `UpdateSoundtrack()` method populates the soundtrack list with every song depending on whether or not the necessary achievement has been achieved.:
```csharp
        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Under The Sea"));
        for (int i = 0; i < AchievementManager.instance.States.Count; i++)
        {
            if (AchievementManager.instance.States[i].Achieved)
            {
                switch (i)
                {
                    case 0:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Fathoms Below"));
                        break;
                    case 1:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Main Titles"));
                        break;
                    case 2:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Fanfare"));
                        break;
                    case 3:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Daughters Of Triton"));
                        break;
                    case 4:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Part Of Your World"));
                        break;
                    case 5:
                        break;
                    case 6:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Part Of Your World Reprise"));
                        break;
                    case 7:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Poor Unfortunate Souls"));
                        break;
                    case 8:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Les Poissons"));
                        break;
                    case 9:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Kiss The Girl"));
                        break;
                    case 10:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Fireworks"));
                        break;
                    case 11:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Jig"));
                        break;
                    case 12:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "The Storm"));
                        break;
                    case 13:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Destruction Of The Grotto"));
                        break;
                    case 14:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Flotsam And Jetsam"));
                        break;
                    case 15:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Tour Of The Kingdom"));
                        break;
                    case 16:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Bedtime"));
                        break;
                    case 17:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Wedding Announcement"));
                        break;
                    case 18:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Eric To The Rescue"));
                        break;
                    case 19:
                        Soundtrack.Add(Array.Find(Sounds, sound => sound.Name == "Happy Ending"));
                        break;
                    default:
                        break;
                }
            }
        }
```

##### PlaySoundtrack
In the `PlaySoundtrack()` method the actual soundtrack is started and will start playing:
```csharp
        Soundtrack[0].Source.Play();
        CurrentlyPlaying = 0;
```

##### NextSong
The `NextSong()` method will stop all currently playing songs and cancel the invoke function to automatically start playing the next song. Next it wil increment the index and start the new song followed by a new invoke function to play the next song when this one is finished:
```csharp
        CancelInvoke();
        Soundtrack[CurrentlyPlaying].Source.Stop();
        CurrentlyPlaying = (CurrentlyPlaying + 1) % Soundtrack.Count;
        Soundtrack[CurrentlyPlaying].Source.Play();
        float length = Soundtrack[CurrentlyPlaying].Source.clip.length + 0.5f;
        Invoke("NextSong", length);
```

##### PreviousSong
The `PreviousSong()` method will stop all currently playing songs and cancel the invoke function to automatically start playing the next song. Next it wil decrement the index and start the new song followed by a new invoke function to play the next song when this one is finished:
```csharp
        CancelInvoke();
        Soundtrack[CurrentlyPlaying].Source.Stop();
        if (CurrentlyPlaying == 0)
        {
            CurrentlyPlaying = Soundtrack.Count -1;
        }
        else
        {
            CurrentlyPlaying--;
        }
        Soundtrack[CurrentlyPlaying].Source.Play();
        float length = Soundtrack[CurrentlyPlaying].Source.clip.length + 0.5f;
        Invoke("NextSong", length);
```

##### Pause
The `Pause()` method will cancel the invoke function before doing a check whether or not the song is currently paused or not. If it isn't it will pause it otherwise it will resume palying the song:
```csharp
        CancelInvoke();
        if (IsPaused())
        {
            Soundtrack[CurrentlyPlaying].Source.Pause();
        }
        else
        {
            Soundtrack[CurrentlyPlaying].Source.Play();
            float length = Soundtrack[CurrentlyPlaying].Source.clip.length + 0.5f;
            Invoke("NextSong", length);
        }
```

##### IsPaused
The `IsPaused()` method is a simple check to see if a song is paused:
```csharp
        if (Soundtrack[CurrentlyPlaying].Source.isPlaying)
        {
            return true;
        }
        else
        {
            return false;
        }
```

##### GetSongText
The `GetSongText()` method that returns the name of the song that is currently playing:
```csharp
        return Soundtrack[CurrentlyPlaying].Name;
```

##### SetMusicVolume
The `SetMusicVolume()` method sets the volume of the song depending on the value it gets from the sliders. The calculations are converting the value from the linear slider down to the logaritmic values the mixer uses:
```csharp
        MainMixer.SetFloat("MusicVolume", Mathf.Log10(volume) * 20);
```

##### SetSFXVolume
The `SetSFXVolume()` method sets the volume of the song depending on the value it gets from the sliders. The calculations are converting the value from the linear slider down to the logaritmic values the mixer uses:
```csharp
        MainMixer.SetFloat("SFXVolume", Mathf.Log10(volume) * 20);
```

## MenuMaster
### ButtonHandler.cs
#### Variables
| Variable | Explanation |

| :--- | :--- |
| `GameObject MenuPanel` | The menu panel. |
| `GameObject CreditsPanel` | The credits panel. |
| `GameObject MusicPanel` | The music panel. |
| `Button MusicPauseButton` | The pause music button. |
| `Sprite Paused` | The paused sprite. |
| `Sprite Playing` | The playing sprite. |
| `Text MusicTitle` | The name of the song that is currently playing. |
| `AudioManager AudioMaster` | The Audiomanager in the scene. |

#### Methods
##### Awake
The `Awake()` method calls the connects the audiomaster to the buttonhandler:
```csharp
        AudioMaster = GameObject.Find("AudioMaster").GetComponent<AudioManager>();
```

##### Update
The `Update()` method updates the text of the song that is playing:
```csharp
        MusicTitle.text = AudioMaster.GetSongText();
```

##### LoadScene
The `LoadScene()` method loads in a new scene:
```csharp
        SceneManager.LoadScene(scene);
```

##### ToggleMenuPanels
The `ToggleMenuPanels()` method toggles the corresponding panel:
```csharp
        switch (panel)
        {
            case "credits":
                MenuPanel.SetActive(false);
                CreditsPanel.SetActive(true);
                AchievementManager.instance.Unlock("credits");
                break;
            case "menu":
                SceneManager.LoadScene("MainMenu");
                break;
            case "music":
                MusicPanel.SetActive(!MusicPanel.activeSelf);
                AchievementManager.instance.Unlock("volume");
                break;
            default:
                break;
        }
```

##### QuitGame
The `QuitGame()` method quits the game:
```csharp
        Application.Quit();
```

##### NextSong
The `NextSong()` method calls the right methods from the audiomaster to simplify the User experience:
```csharp
        AudioMaster.UpdateSoundtrack();
        AudioMaster.NextSong();
```

##### PreviousSong
The `PreviousSong()` method calls the right methods from the audiomaster to simplify the User experience:
```csharp
        AudioMaster.UpdateSoundtrack();
        AudioMaster.PreviousSong();
```

##### Pause
The `Pause()` method calls the right methods from the audiomaster to simplify the User experience:
```csharp
        bool temp = AudioMaster.IsPaused();
        if (temp)
        {
            MusicPauseButton.image.sprite = Paused;
        }
        else
        {
            MusicPauseButton.image.sprite = Playing;
        }
        AudioMaster.Pause();
```

### SequenceAnimator.cs
#### Variables
| Variable | Explanation |

| :--- | :--- |
| `GameObject AnimationContainer` | The container that encompases all animated items. |
| `List<Animator> Animators` | The list of animated items. |

#### Methods
##### Start
The `Start()` method initialises the title animation:
```csharp
        Animators = new List<Animator>(AnimationContainer.GetComponentsInChildren<Animator>());

        StartCoroutine(DoAnimation());
```

##### DoAnimation
The `DoAnimation()` method animates each letter independently in sequence:
```csharp
        while (true)
        {
            foreach (var animator in Animators)
            {
                animator.SetTrigger("DoAnimation");
                yield return new WaitForSeconds(1f);
            }

            yield return new WaitForSeconds(3f);
        }
```

### SliderHandler.cs
#### Variables
| Variable | Explanation |

| :--- | :--- |
| `AudioManager AudioMaster` | The audiomanager. |
| `Slider Music` | The music slider. |
| `Slider SFX` | The sfx slider. |

#### Methods
##### Awake
The `Awake()` method initialises the title animation:
```csharp
        AudioMaster = GameObject.Find("AudioMaster").GetComponent<AudioManager>();

        UpdateSlider();
```

##### UpdateSlider
The `UpdateSlider()` method sets the slidervalues to their correct position corresponding to the value it gains from the mixer. Due to the mixer being logaritmical and the sliders being linear a small correction is needed before setting the value:
```csharp
        float tempVolume;

        AudioMaster.MainMixer.GetFloat("MusicVolume", out tempVolume);
        Music.value = Mathf.Pow(10, (tempVolume / 20));
        AudioMaster.MainMixer.GetFloat("SFXVolume", out tempVolume);
        SFX.value = Mathf.Pow(10, (tempVolume / 20));
```

##### SetMusicVolume
The `SetMusicVolume()` method calls the corresponding function on the audiomanager:
```csharp
        AudioMaster.SetMusicVolume(volume);
```

##### SetSFXVolume
The `SetSFXVolume()` method calls the corresponding function on the audiomanager:
```csharp
        AudioMaster.SetSFXVolume(volume);
```

[Back to main](/index.md)

<< [Player Controls](/PlayerControls.md) << >> [Assets used](/Assets.md) >>
