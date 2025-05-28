## The Player Controller:

The journey began, as many do, with the **Player Controller**. wanted a character that could move, jump, and look left or right. Simple enough.
The initial thought process was straightforward: get horizontal input, apply it to the Rigidbody, and make the character jump when the "Jump" button is pressed. Here's a snippet of the code :

```csharp
public class PlayerController : MonoBehaviour
{
    // ... other variables ...
    public float speed;
    public float jumpForce;
    private float moveInput;
    private bool isGrounded;
    public Rigidbody2D RigidBody;
    [SerializeField] private Animator animator;
    
    // ... Awake and other methods ...

    private void Update()
    {
        moveInput = Input.GetAxis("Horizontal");
        RigidBody.linearVelocity = new Vector2(moveInput * speed, RigidBody.linearVelocity.y);
        
        // ... flip logic ...
        
        animator.SetBool("isRunning", moveInput != 0 );

        if (Input.GetButtonDown("Jump") && isGrounded == false) 
        {
            RigidBody.AddForce(new Vector2(RigidBody.linearVelocity.x, jumpForce));
            animator.SetBool("isJumping", true);
        }
    }
    
}
```
 The First Hurdle? Jumping

The initial `if (Input.GetButtonDown("Jump") && isGrounded == false)` condition for jumping felt right. The character should only jump if they're not grounded.
Except, it led to frustrating bugs. Sometimes the jump wouldn't register at all. Other times, the character would perform a tiny leap. 

After hours of debugging, print statements, realized that `isGrounded` logic was flipped in the `OnCollisionEnter2D` and `OnCollisionExit2D` methods.  setting `isGrounded = false` when entering a collision with the ground, and `isGrounded = true` when exiting it! 

A small victory, but a victory nonetheless.

---

# The Flip

Then came the character flip. When the character moved left, wanted them to face left. When they moved right, they should face right. 

`Flip()` method looked like this:

```csharp
private void Flip()
{
    m_FacingRight = !m_FacingRight;
    transform.Rotate(0f, 180f, 0f);

    // Optional: Flip weapon manually if needed
    Transform weapon = transform.Find("Weapon");
    if (weapon != null)
    {
        Vector3 scale = weapon.localScale;
        scale.x *= -1;
        weapon.localScale = scale;
    }
}
```

# The Health System

Once movement and basic interaction were sorted then  turned to core game mechanics: **health**. wanted a flexible health system that could be used by both the player and potentially other entities (enemies, destructible objects). 

## Here's the setup:

```csharp
public interface IHealth
{
    void TakeDamage(int amount);
    void Heal(int amount);
}

public abstract class BaseHealth : MonoBehaviour, IHealth
{
    public int maxHealth = 5;
    protected int currentHealth;

    public System.Action<int> OnHealthChanged; // <--- The event!

    // ... Start, TakeDamage, Heal methods ...

    protected abstract void Die(); // <--- Must be implemented by derived classes
    
    // ... ResetHealth ...
}

public class PlayerHealth : BaseHealth
{
    public HealthBar healthBar;

    protected override void Start()
    {
        base.Start();

        if (healthBar != null)
        {
            healthBar.SetMaxHealth(maxHealth);
            healthBar.SetHealth(currentHealth);

            OnHealthChanged += healthBar.SetHealth; // <--- Subscribing to the event
        }
    }
    
    protected override void Die()
    {
        Debug.Log("Player died!");
        gameObject.SetActive(false); // <--- Player just disappears for now
    }
}
```

The initial concept was solid. IHealth defines the contract, BaseHealth provides common functionality, and PlayerHealth implements the player-specific Die() method and handles UI updates.

The challenge wasn't in the structure itself, but in the implementation details and how to make it flexible. We wanted a health bar to update automatically whenever health changed.Instead of PlayerHealth directly calling a HealthBar.UpdateHealth() method, BaseHealth could simply announce that its health had changed. Any subscriber, like the HealthBar, could then react.The struggle with when to Invoke the event, when to Subscribe, and when to Unsubscribe.
It was a shift from direct method calls to a more decoupled system. The benefit is immense: BaseHealth doesn't need to know anything about HealthBar, making the system far more modular. 

The Die() method being abstract was also a point of conflict initially. It meant every subclass had to implement its own death logic. While this enforced good design.
It forced to think about the specific death behavior for each entity. For the player, it's gameObject.SetActive(false); (for now, a simple disable). This seemingly small design choice had significant implications for how we structured our game objects.




