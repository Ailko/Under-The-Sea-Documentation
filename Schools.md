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
