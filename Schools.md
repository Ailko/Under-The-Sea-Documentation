# School

The schools of fish are split up in 3 seperates scripts:
- `WaypointProgressTracker.cs` from the standard assets of Unity (which we won't discuss here).
- `School.cs`, which is used for the parent object grouping the fish together.
- `SchoolFishScript.cs`, which is used for the fish themselves.

## School.cs

### Variables
| Variables | Explanation |
| :--- | :--- |
| `GameObject[] fishVariations`	| This is an array of all the different models of fish allow in this school. |
| `float minSpeed` | Relative minimum speed a fish is allowed to have in this school. |
| `float maxSpeed` | Relative maximum speed a fish is allowed to have in this school. |
| `float minRotSpeed` | Minimum rotation speed a fish is allowed to have in this school. |
| `float maxRotSpeed` | Maximum rotation speed a fish is allowed to have in this school. |
| `float scaleVariance` | The variance in size the fish models are allowed to have. |
| `string fishScript` | The script for the fishes in this school. By default this will be set to the standard script. |
| `float schoolRadius` | The maximum distance away from the center of the school a fish is allowed to swim. |
| `int schoolSize` | The amount of fish in this school. |

### Methods
#### Start
The private `Start()` method consists of two parts:
- The creation
- The variable setting


We will be discussing both.


First, the creation:


This part of the `School.cs` script will create the fishes.


Here's the code:
```csharp
for(int i = 0; i < schoolSize; i++)
{
	int variation = Random.Range(0, fishVariations.Length);
	Instantiate
	(
		fishVariations[variation],
		transform.position + new Vector3
		(
			Random.Range(-1f, 1f),
			Random.Range(-1f, 1f),
			Random.Range(-1f, 1f)
		).normalized * Random.Range(0.0f, schoolRadius),
		Quaternion.Euler(transform.rotation.eulerAngles + Quaternion.Euler(0, -90, 0).eulerAngles),
		transform
	);
}
```


It will select a fish variation at random and instantiate this with the Unity `Instantiate<GameObject>(GameObject original, Vector3 position, Quaternion 
rotation, Transform parent)` method. We will cover each variable of the `Instantiate` function:
- `GameObject original`: Here the randomly selected fish variation is passed.
- `Vector3 position`: The worlposition is a random position relative to the school inside the radius. This is achieved by creating a Vector3 with x, y and z 
coordinates randomly selected between 0 and 1. This Vector3 is then normalized and multiplied with the radius to keep it inbounds and upscaled as large as is 
necessary to fill the radius. This is then added to the school's position.
- `Quaternion rotation`: The rotation at initialisation is the same for all the fishes, straight ahead.
- `Transform parent`: As the parent the school's transform is passed.


The second part of the `Start()` method is used for assigning variables to the SchoolFishScript. It looks a bit daunting but it will be much clearer after 
the explanation.


The code:
```csharp
for(int i = 0; i < transform.childCount; i++)
{
	if (!transform.GetChild(i).GetComponent<Camera>())
	{
		try
		{
			System.Type type = System.Type.GetType(fishScript);
			if (type.IsSubclassOf(typeof(SchoolFishScript)))
				transform.GetChild(i).gameObject.AddComponent(type.GetType());
			else
				throw new System.Exception("Wrong script type");
		}
		catch (System.Exception)
		{
			transform.GetChild(i).gameObject.AddComponent<SchoolFishScript>();
		}
		transform.GetChild(i).gameObject.transform.localScale *= Random.Range(1 - scaleVariance, 1 + scaleVariance);
		transform.GetChild(i).gameObject.GetComponent<SchoolFishScript>().speed = Random.Range(minSpeed, maxSpeed);
		transform.GetChild(i).gameObject.GetComponent<SchoolFishScript>().rotSpeed = Random.Range(minRotSpeed, maxRotSpeed);
	}
}
```


Since there is a possiblity of adding cameras as children of schools of fish for cinematics we check if the child is a camera, if it isn't we continue.


First we try to get a script with the entered name. If this fails the default script is applied. If a script with that name exists we check if it is a 
subclass of the base SchoolFishScript, once again, if this fails, the base script is applied. If none of the checks fail the script is added as a component 
to the fish GameObject.


Then we decide the scale of the fish by multiplying it with a random number between 1 - the determined scalevariance and 1 + the determined scale variance.

After this we set the speed to a random number between the minSpeed and maxSpeed and the same principle is applied to the rotational speed.
