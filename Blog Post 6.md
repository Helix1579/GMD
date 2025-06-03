# Milestone 3:Crafting Multi-Staged Boss Battles

 After building the foundational run-and-gun mechanics (Milestone 1) and populating the level with dynamic AI enemies (Milestone 2), it was time for the  multi-staged boss battles. This milestone was about delivering a truly amazing boss battle where the environment would change based on the health of boss. It was a leap in complexity, introducing environmental changes, evolving mechanics, and a whole new level of challenge. We tried our best to keep up with the expectation , but none the less we made a multiphased boss battle.

## The Architect of Annihilation: My Boss AI

The core of this milestone was the `BossAI` script. The vision for the boss was something that transformed as its health reduced, pushing the player to adapt. This meant differnt phases, each with its own set of rules, attack patterns, and even visual changes.

```csharp
public class BossAI : MonoBehaviour
{
    public enum BossPhase { Phase1, Phase2, Phase3, Dead }
    private BossPhase currentPhase = BossPhase.Phase1;

    [Header("Combat Settings")]
    public Transform player;
    public float[] phaseFireRates = { 2f, 1.2f, 0.6f }; // Faster attacks in later phases
    public float[] phaseAttackRange = { 10f, 8f, 5f }; // Closer range in later phases
    
    [Header("Bullet Prefabs Per Phase")]
    public GameObject[] bulletPrefabs; // Different bullet types for different phases
    public Sprite[] turretSprites; // Visual changes for turrets

    [Header("Turrets")]
    public TurretController[] turrets; // Boss relies on external turrets
    
    [Header("Turrets Weapon Sound")]
    public AudioClip[] bulletSounds; // Dynamic sound changes

    private int phase2Threshold; // Health % for phase 2
    private int phase3Threshold; // Health % for phase 3
    
    private EnemyHealth bossHealth; // The boss's health component
    private bool isInitialized = false;
    [SerializeField] private PlayerWinMenu playerWinMenu; // To show win screen
    // ... Awake and InitializeBoss methods ...
    void Update()
    {
        if (!isInitialized || currentPhase == BossPhase.Dead || player == null) return;

        int currentHp = Mathf.Clamp(bossHealth.CurrentHealth, 0, bossHealth.maxHealth); 

        // Detect health changes to trigger phase transitions
        if (currentHp != bossHp) // bossHp tracks health at last check
        {
            HandlePhaseTransition(currentHp);
            bossHp = currentHp;
        }
    }
    // ... HandlePhaseTransition, EnterPhase, SetTurrets methods ...
}
```
The approach involved having arrays for phaseFireRates, phaseAttackRange, bulletPrefabs, turretSprites, and bulletSounds. The index of these arrays would correspond to the BossPhase enum, allowing for a clean and scalable way to define phase-specific attributes.

### Hardship 1
The most critical part of a multi-stage boss is the HandlePhaseTransition logic. The initial attempts were riddled with bugs: phases not triggering, triggering multiple times, or even worse, skipping phases entirely.

```csharp
void HandlePhaseTransition(int hp)
{
    if (hp <= 0)
    {
        EnterPhase(BossPhase.Dead);
        return;
    }

    // Always check for the *most advanced* phase first
    if (hp <= phase3Threshold && currentPhase != BossPhase.Phase3)
    {
        EnterPhase(BossPhase.Phase3);
    }
    else if (hp <= phase2Threshold && currentPhase == BossPhase.Phase1) // <--- Key change here
    {
        EnterPhase(BossPhase.Phase2);
    }
    // No need to go back to Phase1 once you're in Phase2 or 3
}
```

The breakthrough came with understanding that we needed to check for the most advanced phase first. If the boss's health dropped below the Phase 3 threshold, it should immediately jump to Phase 3, regardless of whether it briefly passed through the Phase 2 threshold. Also, explicitly checking currentPhase != BossPhase.Phase3 or currentPhase == BossPhase.Phase1 prevented re-entering a phase and ensured a linear progression. This simple ordering and conditional check was life saver, previously had issues with the boss stuck between phases or not entering the correct one.

### Hardship 2
Instead of the boss itself being a single entity,we decided to give it TurretController children. These turrets would perform the actual shooting. This design offered a lot of flexibility: we could have multiple turrets, place them strategically around the boss, and they could independently handle their own targeting and firing. The BossAI would then tell its turrets what to shoot and how fast.

```chsarp
void SetTurrets(float rate, GameObject prefab, float range, Sprite sprite, AudioClip sound)
{
    foreach (var turret in turrets)
    {
        turret.UpdateFireRate(rate);
        turret.UpdateBulletPrefab(prefab);
        turret.UpdateAttackRange(range);
        turret.UpdateTurretSprite(sprite);
        turret.UpdateTurretSound(sound);
    }
}
```
The major challenge was ensuring the BossAI correctly initialized and updated these turrets. Early on, some turrets wouldn't fire, or they'd retain old settings.

-Improper initialization: Not all turrets were being correctly added to the turrets array in the Inspector.
-Timing issues: InitializeBoss() being called before GameManager.Instance or other dependencies were ready, leading to null bossHealth or incorrect threshold values. Adding an isInitialized flag and ensuring InitializeBoss() was called at the correct time.
Missing references: Forgetting to assign bulletPrefabs, turretSprites, or bulletSounds for all phases in the Inspector. Unity's editor can be forgiving, but runtime errors are annoying. 

## Final Recap
### Milestone 1: The Run and Gun Foundation
We began with our games foundation : player movement and shooting. This involved Rigidbody2D for smooth motion, getting the isGrounded checks just right (after much trial and error!), and implementing the PlayerShooter to fire bullets. The biggest headaches here were often the simplest: a flipped isGrounded boolean, or a firePoint misaligned by a pixel. But getting the core loop of shooting and hitting targets was incredibly satisfying.We learned the hard way that tiny code mistakes can cause huge gameplay frustrations.

### Milestone 2: Intelligent Adversaries & Scalable Challenge
Next,was Enemy AI and Difficulty Levels. We implemented different AI behaviors like Patrol and Ranged Chase, making enemies react dynamically to the player's presence and actions (like shooting at them!). Integrating NavMeshAgent for enemy pathfinding in a 2D environment was a unique challenge. Moreover, introduced DifficultySetting ScriptableObjects and a GameManager to allow players to choose their challenge, scaling enemy health, damage, and spawn rates. This milestone taught the power of modular design through interfaces (IEnemyAI) and the flexibility of data-driven game parameters. The feeling of seeing enemies intelligently chase and fight back was a huge leap forward for the game's immersiveness.

### Milestone 3: The Boss Battle
The Boss Battles. This was about pushing the complexity of game design. By breaking the boss down into multi-staged phases, each with unique fireRates, bulletPrefabs, and even turretSprites, we created a dynamic and evolving challenge. The core hardship was robustly handling phase transitions based on health thresholds, ensuring a smooth and intentional progression through the fight. The boss's attack logic into TurretController components provided immense flexibility and allowed for complex attack patterns. While not explicitly in the code, the underlying challenge of shifting environments was not completely fulfilled , but we gave our best in making it evolving . 
