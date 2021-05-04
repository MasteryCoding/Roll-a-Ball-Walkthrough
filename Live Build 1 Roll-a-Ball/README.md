# Live Build 1: Roll-a-Ball

## Step 1: Introductions

1. Explain what the class will be building today and emphasize that they can interrupt and ask questions if they're falling behind.
2. Show off finished project. [TODO: Include Link]
3. Create a new Unity Project named `Roll-a-Ball`.

## Step 2: Gather Assets

Download and import the [RAB_Assets.unitypackage](./Resources/RAB_Assets.unitypackage) from the section.

![Import](./Resources/Import.png)

Download Skyboxes from Unity Asset store.
Recommended Package: <https://assetstore.unity.com/packages/2d/textures-materials/sky/customizable-skybox-174576>

![Skyboxes](./Resources/Skyboxes.png)

Explain that Skyboxes change the background of the scene. You can drag any of these into the sky of the scene to change the skybox.

![Syboxes Project](./Resources/SkyboxesProjectView.png)

### RAB_Assets

The RAB Asset Pack is from Mastery Coding and free to use for any portfolio projects with attribution. Just include the text "RAB Asset Pack - Mastery Coding 2021" anywhere on the Github README.

This asset pack comes with three folders.

**Materials**: Contains the color data for all the models in the pack.

![Materials](./Resources/Materials.png)

**Models**: Contains the imported models and UV maps. Do not drag these into your scene directly.

![Models](./Resources/Models.png)

**Prefabs**: Contains all models with the correct material and collider settings.

### Organize Project

Create a new Folder called `Resources` and drag the `Customizeable Skybox`, `Models`, and `Materials` folders inside.

Your project window should look like this:

![Organized Project](./Resources/OrganizedProject.png)

You will be working inside the Prefabs folder mostly.

### Prefab List

#### Platforms

![Platforms](./Resources/Platforms.png)

Platforms make up the ground that the character travels on. They use **Mesh Colliders**.

#### Terrain

![Terrain](./Resources/Terrain.png)

Terrain is used to spice up the scene so the ground is not always flat. They also use **Mesh Colliders**.

#### Decor

![Decor](./Resources/Decor.png)

Decor can be used as an obstacle, or to make the scene prettier. There are three types of decor objects.

1. Trees
2. Rocks
3. Clouds

All of these have **Mesh Colliders** except for the clouds.

#### Items

![Crate](./Resources/Crate.png)

Crates are items you can put in the scene. The crate Prefab has a **Box Collider** and a non-kinematic **Rigidbody**.

## Step 3: Set Up Scene

* Rename the Sample Scene to `Level 1`
* Open the **Lighting Window**'s Environment tab and set desired Skybox. *Window > Rendering > Lighting*
* Pin the lighting window to the inspector.

![Lighting Skybox](./Resources/LightingSkybox.png)

* Turn the skybox off in the scene view.

![Toggle Skybox Off](./Resources/SkyboxOff.png)

## Step 4: Create A Level

Add an `XXL_Grass_Platform` to the scene. Make sure it is at position (0,0,0).

![SceneAddPlatform](./Resources/SceneAddPlatform.png)

Add `Terrain` prefabs to the platform. Make sure there are no overhangs and nothing is floating above the platform.

![SceneAddTerrain](./Resources/SceneAddTerrain.png)

Add `Decor` prefabs to spruce up the level.

![SceneAddDecor](./Resources/SceneAddDecor.png)

### Organize Hierarchy

Create an Empty GameObject called `Level` in the scene.

Create Empty GameObjects inside `Level` to group objects together. It should look something like this:

![Hierarchy](./Resources/HierarchyOrganization.png)

## Step 5: Set Up Lighting

In the lighting window's **Scene Tab**, create new lighting settings.

Set the following settings:

* Lightmapper: Progressive GPU (If computer has a good graphics card)
* Direct Samples: 16
* Indirect Samples: 200
* Environment Samples: 160
* Compress Lightmaps: False
* Ambient Occlusion: True
* Auto Generate: False

Return to Lighting > Environment and set:

* Environment Lighting
  * Intensity Multiplier: ~1
* Environment Reflections
  * Intensity Multiplier: ~.25
  * Resolution: 128
  * Compression: Uncompressed

Make any other environment tweaks to your scene, then click `Generate Lighting` and wait for the lightmaps to bake.

![GoodSkybox](./Resources/GoodSkybox.png)

A lot of the settings that were tweaked, are so the lighting generates faster. When we build our game for publishing, we can turn these values up and get a better looking scene.

Show Baked Lightmaps briefly. These will be covered in a later chapter.

## Step 6: Controlling Movement

Take the `Ball` model from *Assets > Resources > Models > Ball* and drag it into the Scene.

![Ball](./Resources/Ball.png)

**Note**: This is the only time we drag anything from Assets > Resources > Models.

Create a new folder in `Assets` called `Scripts`

* *Movement.cs*
* *InputHandler.cs*

![Scripts](./Resources/Scripts.png)

### Prep

Open *Movement.cs* and remove all unecessary code.

```cs
// Movement.cs
using UnityEngine;

public class Movement : MonoBehaviour
{
  
}
```

### Get RigidBody

1. Create a private Rigidbody property named rb
2. Import the Player **Rigidbody** inside the `Awake()` method.
3. Require the Rigidbody component.

```cs
// Movement.cs
[RequireComponent(typeof(Rigidbody))]
public class Movement : MonoBehaviour
{
  Rigidbody rb;
  void Awake()
  {
    rb = GetComponent<Rigidbody>();
  }
}
```

### Configure Player GameObject

1. Drag Movement Script onto the `Ball` GameObject.
2. Add a Rigidbody
3. Add a Sphere Collider
4. Set Tag to `Player`

![BallInspector](Resources/BallInspector.png)

### Move That Ball

Add a `Move()` Method to the Ball that:

1. Takes in a movement Vector.
2. Multiplies it by a serialized acceleration property.
3. Uses the result to add an acceleration vector to the Rigidbody.

```cs
// Movement.cs
[RequireComponent(typeof(Rigidbody))]
public class Movement : MonoBehaviour
{
  [SerializeField] float acceleration;
  Rigidbody rb;
  
  private void Awake()
  {
    rb = GetComponent<Rigidbody>();
  }

  public void Move(Vector3 moveDirection) {
    // Ensure vector is not greater that 1 in magnitude.
    if (moveDirection.magnitude > 1) moveDirection = moveDirection.normalized;

    // Multiply by acceleration
    Vector3 forceVector = moveDirection * acceleration;

    // Use the forceVector to add an acceleration to the ball. Ignoring its mass
    rb.AddForce(forceVector, ForceMode.Acceleration);
  }
```

**Note**: By using `ForceMode.Acceleration`, we can change the mass of the ball later without it affecting its movement.

**Note**: Recall that AddForce() does not require multiplying by Time.deltaTime.

### Set Up Input Handler

1. Create an empty GameObject called `GameLogic` in the Hierarchy.
2. Attach the *InputHandler.cs* script to it.
3. Remove all unnecessary logic.
4. Create and `Awake()` method and get a reference to the player with `GameObject.FindGameObjectWithTag()`.
5. Extract the Movement component from the player.

```cs
// InputHandler.cs
using UnityEngine;

public class InputHandler : MonoBehaviour
{
  private Movement playerMovement;
  private void Awake()
  {
    GameObject player = GameObject.FindGameObjectWithTag("Player");
    if (player == null)
    {
      Debug.LogError("No GameObject with 'Player' tag in scene.");
      return;
    }
    playerMovement = player.GetComponent<Movement>();
    if (playerMovement == null)
    {
      Debug.LogError("Player exists but does not have a Movement component.");
      return;
    }
  }
}
```

### Get Axes

Since we're working with forces for movement, we will be using `FixedUpdate()`

1. Create a `FixedUpdate()` method inside the `InputHandler` class.
2. Capture the Horizontal and Vertical axes in floats called `x` and `y`.
3. Call the `Move()` method with a new `Vector3` using x for x and y for z movement.

```cs
// InputHandler.cs
private void FixedUpdate() {
  // Get Movement Input Axes 
  float x = Input.GetAxis("Horizontal");
  float y = Input.GetAxis("Vertical");

  // Send inputs 
  playerMovement.Move(new Vector3(x, 0, y));
}
```

### Test Movement

In your project,

1. Point the `Main Camera` at your ball facing downwards.
2. Tweak the `acceleration` value of the Movement component on the Ball. ~600 is a good value.
3. Press start and try using WASD to move the ball.

![MovementTest](./Resources/MovementTest.gif)

If it works, great! If not, use this checklist for troubleshooting:

1. Your console isn't printing any error messages.
2. The `Ball` gameobject has a movement script attached and is tagged `player`.
3. Movement has an acceleration of ~600.
4. Your *InputHandler.cs* script is attached to the active `GameLogic` object in the scene.
5. Your *Game View* is the active window.

### Create a Player Prefab

Now that the player movement has been created, lets save all of that logic.

1. Drag the Ball GameObject into your *Assets > Prefabs* folder.
2. Select *Create Original Prefab* from the resulting prompt.

This will unbind the *Model* and create a brand new prefab with the same Mesh and Material data.

## Step 7: Controlling the Camera

In this project, we want our camera to follow the player.

The mouse axes should be able to pivot the camera around the player.

An easy way to do this, instead of moving the camera directly, is to parent the camera to a pivot.

### Create a Camera Pivot

1. Create an Empty GameObject called `Camera Pivot` and make the `Main Camera` a child.
2. Zero the `Main Camera` position and rotation relative to the pivot.
3. Set the `Main Camera` position.z field in the transform to -5.

![Camera Pivot](Resources/CameraPivot.gif)

Now we can simply rotate the pivot instead of the camera and keep the position of the pivot in the same place as the ball at all times.

### CameraController Script Setup

Now let's write code to control the camera.

1. Create a new script called `CameraController.cs` in your *Assets > Scripts* folder.
2. Attach `CameraController.cs` to the `Camera Pivot` GameObject.
3. Remove unecessary code.
4. Create two private properies to store a reference to the `Player` GameObject and the `Camera` component of the `Main Camera`.
5. Copy and Paste the code to get the `Player`.
6. Get the Camera component in the children of the `Camera Pivot`.

```cs
// CameraController.cs
public class CameraController : MonoBehaviour
{
  private Camera cam;
  private GameObject player;
  private void Awake()
  {
    // Get the Camera in children.
    cam = GetComponentInChildren<Camera>();
    if (cam == null)
    {
      Debug.LogError("No Camera component was found in children.");
      return;
    }

    player = GameObject.FindGameObjectWithTag("Player");
    if (player == null)
    {
      Debug.LogError("No GameObject with 'Player' tag in scene.");
      return;
    }
  }
}
```

### Mouse Look

Now we just need to get the mouse input and rotate the camera pivot accordingly.

In the *CameraController.cs* script:

1. Since we're using the mouse as axis input, lock and hide the cursor in `Start()`.
2. Define a serialized float called `lookSensitivity` to use in the inspector.
3. Define a `LateUpdate()` method that:
   1. Sets the CameraPivot's position to the Player's position.
   2. Makes the camera look at the Player.
4. Define a public `Look()` method that takes in a `Vector2` called `mouseVector`.
5. Calculate the rotation about the x-axis (up and down) and the y-axis (left and right) using the `lookSensitivity`.
6. Set new rotation.

```cs
// CameraController.cs
public class CameraController : MonoBehaviour
{
  [SerializeField] private float lookSensitivity;
  private Camera cam;
  private GameObject player;

  private void Awake() {...}

   private void Start() {
    // Lock and Hide the cursor
    Cursor.lockState = CursorLockMode.Locked;
    Cursor.visible = false;
  }

  private void LateUpdate()
  {
    // Sets camera to player position right before each frame.
    transform.position = player.transform.position;

    // Rotates the camera to look at player.
    cam.transform.LookAt(player.transform.position);
  }
  
  public void Look(Vector2 mouseVector)
  {
    // Calculate new rotations on x and y
    float xRot = transform.eulerAngles.x + mouseVector.y * lookSensitivity;
    float yRot = transform.eulerAngles.y + mouseVector.x * lookSensitivity;

    // Don't rotate on Z axis
    transform.eulerAngles = new Vector3(xRot, yRot, 0);
  }
}
```

In the *InputHandler.cs* script:

1. Create a serialized CameraController property called `camControl`.
2. Define a LateUpdate() method, since we are moving the camera.
3. Get the "Mouse X" and "Mouse Y" axes.
4. Call the `Look()` method with the x and y values.

```cs
// InputHandler.cs
public class InputHandler : MonoBehaviour
{
  [SerializeField] CameraController camControl;
  private Movement playerMovement;

  private void Awake() {...}
  private void FixedUpdate() {...}
  
  void LateUpdate()
  {
    // Gets Mouse Axes
    float mouseX = Input.GetAxis("Mouse X");
    float mouseY = Input.GetAxis("Mouse Y");

    // Sends Mouse Axes to Look() method 
    camControl.Look(new Vector2(mouseX, mouseY));
  }
}
```

### Test MouseLook

In your scene,

1. Attach the `CameraController` instance in the `Camera Pivot` to the `InputHandler` component of the `Game Logic` gameobject.
2. Adjust look sensitivity. ~5-10 is a good amount.

![Wonky Controls](Resources/WonkyControls.gif)

You'll probably notice that this does not feel good. This is because:

1. The camera clips through the floor and other objects.
2. The Mouse Y axis is inverted.

To solve this, we can can limit the X-Rotation between 0 and 90 degrees and invert the Mouse Y axis.

```cs
// InputHandler.cs
private void FixedUpdate()
{
  // Get Movement Input Axes 
  float x = Input.GetAxis("Horizontal");
  float y = -Input.GetAxis("Vertical"); // Inverted

  // Send inputs 
  playerMovement.Move(new Vector3(x, 0, y));
}
```

```cs
// CameraController.cs
public void Look(Vector2 mouseVector)
{
  // Clamp the xRot between MIN and MAX angles
  float xRot = transform.eulerAngles.x + mouseVector.y * lookSensitivity;
  xRot = Mathf.Clamp(xRot, MIN_X_ROT, MAX_X_ROT);

  float yRot = transform.eulerAngles.y + mouseVector.x * lookSensitivity;

  transform.eulerAngles = new Vector3(xRot, yRot, 0);
}
```

## Polish

1. Clipping Pane
2. Uniform Motion Vector
