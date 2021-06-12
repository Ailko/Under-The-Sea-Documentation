# School

The schools of fish are split up in 3 seperates scripts:
- `WaypointProgressTracker.cs` from the standard assets of Unity (which we won't discuss here).
- `School.cs`, which is used for the parent object grouping the fish together.
- `SchoolFishScript.cs`, which is used for the fish themselves.

## School.cs

### Variables

| Variable | Explanation |

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

This class uses 1 method:
- `void Start()`

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


Then we decide the scale of the fish by multiplying it with a random number between 1 - the determined scale variance and 1 + the determined scale variance.

After this we set the speed to a random number between the minSpeed and maxSpeed and the same principle is applied to the rotational speed.

## SchoolFishScript.cs
### Variables

| Variable | Explanation |

| :--- | :--- |
| `float speed` | This variable determines the maximal speed a fish can have relative to the school. |
| `float rotSpeed` | This variable determines how fast a fish can turn. |
| `float moveTimeOut` | This variable determines how long a fish goes in timeout when they exit the allowed radius. |
| `float timer` | This variable is used when a fish enters timeout. |
| `Vector3 target` | This variable is the position the fish tries to swim towards. |
| `Vector3 direction` | This variable tracks what direction the fish is moving in. |

### Methods

This class uses 3 methods:
- `void Start()`
- `void Update()`
- `Vector3 NewTarget()`

We will first look at the `newTarget()` method as this one is used by the other.

#### NewTarget
This is a fairly short and straightforward method.


The code:
```csharp
private Vector3 NewTarget()
{
	Vector3 NewTarget = new Vector3(Random.Range(-1f, 1f), Random.Range(-1f, 1f), Random.Range(-1f, 1f)).normalized;
	NewTarget *= GetComponentInParent<School>().schoolRadius * Random.Range(0f, 1f);
	return NewTarget;
}
```


The new target method returns, you guessed it, a new target for the fish. This is achieved by instantiating a Vector3 with random x, y and z coordinates between -1 and 1. This Vector3 is then normalized and multiplied with the 
radius of the parent (the school) and then multiplied with a random number between 0 and 1 so the target isn't always at the edges of the school.

#### Start
The `Start()` method in this class is only 1 line long.

The code:
```csharp
private void Start()
{
	target = NewTarget();
}
```


At initialisation the only thing that happens is that the fish is given a first target.

#### Update

The `Update()` method exists of multiple sections. First we will show the entire method and then we will discuss the seperate parts:

```csharp
void Update()
{
	if (Vector3.Distance(target, transform.localPosition) < 0.1f)
	{
		target = NewTarget();
		timer = moveTimeOut;
	}

	if (timer <= 0)
	{
		if (transform.localPosition.magnitude > GetComponentInParent<School>().schoolRadius)
		{
			target = Vector3.zero;
			timer = moveTimeOut;
		}
	
		direction = Vector3.MoveTowards(transform.localPosition, target, speed * Time.deltaTime);

		transform.localPosition = direction;

		if (direction.z > 0)
			direction.z = -180 + direction.z;
	
		transform.localRotation = Quaternion.Lerp
			(
				transform.localRotation,
				Quaternion.Euler(0, Quaternion.LookRotation(direction).eulerAngles.y + 90, 0),
				rotSpeed * Time.deltaTime
			);
	}
	else
	{
		timer -= Time.deltaTime;
		GetComponent<Rigidbody>().AddForce(transform.parent.transform.position - transform.position);
	}
		
	if (transform.localPosition.magnitude > GetComponentInParent<School>().schoolRadius)
		transform.localPosition = transform.localPosition.normalized * GetComponentInParent<School>().schoolRadius;
}
```


The first section is for checking wether the fish has reached it's target.
```csharp
if (Vector3.Distance(target, transform.localPosition) < 0.1f)
{
	target = NewTarget();
	timer = moveTimeOut;
}
```
The timeout is applied so the faster fishes aren't constantly zipping around the school since this would look rather unnatural.


Then a check is performed for wether the fish is in timeout, if it is, the timer will be decreased and a force towards the center of the school is applied.
```csharp
timer -= Time.deltaTime;
GetComponent<Rigidbody>().AddForce(transform.parent.transform.position - transform.position);
```


If the fish is not in timeout a check is performed on wether the fish is too far removed from the center of the school. If so, it's target is set to the center of the school and a timeout is applied.
```csharp
if (transform.localPosition.magnitude > GetComponentInParent<School>().schoolRadius)
{
	target = Vector3.zero;
	timer = moveTimeOut;
}

```

Then it's direction is determined. This is the local position it will next move towards.
```csharp
direction = Vector3.MoveTowards(transform.localPosition, target, speed * Time.deltaTime);
``` 


Then the local position is update.
```csharp
transform.localPosition = direction;
```


If the z direction is larger than 0 it will then be set to the mirrored negative direction. This is done so the fish don't turn around and appear to be swimming in the opposite direction the school is swimming, appearing to swim 
backwards.
```csharp
if (direction.z > 0)
	direction.z = -180 + direction.z;
```


Then the fish's rotation is set. Only allowing it to turn around it's y axis.
```csharp
transform.localRotation = Quaternion.Lerp
(
	transform.localRotation,
	Quaternion.Euler(0, Quaternion.LookRotation(direction).eulerAngles.y + 90, 0),
	rotSpeed * Time.deltaTime
);
```
90 is added to the y rotation since the models are turned 90 degrees from looking forward.


All the way at the end of the file a last check for being outside the school is done, if the fish is found to be outside of the schoolradius, it will be teleported back towards the border of the school.
```csharp
if (transform.localPosition.magnitude > GetComponentInParent<School>().schoolRadius)
	transform.localPosition = transform.localPosition.normalized * GetComponentInParent<School>().schoolRadius;
```

[Back to main](/index.md)

| Previous | Next |

| :--- | ---: |
| [Main](/index.md) | [Player Controls](/PlayerControls.md) |
