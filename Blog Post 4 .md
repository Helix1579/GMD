# First Milestone – Run and Gun!

Every game starts with a dream. For me, it was the classic **run and gun** experience: a player, a relentless stream of enemies, and the satisfying *thwack* of a bullet finding its mark. This, I decided, would be my first major milestone.

It sounds straightforward enough, right?

Player shoots, enemy dies, player gets score, new weapon unlocks.

Ah, the sweet naivety of a burgeoning game developer. Let me tell you, getting even this "simple" loop working was a journey fraught with more unexpected twists than a spaghetti dinner.

---

## The Symphony of Destruction: `PlayerShooter`

My journey began with the `PlayerShooter` script. This was the heart of the "gun" in "run and gun." My vision was to have a **base weapon**, and then allow the player to **unlock more powerful or distinct weapons** as they progressed.

```csharp
public class PlayerShooter : Weapon, IShooter
{
    // ...
    public WeaponData baseWeapon;
    public List<WeaponData> unlockedWeapons = new List<WeaponData>();
    private int currentWeaponIndex = 0;
    private float nextFireTime = 0f;
    // ...
    private void Update()
    {
        base.Update(); // Call base Weapon's Update for cooldown
        if (Input.GetButton("Fire1") && Time.time >= nextFireTime)
        {
            Vector2 direction = GetShootDirection();
            TryShoot(direction, "Player");
        }
        if (Input.GetKeyDown(KeyCode.H))
        {
            SwitchWeapon();
        }
    }
    // ...
    public override void TryShoot(Vector2 direction, string shooterTag)
    {
        // ... (instantiate bullet, set direction, damage, destroy) ...
    }
    // ...
}
```
The initial thought process for shooting seemed solid:

Check for input (Fire1)

Check if enough time has passed since the last shot (nextFireTime)

Create a bullet

But oh, the subtle traps!

Hardship 1: The Invisible Bullet Syndrome
My bullets were often spawning and immediately disappearing. This wasn’t a cool feature—it was a bug!

After much head-scratching, the culprit was often one of two things:

The bulletLifeTime was too short, causing bullets to despawn before they even left the firePoint.

The firePoint.position was slightly off, causing the bullet to collide with the player collider itself, triggering an immediate destruction (especially if bullet-on-bullet collision logic was implemented).

The fix involved:

Careful adjustment of bulletLifeTime

Ensuring the firePoint was positioned just outside the player's collider

Simple things, but they eat away at your soul when you can't find them!

## Hardship 2: Directional Woes

I wanted the player to **shoot in the direction they were moving**, or if standing still, in the **direction they were facing**. My `GetShootDirection()` method was meant to handle this:

```csharp
private Vector2 GetShootDirection()
{
    float h = Input.GetAxisRaw("Horizontal");
    float v = Input.GetAxisRaw("Vertical");

    if (h == 0 && v == 0)
        return playerController.IsFacingRight ? Vector2.right : Vector2.left; // <--- The culprit!

    return new Vector2(h, v).normalized;
}
```
This looked great on paper, but in practice, it introduced a new annoyance.

If the player was holding only the vertical input (e.g., shooting straight up or down while standing still), the h == 0 && v == 0 condition was true, and the character would shoot horizontally based on facing direction instead of vertically.

It was a facepalm moment when I realized this.

The Fix
The "fix" was a more robust check that:

Prioritized actual directional input over facing direction

Or, for the purpose of a run-and-gun, simplified it to:

Always shoot horizontally when moving

Use facing direction if not moving horizontally

For my milestone, I decided to keep it simple and focus on horizontal shooting for now, pushing advanced directional aiming to a later stage.

Lesson learned: Prioritization is key when you're overwhelmed!

## The Scorekeeper and the Armory: GameManager

The `GameManager` was the **central nervous system** for my milestone. It needed to **track score**, and crucially, **unlock new weapons**.

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager Instance;
    public int playerScore = 0;
    public WeaponData weaponAt500; // <--- Weapon ScriptableObjects!
    public WeaponData weaponAt1500;
    // ...
    public void AddScore(int amount)
    {
        playerScore += amount;
        Debug.Log("Score: " + playerScore);

        PlayerShooter shooter = FindObjectOfType<PlayerShooter>(); // <--- Another FindObjectOfType...
        if (shooter == null) return;

        if (playerScore >= 500 && weaponAt500 != null)
        {
            shooter.UnlockWeapon(weaponAt500);
            weaponAt500 = null; // Prevent unlocking again
        }
        // ...
    }
    // ...
}
```

The Revelation: ScriptableObjects
Using ScriptableObjects (WeaponData) for weapons was a game-changer! It meant I could define different weapons (damage, fire rate, bullet prefab) as Unity assets, without constantly tweaking values in code. This dramatically improved iteration time and flexibility.

Hardship 3: The Persistent Unlock Bug
My logic for unlocking weapons was:

```csharp
if (playerScore >= 500 && weaponAt500 != null)
{
    shooter.UnlockWeapon(weaponAt500);
    weaponAt500 = null;
}

```
Setting weaponAt500 = null; was crucial to prevent the weapon from unlocking multiple times. However, during testing, I'd often reset the game and find that the weapons wouldn’t unlock again!

Why? The GameManager instance was DontDestroyOnLoad, so weaponAt500 stayed null across game restarts — a classic state-not-resetting bug.

The Fix
Introduce a ResetGame() method that restores initial state, including re-assigning the default WeaponData references:


```csharp
public void ResetGame()
{
    playerScore = 0;
    Debug.Log("Reset game");

    weaponAt500 = defaultWeaponAt500; // <--- Resetting the references!
    weaponAt1500 = defaultWeaponAt1500;

    Debug.Log("GameManager reset.");
}
```
This required storing the original data in defaultWeaponAt500, etc. Small detail, but these state management quirks can be painfully hard to track in larger projects.

Hardship 4: FindObjectOfType – The Easy, But Dangerous Friend
You’ll notice:

PlayerShooter shooter = FindObjectOfType<PlayerShooter>();
While this is convenient for prototyping, FindObjectOfType is notoriously inefficient. In a larger game, it becomes a performance bottleneck.

The hardship here isn’t just technical — it’s philosophical:
Do you resist the urge to optimize prematurely, or do you build with scalability in mind from day one?

For the milestone, functionality trumped performance. But that nagging feeling that it’s "not quite right" remains.

The Plan Forward
Eventually, the PlayerShooter reference should be:

Assigned in the Inspector, or

Injected via a robust dependency injection system

Once the project matures, refactoring this will be essential.

## The UI: Showing Off My Hard-Won Progress

Finally, the **UI elements** were crucial to making the milestone feel **complete** — a `ScoreUI` to display the player's score and a `WeaponUI` to show the currently equipped weapon.

```csharp
public class ScoreUI : MonoBehaviour
{
    private TextMeshProUGUI scoreText;

    private void Update()
    {
        if (GameManager.Instance != null)
        {
            scoreText.text = $"Score: {GameManager.Instance.playerScore}";
        }
    }
}
```
Hardship 5: The Null Reference Epidemic
This line:

```csharp
scoreText.text = $"Score: {GameManager.Instance.playerScore}";
```
...seemed innocent enough, but it led to countless null reference exceptions early on.

Why?
GameManager.Instance might not have been initialized before ScoreUI.Update() was called.

scoreText might not have been assigned at all (e.g., via Awake() or Start()).

These issues caused frustrating and confusing runtime errors, especially when trying to debug multiple systems at once.

The Fix
Use the classic null check:
```csharp
if (GameManager.Instance != null)
```
Properly assign scoreText in Awake():
```csharp
private void Awake()
{
    scoreText = GetComponent<TextMeshProUGUI>();
}
```
Lesson learned: UI code is deceptively simple. But even a single missing reference can break the whole visual feedback loop — and nothing kills a milestone demo like a broken score display.

