

## Introduction

Developing **Retro Rampage** has been an exciting challenge, filled with technical hurdles and rewarding breakthroughs. From building player physics and animations to designing a 2D platforming level, we encountered many learning opportunities along the way. In this post, we’ll explore the technical aspects of the project, the difficulties we faced, and how we tackled them.

---

## Implementing Player Physics and Animation

The first step in our development journey was creating the player's core movement mechanics. This involved handling **player physics**, such as movement and jumping, while also integrating animations to make the game feel responsive and lively.

We used **Unity's Rigidbody2D** component to manage physics-based movement. Here’s the script that controls player movement:

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    public float speed;
    public float jumpForce;
    private float moveInput;
    private bool isGrounded;
    public Rigidbody2D RigidBody;
    
    private void Update()
    {
        moveInput = Input.GetAxis("Horizontal");
        RigidBody.velocity = new Vector2(moveInput * speed, RigidBody.velocity.y);

        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            RigidBody.AddForce(new Vector2(RigidBody.velocity.x, jumpForce));
        }
    }
    
    private void OnCollisionEnter2D(Collision2D other)
    {
        if (other.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }
    }
    
    private void OnCollisionExit2D(Collision2D other)
    {
        if (other.gameObject.CompareTag("Ground"))
        {
            isGrounded = false;
        }
    }
}
```
## Challenges Faced
Camera Lagging Behind: Initially, the camera movement felt sluggish. We fine-tuned the followSpeed and used Vector3.Slerp() to create a smooth effect.

Camera Clipping Issues: The camera sometimes clipped through objects, which required us to adjust the offset and camera boundaries.


## Creating the 2D Level with Tilemaps

With movement and physics in place, we shifted our focus to designing the game world. We built the level using Unity's Tilemap system, allowing us to efficiently create a 2D stage.

## Challenges Faced
Tile Alignment Issues: Some tiles didn’t align properly, causing small gaps. We adjusted the tile pivots and grid settings to fix this.

Layering Problems: The player sometimes appeared behind objects. We resolved this by carefully managing sorting layers and Z-ordering.

## Conclusion

Through this journey, we learned a lot about handling player physics, camera movement, and level design. Developing Retro Rampage has been a rewarding experience, and we look forward to expanding the game with new features.



