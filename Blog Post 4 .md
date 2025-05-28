# First Milestone – Run and Gun!

It was the classic **run and gun** experience: a player, a relentless stream of enemies, and the satisfying sound of a bullet finding its mark. This, We decided, would be our first major milestone.
Player shoots, enemy dies, player gets score, new weapon unlocks.
---

## `PlayerShooter`

The journey began with the `PlayerShooter` script. This was very important in  "run and gun." The vision was to have a **base weapon**, and then allow the player to **unlock more powerful or distinct weapons** as they progressed.
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
The initial thought process for shooting seemed solid.Check for input (Fire1).Check if enough time has passed since the last shot (nextFireTime).Create a bullet.The bullets were often spawning and immediately disappearing. This wasn’t a cool feature—it was a bug!The bulletLifeTime was too short, causing bullets to despawn before they even left the firePoint.The firePoint.position was slightly off, causing the bullet to collide with the player collider itself, triggering an immediate destruction.
The fix involved: Careful adjustment of bulletLifeTime.Ensuring the firePoint was positioned just outside the player's collider

We wanted the player to **shoot in the direction they were moving**, or if standing still, in the **direction they were facing**. The `GetShootDirection()` method was meant to handle this:

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

If the player was holding only the vertical input (e.g., shooting straight up or down while standing still), the h == 0 && v == 0 condition was true, and the character would shoot horizontally based on facing direction instead of vertically. Prioritized actual directional input over facing direction Or, for the purpose of a run-and-gun, simplified it to , Always shoot horizontally when moving Use facing direction if not moving horizontally

## The Scorekeeper and the Armory: GameManager

The `GameManager` was the **central nervous system** for the milestone. It needed to **track score**, and more importantly, **unlock new weapons**.

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
Using ScriptableObjects (WeaponData) for weapons was a game-changer! It meant that we could define different weapons (damage, fire rate, bullet prefab) as Unity assets, without constantly tweaking values in code. This improved iteration time and flexibility.

The logic for unlocking weapons :

```csharp
if (playerScore >= 500 && weaponAt500 != null)
{
    shooter.UnlockWeapon(weaponAt500);
    weaponAt500 = null;
}

```
Setting weaponAt500 = null; was crucial to prevent the weapon from unlocking multiple times. However, during testing,  often reset the game and find that the weapons wouldn’t unlock again!

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
This required storing the original data in defaultWeaponAt500, etc. Small detail, but these state management  can be hard to track in larger projects.


## The UI

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

This line:

```csharp
scoreText.text = $"Score: {GameManager.Instance.playerScore}";
```

GameManager.Instance might not have been initialized before ScoreUI.Update() was called.

scoreText might not have been assigned at all (e.g., via Awake() or Start()).

These issues caused frustrating and confusing runtime errors, especially when trying to debug multiple systems at once.

Properly assign scoreText in Awake():
```csharp
private void Awake()
{
    scoreText = GetComponent<TextMeshProUGUI>();
}
```
 UI code is simple. But even a single missing reference can break the whole visual feedback loop.

