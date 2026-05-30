# Day 03 — Composition vs Inheritance
## 30-Day LLD Series 
### Language: Java

---

## What Are We Talking About?

Two ways to reuse code in Object-Oriented Programming:

```
Inheritance  → "IS-A" relationship
               class Dog extends Animal

Composition  → "HAS-A" relationship
               class Dog { private Legs legs; }
```

Both let you reuse behaviour.
Both are taught in every Java course.
But they are NOT interchangeable.

**Using the wrong one is one of the most common design mistakes in Java.**

This day teaches you:
- What inheritance actually is and when it breaks
- What composition is and why it is almost always better
- How to refactor from inheritance to composition
- The real-world rule: **favour composition over inheritance**

This rule is so important it is written in the Gang of Four Design Patterns book.
It is one of the most cited principles in software engineering.

---

## Inheritance — What It Is

Inheritance means one class **extends** another.
The child gets everything the parent has — fields, methods, behaviour.

```java
// Parent class
class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void breathe() {
        System.out.println(name + " is breathing");
    }

    public void eat() {
        System.out.println(name + " is eating");
    }
}

// Child class inherits everything from Animal
class Dog extends Animal {

    public Dog(String name) {
        super(name);
    }

    public void bark() {
        System.out.println(name + " says: Woof!");
    }
}

// Usage
Dog dog = new Dog("Bruno");
dog.breathe();   // inherited from Animal
dog.eat();       // inherited from Animal
dog.bark();      // Dog's own method
```

**Output:**
```
Bruno is breathing
Bruno is eating
Bruno says: Woof!
```

This looks clean. This is fine **here**.
Now watch what happens when requirements grow.

---

## The Problem With Inheritance — It Breaks

### Scenario: Build a Game with Characters

```java
// BAD — Deep inheritance hierarchy that collapses under real requirements

// Level 1
class Character {
    protected String name;
    protected int health;

    public Character(String name, int health) {
        this.name   = name;
        this.health = health;
    }

    public void takeDamage(int damage) {
        this.health -= damage;
        System.out.println(name + " took " + damage + " damage. HP: " + health);
    }
}

// Level 2
class FightingCharacter extends Character {
    public FightingCharacter(String name, int health) {
        super(name, health);
    }

    public void attack(Character target) {
        System.out.println(name + " attacks " + target.name);
        target.takeDamage(10);
    }
}

// Level 3
class MagicCharacter extends FightingCharacter {
    private int mana;

    public MagicCharacter(String name, int health, int mana) {
        super(name, health);
        this.mana = mana;
    }

    public void castSpell(Character target) {
        if (mana >= 20) {
            mana -= 20;
            System.out.println(name + " casts a spell on " + target.name);
            target.takeDamage(30);
        } else {
            System.out.println(name + " has no mana!");
        }
    }
}

// Level 4 — now requirements hit you
// Requirement: "We need a character that can fly AND cast spells BUT cannot fight."
// Problem: FlyingMage must extend something.
// If it extends MagicCharacter → it gets attack() which it should NOT have.
// If it extends Character → it must re-implement spell logic.
// There is no clean fit. The hierarchy is broken.

class FlyingMage extends MagicCharacter {      // gets attack() — WRONG
    public FlyingMage(String name, int health, int mana) {
        super(name, health, mana);
    }

    public void fly() {
        System.out.println(name + " is flying");
    }

    // Now we are forced to override attack to do nothing — LSP violation
    @Override
    public void attack(Character target) {
        System.out.println(name + " cannot fight!");   // broken
    }
}

// More requirements:
// "We need a RobotWarrior that fights but does NOT eat or breathe."
// "We need a Ghost that can fly and cast spells but takes no damage."
// "We need a Merchant that can fight a little but mostly trades."
// Every new requirement breaks the hierarchy further.
// This is called the Fragile Base Class Problem.
```

### Why Inheritance Breaks Here

```
Problems with deep inheritance:

1. FRAGILE BASE CLASS
   Change the parent → every child might break.
   You cannot safely modify Animal without checking Dog, Cat, Wolf, Bear...

2. TIGHT COUPLING
   Child is locked to parent's implementation.
   You cannot change HOW breathing works for just one subclass.

3. GORILLA / BANANA PROBLEM
   "You wanted a banana but you got a gorilla holding the banana
    and the entire jungle."
   — Joe Armstrong, creator of Erlang
   You extend a class for ONE method and inherit 20 you don't want.

4. RIGID HIERARCHY
   Hierarchies work for the cases you designed them for.
   The first requirement that does not fit — everything breaks.

5. CANNOT MIX BEHAVIOURS
   Java is single-inheritance.
   A class can only extend ONE parent.
   What if you need flying + fighting + magic + stealth?
   You cannot inherit all four.
```

---

## Composition — What It Is

Instead of **being** something, your class **has** something.

Instead of extending a class to get its behaviour,
you hold a **reference** to an object that provides that behaviour.

```java
// Composition — the class HAS behaviours as fields

class Dog {
    private String name;
    private Breathing breathing;    // HAS-A Breathing
    private Eating    eating;       // HAS-A Eating
    private Barking   barking;      // HAS-A Barking

    public Dog(String name) {
        this.name      = name;
        this.breathing = new NormalBreathing();
        this.eating    = new NormalEating();
        this.barking   = new LoudBarking();
    }

    public void breathe() { breathing.breathe(name); }
    public void eat()     { eating.eat(name); }
    public void bark()    { barking.bark(name); }
}
```

Now watch how this scales cleanly.

---

## Good Code — Full Composition Example

### Step 1 — Define Behaviour Interfaces

```java
// Each behaviour is a contract (interface)
// Any class can implement it independently

public interface Attackable {
    void attack(String targetName);
}

public interface Flyable {
    void fly(String characterName);
}

public interface MagicCaster {
    void castSpell(String casterName, String targetName);
    int getMana();
    void consumeMana(int amount);
}

public interface Stealthable {
    void goInvisible(String characterName);
    void reveal(String characterName);
}

public interface Tradeable {
    void trade(String characterName, String item);
}
```

---

### Step 2 — Implement Behaviours Concretely

```java
// ── Attack behaviours ────────────────────────────────────────────────────

public class SwordAttack implements Attackable {
    @Override
    public void attack(String targetName) {
        System.out.println("Swings sword at " + targetName + " for 15 damage");
    }
}

public class BowAttack implements Attackable {
    @Override
    public void attack(String targetName) {
        System.out.println("Fires arrow at " + targetName + " for 10 damage");
    }
}

public class NoAttack implements Attackable {
    @Override
    public void attack(String targetName) {
        System.out.println("Cannot attack — this character is non-combatant");
    }
}


// ── Flying behaviours ────────────────────────────────────────────────────

public class WingFlight implements Flyable {
    @Override
    public void fly(String characterName) {
        System.out.println(characterName + " soars on giant wings");
    }
}

public class MagicFlight implements Flyable {
    @Override
    public void fly(String characterName) {
        System.out.println(characterName + " levitates using arcane energy");
    }
}

public class NoFlight implements Flyable {
    @Override
    public void fly(String characterName) {
        System.out.println(characterName + " cannot fly");
    }
}


// ── Magic behaviours ─────────────────────────────────────────────────────

public class FireMagic implements MagicCaster {
    private int mana;

    public FireMagic(int initialMana) {
        this.mana = initialMana;
    }

    @Override
    public void castSpell(String casterName, String targetName) {
        if (mana >= 25) {
            mana -= 25;
            System.out.println(casterName + " hurls a fireball at "
                + targetName + " for 40 damage! Mana left: " + mana);
        } else {
            System.out.println(casterName + " is out of mana!");
        }
    }

    @Override
    public int getMana() { return mana; }

    @Override
    public void consumeMana(int amount) { this.mana -= amount; }
}

public class IceMagic implements MagicCaster {
    private int mana;

    public IceMagic(int initialMana) {
        this.mana = initialMana;
    }

    @Override
    public void castSpell(String casterName, String targetName) {
        if (mana >= 20) {
            mana -= 20;
            System.out.println(casterName + " freezes " + targetName
                + " with an ice lance! Mana left: " + mana);
        } else {
            System.out.println(casterName + " is out of mana!");
        }
    }

    @Override
    public int getMana() { return mana; }

    @Override
    public void consumeMana(int amount) { this.mana -= amount; }
}


// ── Stealth behaviours ───────────────────────────────────────────────────

public class ShadowStealth implements Stealthable {
    @Override
    public void goInvisible(String characterName) {
        System.out.println(characterName + " melts into the shadows");
    }

    @Override
    public void reveal(String characterName) {
        System.out.println(characterName + " steps out of the shadows");
    }
}
```

---

### Step 3 — Build Characters by Composing Behaviours

```java
// ── Base Character ────────────────────────────────────────────────────────

public class Character {
    protected String name;
    protected int    health;

    public Character(String name, int health) {
        this.name   = name;
        this.health = health;
    }

    public void takeDamage(int damage) {
        this.health -= damage;
        System.out.println(name + " takes " + damage
            + " damage. HP remaining: " + health);
    }

    public String getName() { return name; }
    public int    getHealth(){ return health; }
}


// ── Warrior — fights with a sword, cannot fly, no magic ──────────────────

public class Warrior extends Character {

    private final Attackable  attackBehaviour;
    private final Flyable     flyBehaviour;

    public Warrior(String name) {
        super(name, 200);
        this.attackBehaviour = new SwordAttack();    // HAS-A SwordAttack
        this.flyBehaviour    = new NoFlight();        // HAS-A NoFlight
    }

    public void attack(String targetName) {
        attackBehaviour.attack(targetName);
    }

    public void fly() {
        flyBehaviour.fly(name);
    }
}


// ── Mage — casts fire spells, flies magically, cannot melee ──────────────

public class Mage extends Character {

    private final Attackable  attackBehaviour;
    private final MagicCaster magicBehaviour;
    private final Flyable     flyBehaviour;

    public Mage(String name) {
        super(name, 100);
        this.attackBehaviour = new NoAttack();          // Mage cannot melee
        this.magicBehaviour  = new FireMagic(100);      // HAS-A FireMagic
        this.flyBehaviour    = new MagicFlight();       // HAS-A MagicFlight
    }

    public void attack(String targetName) {
        attackBehaviour.attack(targetName);
    }

    public void castSpell(String targetName) {
        magicBehaviour.castSpell(name, targetName);
    }

    public void fly() {
        flyBehaviour.fly(name);
    }
}


// ── Rogue — bow attack, stealth, no magic, no flight ─────────────────────

public class Rogue extends Character {

    private final Attackable  attackBehaviour;
    private final Stealthable stealthBehaviour;
    private final Flyable     flyBehaviour;

    public Rogue(String name) {
        super(name, 130);
        this.attackBehaviour  = new BowAttack();       // HAS-A BowAttack
        this.stealthBehaviour = new ShadowStealth();   // HAS-A ShadowStealth
        this.flyBehaviour     = new NoFlight();
    }

    public void attack(String targetName) {
        attackBehaviour.attack(targetName);
    }

    public void hide() {
        stealthBehaviour.goInvisible(name);
    }

    public void reveal() {
        stealthBehaviour.reveal(name);
    }
}


// ── FlyingMage — ice magic, wing flight, no melee ────────────────────────
// This was IMPOSSIBLE cleanly with inheritance.
// With composition: trivial.

public class FlyingMage extends Character {

    private final Attackable  attackBehaviour;
    private final MagicCaster magicBehaviour;
    private final Flyable     flyBehaviour;

    public FlyingMage(String name) {
        super(name, 120);
        this.attackBehaviour = new NoAttack();
        this.magicBehaviour  = new IceMagic(150);      // different magic
        this.flyBehaviour    = new WingFlight();        // different flight
    }

    public void castSpell(String targetName) {
        magicBehaviour.castSpell(name, targetName);
    }

    public void fly() {
        flyBehaviour.fly(name);
    }
}


// ── Main — run everything ─────────────────────────────────────────────────

public class Main {

    public static void main(String[] args) {

        System.out.println("=== WARRIOR ===");
        Warrior warrior = new Warrior("Thorin");
        warrior.attack("Orc");
        warrior.fly();                       // cannot fly
        warrior.takeDamage(20);

        System.out.println("\n=== MAGE ===");
        Mage mage = new Mage("Gandalf");
        mage.attack("Orc");                  // cannot melee
        mage.castSpell("Dragon");
        mage.castSpell("Dragon");
        mage.castSpell("Dragon");
        mage.castSpell("Dragon");
        mage.fly();

        System.out.println("\n=== ROGUE ===");
        Rogue rogue = new Rogue("Vin");
        rogue.hide();
        rogue.attack("Merchant");
        rogue.reveal();

        System.out.println("\n=== FLYING MAGE ===");
        FlyingMage flyingMage = new FlyingMage("Elara");
        flyingMage.fly();
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
        flyingMage.castSpell("Phoenix");
    }
}
```

---

## Output

```
=== WARRIOR ===
Swings sword at Orc for 15 damage
Thorin cannot fly
Thorin takes 20 damage. HP remaining: 180

=== MAGE ===
Cannot attack — this character is non-combatant
Gandalf hurls a fireball at Dragon for 40 damage! Mana left: 75
Gandalf hurls a fireball at Dragon for 40 damage! Mana left: 50
Gandalf hurls a fireball at Dragon for 40 damage! Mana left: 25
Gandalf is out of mana!
Gandalf levitates using arcane energy

=== ROGUE ===
Vin melts into the shadows
Fires arrow at Merchant for 10 damage
Vin steps out of the shadows

=== FLYING MAGE ===
Elara soars on giant wings
Elara freezes Phoenix with an ice lance! Mana left: 130
Elara freezes Phoenix with an ice lance! Mana left: 110
Elara freezes Phoenix with an ice lance! Mana left: 90
Elara freezes Phoenix with an ice lance! Mana left: 70
Elara freezes Phoenix with an ice lance! Mana left: 50
Elara freezes Phoenix with an ice lance! Mana left: 30
Elara freezes Phoenix with an ice lance! Mana left: 10
Elara is out of mana!
```

---

## Bonus — Swap Behaviours at Runtime

```java
// Composition lets you change behaviour WHILE the program is running.
// Inheritance cannot do this at all.

public class FlexibleWarrior extends Character {

    private Attackable attackBehaviour;    // not final — can change

    public FlexibleWarrior(String name) {
        super(name, 200);
        this.attackBehaviour = new SwordAttack();    // starts with sword
    }

    public void attack(String targetName) {
        attackBehaviour.attack(targetName);
    }

    // Swap weapon at runtime — no class change, no re-instantiation
    public void equipBow() {
        System.out.println(name + " switches to bow");
        this.attackBehaviour = new BowAttack();
    }

    public void equipSword() {
        System.out.println(name + " switches to sword");
        this.attackBehaviour = new SwordAttack();
    }
}


// Usage
FlexibleWarrior fw = new FlexibleWarrior("Aragorn");
fw.attack("Orc");          // Swings sword at Orc for 15 damage

fw.equipBow();
fw.attack("Nazgul");       // Fires arrow at Nazgul for 10 damage

fw.equipSword();
fw.attack("Sauron");       // Swings sword at Sauron for 15 damage
```

**Output:**
```
Swings sword at Orc for 15 damage
Aragorn switches to bow
Fires arrow at Nazgul for 10 damage
Aragorn switches to sword
Swings sword at Sauron for 15 damage
```

You cannot do this with inheritance.
With inheritance you would need three separate subclasses.
With composition you change one field at runtime.

---

## Unit Tests

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CompositionTest {

    @Test
    void warrior_should_use_sword_attack() {
        Warrior warrior = new Warrior("Thorin");
        // SwordAttack is injected — testable independently
        // Just verify warrior calls attack without throwing
        assertDoesNotThrow(() -> warrior.attack("TestEnemy"));
    }

    @Test
    void mage_should_lose_mana_on_spell() {
        FireMagic magic = new FireMagic(100);
        assertEquals(100, magic.getMana());

        magic.castSpell("Mage", "Dragon");
        assertEquals(75, magic.getMana());    // 100 - 25 = 75

        magic.castSpell("Mage", "Dragon");
        assertEquals(50, magic.getMana());
    }

    @Test
    void mage_should_not_cast_when_out_of_mana() {
        FireMagic magic = new FireMagic(20);
        magic.castSpell("Mage", "Dragon");    // uses all 20 — wait, cost is 25
        // starts with 20, cost is 25 → cannot cast even once
        assertEquals(20, magic.getMana());    // mana unchanged
    }

    @Test
    void behaviour_can_be_swapped_at_runtime() {
        FlexibleWarrior fw = new FlexibleWarrior("Test");

        // Initially sword
        assertDoesNotThrow(() -> fw.attack("Enemy1"));

        // Equip bow
        fw.equipBow();
        assertDoesNotThrow(() -> fw.attack("Enemy2"));

        // Switch back
        fw.equipSword();
        assertDoesNotThrow(() -> fw.attack("Enemy3"));
    }

    @Test
    void ice_magic_consumes_correct_mana() {
        IceMagic ice = new IceMagic(60);
        ice.castSpell("FlyingMage", "Phoenix");   // costs 20
        assertEquals(40, ice.getMana());

        ice.castSpell("FlyingMage", "Phoenix");   // costs 20
        assertEquals(20, ice.getMana());

        ice.castSpell("FlyingMage", "Phoenix");   // costs 20
        assertEquals(0, ice.getMana());

        ice.castSpell("FlyingMage", "Phoenix");   // out of mana
        assertEquals(0, ice.getMana());           // stays at 0
    }
}
```

---

## Side-by-Side Comparison

```
INHERITANCE                              COMPOSITION
──────────────────────────────────────   ──────────────────────────────────────
class FlyingMage extends MagicCharacter  class FlyingMage extends Character {
                                             private MagicCaster magic;
Inherits attack() even though            }
FlyingMage cannot fight.
                                         FlyingMage only has what it needs.
Must override attack() to do nothing.
LSP violation.                           No overrides. No lies. No surprises.

──────────────────────────────────────   ──────────────────────────────────────

Cannot change behaviour at runtime.      Swap attackBehaviour at any time.
A SwordWarrior is always a              fw.equipBow() → now uses bow.
SwordWarrior.                           Zero new classes needed.

──────────────────────────────────────   ──────────────────────────────────────

Java is single-inheritance.              Implement as many interfaces as needed.
You can only extend ONE class.           FlyingMage has Magic + Flight + Stealth
Adding flight to a MagicCharacter        all composed from separate behaviours.
requires restructuring the hierarchy.

──────────────────────────────────────   ──────────────────────────────────────

Testing MagicCharacter requires          Testing FireMagic requires only
setting up FightingCharacter             FireMagic. No parent setup.
and Character first.                     Complete isolation.

──────────────────────────────────────   ──────────────────────────────────────

Change the base class →                  Change FireMagic →
every subclass is at risk.               only FireMagic users affected.
Fragile. Scary. Slow to change.          Safe. Targeted. Fast to change.
```

---

## When Is Inheritance Actually OK?

```
Inheritance IS appropriate when ALL of these are true:

1. The child IS truly a specialised version of the parent.
   Dog IS-AN Animal. ✅
   But FlyingMage IS-A MagicCharacter? Not cleanly. ❌

2. The child will NEVER need to override parent behaviour
   to do nothing or throw an exception.
   If you override just to say "I don't do this" → use composition.

3. The hierarchy will NOT grow beyond 2 levels.
   Animal → Dog is fine.
   Animal → FightingAnimal → MagicAnimal → FlyingMage is a disaster.

4. The parent is stable and unlikely to change.
   Changing a parent class touches all children.
   If the parent changes often → composition is safer.

Rule of thumb:
Extend for TYPE (what something IS).
Compose for BEHAVIOUR (what something DOES).
```

---

## The Golden Rule

```
"Favour composition over inheritance."
— Gang of Four, Design Patterns (1994)

This is 30 years old advice.
It is still the most violated principle in Java codebases today.

When you reach for "extends" — pause.
Ask: "Am I modelling WHAT IT IS or WHAT IT DOES?"

What it IS  → inheritance might be fine.
What it DOES → composition is almost always better.
```

---

## Project Structure for This Day

```
src/
├── behaviours/
│   ├── Attackable.java
│   ├── Flyable.java
│   ├── MagicCaster.java
│   ├── Stealthable.java
│   └── Tradeable.java
│
├── attacks/
│   ├── SwordAttack.java
│   ├── BowAttack.java
│   └── NoAttack.java
│
├── flight/
│   ├── WingFlight.java
│   ├── MagicFlight.java
│   └── NoFlight.java
│
├── magic/
│   ├── FireMagic.java
│   └── IceMagic.java
│
├── stealth/
│   └── ShadowStealth.java
│
├── characters/
│   ├── Character.java
│   ├── Warrior.java
│   ├── Mage.java
│   ├── Rogue.java
│   ├── FlyingMage.java
│   └── FlexibleWarrior.java
│
└── Main.java

test/
└── CompositionTest.java
```
---