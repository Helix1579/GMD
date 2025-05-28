# Milestone 2: Unleashing the Beasts – My Dive into Enemy AI and Difficulty

---

With the core "run and gun" loop established, my next big challenge was bringing it to life with **worthy adversaries**. A game without enemies is just a **glorified walking simulator**, and my vision for dynamic, engaging combat demanded more than static targets.

Thus began **Milestone 2: Enemy AI and Difficulty Levels** — a journey into the **fascinating**, yet often **frustrating**, world of artificial intelligence and balancing gameplay challenge.

---

## The Minds of Mayhem: Designing Enemy AI

I envisioned a few distinct enemy behaviors:

- A simple **Patrol** type that just moves around.
- A more aggressive **Ranged Chase** enemy that actively hunts the player.
- A **Turret** type (planned later) that would act as a static but dangerous threat.

To keep these behaviors flexible and maintainable, I introduced an `IEnemyAI` interface — the **contract** that all enemy AIs would follow.

```csharp
public interface IEnemyAI
{
    void SetTarget(Transform target);
    void Init(EnemyConfig config);
    void SetFirePoint(Transform firePoint); // Essential for flexible enemy design
}
Why an Interface?
This interface was crucial to decoupling enemy logic from specific behavior implementations:

I could swap out AI types on different enemy prefabs.

Core enemy logic stayed untouched — no messy conditionals for behavior types.

It encouraged clean, modular design and made future extensions (like adding a turret AI) far simpler.

```
This interface was crucial for allowing me to swap out AI types on different enemy prefabs without changing core enemy logic.
# Ranged Chase AI: The Hunter

---

For Milestone 2, the **RangedChaseAI** was the most complex beast I had to tame. It needed to:

- Detect the player,
- Chase them until within firing range,
- Stop, face the player, and shoot,
- React to player movement,
- Flip its sprite to face the target.

Sounds simple on paper — but the implementation brought with it a swarm of edge cases and painful bugs.

---

## Core Logic: Chase, Stop, and Shoot

Here's the core structure of my `RangedChaseAI` implementation:

```csharp
public class RangedChaseAI : MonoBehaviour, IEnemyAI
{
    private NavMeshAgent agent;         // For pathfinding
    private Transform player;           // Target to chase
    private EnemyConfig config;         // Behavior settings
    private Transform firePoint;        // Where bullets are spawned
    private bool isProvoked = false;    // Whether the enemy is in combat mode

    void Update()
    {
        if (player == null || agent == null || enemy == null) return;

        float distance = Vector2.Distance(transform.position, player.position);

        // Provocation logic
        if (distance <= enemy.attackRange || isProvokedByPlayerShooting)
        {
            isProvoked = true;
        }
        if (distance > enemy.attackRange + 1f && !isProvokedByPlayerShooting)
        {
            isProvoked = false;
            agent.ResetPath();
            return;
        }

        if (!isProvoked) return;

        // Chase or shoot
        if (distance > config.fireDistance)
        {
            agent.SetDestination(player.position);
        }
        else
        {
            agent.ResetPath();
            FlipTowardsPlayer();
            // Shooting cooldown logic here
            ShootAtPlayer();
        }
    }
}
```
Hardship 1: The Stationary Shooter
One of the first bugs? The enemy would just... stand there, doing nothing — even when the player was clearly within range. After some grueling debugging, the issues became clear:

NavMeshAgent not initialized properly, or

Getting stuck on colliders or terrain geometry.

Fixes that helped:
Set agent.updateRotation = false and agent.updateUpAxis = false to avoid Unity’s 3D nav logic interfering with 2D sprites.

Make sure the NavMesh was baked correctly and did not intersect with enemy colliders.

Use agent.ResetPath() before shooting to prevent “nudging” behavior that interrupted aim.

Hardship 2: The FirePoint Flip Flop
Getting the enemy to face the player was straightforward. But then came a ghostly bug: bullets started firing from behind the enemy or out of thin air.

Why? Because when the sprite flipped, the firePoint (bullet spawn location) didn’t flip with it!

The Fix: Flip the FirePoint manually
```csharp
// Inside FlipTowardsPlayer()
// Flip enemy's local X scale
Vector3 localScale = transform.localScale;
localScale.x *= -1;
transform.localScale = localScale;

// Flip FirePoint's local X position
Vector3 fpLocal = firePoint.localPosition;
fpLocal.x = -fpLocal.x; // <--- The magical line!
firePoint.localPosition = fpLocal;
```
# Patrol AI: The Simple Wanderer

The `PatrolAI` was simpler, focusing on moving between two designated points and only engaging if provoked.

### C#

```csharp
public class PatrolAI : MonoBehaviour, IEnemyAI
{
    // ...
    private NavMeshAgent agent;
    private Transform player;
    private EnemyConfig config;
    private bool goingToA = true;
    private bool isProvoked = false;
    // ...
    void Update()
    {
        // ... patrol logic ...
        if (player == null) return;
        float dist = Vector2.Distance(transform.position, player.position);
        if (dist <= enemy.attackRange)
        {
            isProvoked = true;
        }
        if (isProvoked)
        {
            // ... shooting logic ...
            ShootAtPlayer();
        }
    }
    // ...
}
```
Hardship 3: The Passive Aggressor
My patrol enemies were a bit too passive. They'd patrol, and even if the player was right in front of them, they'd just keep walking. The trigger for isProvoked was only based on attackRange.

I quickly realized that enemies also needed to react if the player shot at them. This meant the enemy's TakeDamage method (from the health system) needed to set isProvoked = true on the AI component.

This seemingly small addition drastically improved the enemies' reactive behavior and made the game feel more dynamic. It forced me to think about how different systems (AI, Health, Combat) needed to communicate with each other.

# The Balancing Act: Difficulty Levels

Introducing difficulty was paramount to making the game replayable and accessible. I decided on three levels: **Noob**, **Pro**, and **Rampage**, each with configurable settings for enemy health, damage, and spawn limits.

```csharp
public enum DifficultyLevel { Noob, Pro, Rampage }

[System.Serializable]
public class DifficultySetting
{
    public DifficultyLevel level;
    public int maxEnemiesToSpawn;
    public int maxEnemiesAliveAtOnce;
    public int enemyHealth;
    public int enemyDamage;
}

public class GameManager : MonoBehaviour
{
    // ...
    public DifficultySetting currentDifficulty;
    public DifficultySetting noobSettings;
    public DifficultySetting proSettings;
    public DifficultySetting rampageSettings;

    public void SetDifficulty(string difficulty)
    {
        switch (difficulty)
        {
            // ... assign currentDifficulty based on string ...
        }
        Debug.Log($"Difficulty selected: {currentDifficulty.level}");
    }
    // ...
}

```

Hardship 4: The Static Difficulty Trap
My initial approach to difficulty settings was to just have a DifficultySetting class and set values directly. However, if I changed those values in the Inspector while playing, they wouldn't persist, and if I changed them in code, it became hard to balance.

The solution was using ScriptableObjects for each DifficultySetting. This allowed me to create distinct "Noob Settings", "Pro Settings", and "Rampage Settings" assets in Unity, which could be easily configured and swapped out at runtime via the GameManager.

This decoupled the settings from the GameManager script itself, making them data-driven and far more flexible.

Hardship 5: Orchestrating the Main Menu
Integrating the difficulty selection into the MainMenuManager was a bit of a dance. The game needed to pause (Time.timeScale = 0f), present difficulty options, and then, once selected, resume gameplay (Time.timeScale = 1f) and hide the menu.


```csharp

public class MainMenuManager : MonoBehaviour
{
    // ...
    private void Start()
    {
        Time.timeScale = 0f; // Pause game on start
        difficultyPanel.SetActive(false);
        gameplayGroup.SetActive(false);
        // ... music logic ...
    }

    public void OnSelectDifficulty(string difficulty)
    {
        GameManager.Instance?.SetDifficulty(difficulty);
        Time.timeScale = 1f; // Resume game
        gameObject.SetActive(false); // Hide menu canvas
        gameplayGroup.SetActive(true); // Show gameplay UI
        // ... music logic ...
    }
    // ...
}
```
The immediate headaches involved null references if GameManager.Instance wasn't ready, or UI elements not showing/hiding correctly. Debugging UI state changes and Time.timeScale interactions was a test of patience—especially when input seemed to mysteriously stop working (because Time.timeScale was still 0!).
