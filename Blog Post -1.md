# Roll-a-Ball: Our First Game Development Experience

## Introduction

Starting game development can be an exciting yet challenging journey, and my first project, Roll-a-Ball, was the perfect introduction to Unity and C#. This game, though simple, helped me grasp the fundamentals of game physics, player movement, collision detection, and UI implementation. In this blog post, I'll walk through my experience, key technical aspects, and code snippets that brought my first game to life.

## Game Concept

Roll-a-Ball is a classic beginner project in Unity where players control a rolling ball using keyboard input. The goal is to collect objects scattered around the game environment while navigating within a bounded play area. The game ends when all objects are collected.

## Setting Up the Project

The project started with creating a new 3D Unity project and adding basic game objects:

- **Plane**: Served as the ground.
- **Sphere**: Acted as the player-controlled ball.
- **Pickups**: Small cubes scattered around the plane for the player to collect.

After setting up the environment, I assigned materials to objects to improve the visual appeal.

## Player Movement

To enable player movement, I attached a Rigidbody to the sphere for physics-based interactions. Then, I wrote a simple C# script to control movement using user input:

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
This script allows the player to move the ball using the arrow keys or WASD. The FixedUpdate() method ensures smooth physics-based movement.

## Collecting Pickups

Each pickup object had a Collider component set to "Is Trigger" to detect player interaction. A script was attached to detect when the ball collides with a pickup and removes it from the scene:

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

To ensure detection works, I tagged the player object as "Player" in Unity's Inspector.


## Camera Follow

To enhance the gameplay experience, I implemented a simple camera follow system using a script:

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

This script ensures the camera smoothly follows the ball while maintaining a fixed offset.

## Final Thoughts

Building Roll-a-Ball was an incredible learning experience. It introduced me to Unity's physics engine, user input handling, collision detection, and UI updates. While the game itself is simple, it lays a solid foundation for future projects. My next steps include adding sound effects, visual effects, and refining player controls.
