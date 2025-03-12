# Roll-a-Ball: Our First Game Development Experience

## Introduction 
This is our first game development project , we had a prior experience to editor but still making this simple yet interesting game was challenging. This game helped us with game devlopment fundamentals such as physics , player momvement , collision detection. We will walk you through our experience , technical aspects, and code snippets that brought our first game to life.

## Game Concept

Roll-a-Ball is a classic beginner project in Unity where players control a rolling ball using keyboard input. The goal is to collect objects scattered around the game environment while navigating within a bounded play area. The game ends when all objects are collected.

## Setting Up the Project

The project started with creating a new 3D Unity project and adding basic game objects:

- **Plane**: Served as the ground.
- **Sphere**: Acted as the player-controlled ball.
- **Pickups**: Small cubes scattered around the plane for the player to collect.

After setting up the environment, we assigned materials to objects to improve the visual appeal.

## Player Movement

To enable player movement, we used Rigidbody to the sphere for basic physics interactions. Then, we wrote a simple C# script to control movement using user input:

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    public float speed = 10f;
    private Rigidbody rb;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
    }

    void FixedUpdate()
    {
        float moveHorizontal = Input.GetAxis("Horizontal");
        float moveVertical = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(moveHorizontal, 0.0f, moveVertical);
        rb.AddForce(movement * speed);
    }
}
```
This above piece of code allows player to move the ball around the map using arrow keys. The Fixedupdate method allows a smooth movement throught the game

## Collecting Pickups

When we made the pickup object we had a collider component set to "Is Trigger" to detect player interaction. We made a script which detects when the ball collides with a pickup object , it directly gets removed from the scene.

```csharp
using UnityEngine;

public class Pickup : MonoBehaviour
{
    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            gameObject.SetActive(false); // Hide pickup on collection
        }
    }
}
```
To make sure that the colliding detection works , we flagged the player object as "Player" in Unity's Inspector.


## Camera Follow

This was one of the most important feature of the game , as it makes the game experience really better. We implemented a camera that follows the ball

```csharp
using UnityEngine;

public class CameraController : MonoBehaviour
{
    public Transform player;
    private Vector3 offset;

    void Start()        
    {
        offset = transform.position - player.position;
    }

    void LateUpdate()
    {
        transform.position = player.position + offset;
    }
}
```

The above script ensure the camera follows the ball smoothly , and in the Start() method the offset variable maintans a fixed offset throughout.

## Final Thoughts

Building Roll-a-Ball was an incredible learning experience. It introduced us to Unity's physics engine, user input handling, collision detection, and UI updates. While the game itself is simple, it lays a solid foundation for future projects. 
