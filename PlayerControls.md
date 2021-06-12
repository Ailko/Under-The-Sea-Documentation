[Back to main](/index.md)

# Player Controls

The player's controls are a relatively simple script, it only has 2 methods:
- `Start`
- `Update`

## PlayerMovement.cs

### Variables

| Variable | Explanation |

| :--- | :--- |
| `float Hsensitivity` | The horizontal sensitivity of the mouse. |
| `float Vsensitivity` | The vertical sensitivity of the mouse. |
| `float thrust` | The thrust power the submarine has. |

### Methods

#### Start
In the `Start()` method there is only one line of code:
```csharp
Cursor.lockState = CursorLockMode.Locked;
```
This line of code keeps the cursor in the middle of the screen while also allowing us to read it's movements, thus preventing it from reaching the border of the screen which would cause problems when turning.

#### Update
The `Update()` method exists out of 2 parts, the rotation and the movement. First I will show the entire method:
```csharp
void Update()
{
    float yaw = transform.eulerAngles.y + Input.GetAxis("Mouse X") * Hsensitivity;
    float pitch = transform.eulerAngles.x - Input.GetAxis("Mouse Y") * Vsensitivity;

    transform.eulerAngles = new Vector3(pitch, yaw, 0f);

    Vector3 movement = Vector3.zero;

    movement += transform.forward * Input.GetAxis("Vertical");
    movement += transform.right * Input.GetAxis("Horizontal");
    movement += transform.up * Input.GetAxis("Jump");
    movement -= transform.up * Input.GetAxis("Fire1");

    GetComponent<Rigidbody>().AddForce(movement * thrust);
}
```

##### Rotation
Code:
```csharp
float yaw = transform.eulerAngles.y + Input.GetAxis("Mouse X") * Hsensitivity;
float pitch = transform.eulerAngles.x - Input.GetAxis("Mouse Y") * Vsensitivity;

transform.eulerAngles = new Vector3(pitch, yaw, 0f);
```

First we calculate the new yaw and pitch of the submarine by first getting the current eulerangles and then adding the mouse's horizontal and vertical movement to them (multiplied with the sensitivity). Then the pitch and jaw are set as the x and y of the eulerangles respectively.

##### Movement
Code:
```csharp
movement += transform.forward * Input.GetAxis("Vertical");
movement += transform.right * Input.GetAxis("Horizontal");
movement += transform.up * Input.GetAxis("Jump");
movement -= transform.up * Input.GetAxis("Fire1");

GetComponent<Rigidbody>().AddForce(movement * thrust);
```

The movement simply takes the input axises and multiplies them by the submarine's transform vectors. Then the combined vector is multiplied with the thrust and addes to the Rigidbody's force.

[Back to main](/index.md)


<< [School](/Schools.md) << >>  >>
