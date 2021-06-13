[Back to main](/index.md)

# Snapshot

The snapshot handler script is a small script that handles taking a snapshot of a camera. It has 5 methods:
- `Awake`
- `Update`
- `TakeSnapshot`
- `SnapshotName`
- `OnPostRender`

## SnapshotHandler.cs

### Variables

| Variable | Explanation |

| :--- | :--- |
| `Camera pictureCamera` | The camera that takes the snapshot. |
| `bool takeScreenshotOnNextFrame` | A boolean value that allows to take a snapshot on the next frame |
| `string snapshotName` | The name of the snapshot. |
| `string folderToSave` | The folder to save the taken snapshot. |
| `int width` | The width of the taken snapshot. |
| `int height` | The height of the taken snapshot. |

### Methods

#### Awake
The `Awake()` method sets the picture camera and the `NumberOfPicturesTaken` (a global variable), which tracks the number of snapshots the camera takes. 
```csharp
    private void Awake()
    {
        Globals.NumberOfPicturesTaken = 0;
        pictureCamera = gameObject.GetComponent<Camera>();
    }
```

#### Update
The `Update()` method gets user input and calls the `TakeSnapshot()` method.
```csharp
    private void Update()
    {
        if(Input.GetAxis("Fire2") != 0)
        {
            TakeSnapshot();
        }
    }
```

#### TakeSnapshot
The `TakeSnapshot()` method takes a snapshot.
```csharp
    public void TakeSnapshot()
    {
        // Determine path to save the taken picture
        Globals.NumberOfPicturesTaken++;
        snapshotName = SnapshotName();
        pictureCamera.targetTexture = RenderTexture.GetTemporary(width, height, 24);
        takeScreenshotOnNextFrame = true;
    }
```

#### SnapshotName
Will return a unique name for the snapshot with the following syntax: `picture_widthxheight_{ id }`.
```csharp
    private string SnapshotName()
    {
        return string.Format("{0}/{1}/picture_{2}x{3}_{4}.png", 
            Application.dataPath, 
            folderToSave, 
            width, height, 
            Globals.NumberOfPicturesTaken);
    }
```

#### OnPostRender
The `OnPostRender()` renders the taken snapshot and saves the in the correct folder.
```csharp
    private void OnPostRender()
    {
        if (takeScreenshotOnNextFrame)
        {
            takeScreenshotOnNextFrame = false;
            RenderTexture renderTexture = pictureCamera.targetTexture;

            // Take the screenshot
            Texture2D renderResult = new Texture2D(renderTexture.width, renderTexture.height, TextureFormat.ARGB32, false);
            Rect rect = new Rect(0, 0, renderTexture.width, renderTexture.height);
            renderResult.ReadPixels(rect, 0, 0);

            // Save the screenshot to the correct folder
            byte[] byteArray = renderResult.EncodeToPNG();
            System.IO.File.WriteAllBytes(snapshotName, byteArray);
            Debug.Log($"Picture taken! Saved { snapshotName }");

            RenderTexture.ReleaseTemporary(renderTexture);
            pictureCamera.targetTexture = null;
        }
    }
```

[Back to main](/index.md)

<< [User Interface](UI.md) << >> [Assets used](/Assets.md) >>
