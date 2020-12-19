# Code Decomposition

To illustrate the different approaches to decomposition,
let’s take a look at this combat calculation for a role-playing game I’m building:

    damage = attack_rating *
         Random.new.rand(0.9..1.0) *
         (256.0 - defense_rating) / 256.0 +
         1

That calculation is a little complex:

- The attacking character has a base attack rating.
- The damage they do for a given attack varies randomly between 90% and 100% of that value.
- The defending character has a defense rating.
- A defense rating of 256 nullifies the attack entirely,
- while a rating less than 256 nullifies a proportional amount of the damage.
- 1 is added to ensure that, even if the attack is nullified, it always does 1 “scratch” damage.
- (Rather than adding it conditionally, for simplicity I just always add 1 to the damage.)
- The fact that I had to explain the code is a bad sign:
   - it means the code doesn’t communicate its intent without additional commentary.

Let’s look at how decomposition can make its meaning clearer.

## Reassignment
One way to decompose this expression is to break it up into steps,
storing the result of each step in a variable.
This can be referred to as “reassignment,”
because one variable has different values assigned to it at different times:

    damage = attack_rating
    damage *= Random.new.rand(0.9..1.0)
    damage *= (256.0 - defense_rating) / 256.0
    damage += 1

Because each step is much shorter than the original calculation,
the whole is easier to take in. Unfortunately, the unchanging variable name leaves each step’s meaning just as unexplained as it was before.
This weakness is addressed by other decompositions, so it’s almost always better to use a different approach.

## Step Variables
Using multiple variables is the simplest way you can improve the reassigment decomposition.
You can assign each step to a separate variable with a descriptive name:

    attack_modifier = Random.new.rand(0.9..1.0)
    defense_modifier = (256.0 - defense_rating) / 256.0

- ttack_damage = attack_rating * attack_modifier
- damage_after_defense = attack_damage * defense_modifier
- damage_with_scratch = damage_after_defense + 1
Step variables can be reused within the scope of the function they’re defined in, but not elsewhere.
One downside is that the surrounding function still contains every detail of the calculation,
making it difficult to get a high-level understanding.

Other decompositions provide more levels of abstraction and allow code to be reused more broadly.

## Functional Chaining
In a step variable decomposition, each variable names what it’s computing.
Function names do the same thing, which suggests that we could convert each variable into a function.
These functions can be chained together to build up the calculation, passing return values from one into another:

scratch(defend(randomize_damage(attack_rating), defense_rating))

    def randomize_damage(attack_rating, rng = Random.new)
      attack_rating * rng.rand(0.9..1.0)
    end

    def defend(damage, defense_rating)
      damage * damage_modifier(defense_rating)
    end

    def damage_modifier(defense_rating)
      (256.0 - defense_rating) / 256.0
    end

    def scratch(damage)
      damage + 1
    end

With functional chaining, you can read the high-level sequence of operations without needing to think about how each is implemented.
But there’s a related con: that sequence of functions is called from the inside-out,
making it a bit difficult to follow the flow.
To address this, many functional languages provide a pipe operator to turn the expression inside-out:

    attack_rating |> randomize_damage
                  |> defend(defense_rating)
                   |> scratch
                   
Making chainable functions public allows the rest of your app to recombine them in different ways.
If you instead want to limit how much your app is coupled to the implementation of these functions,
you can make them private and use them to build up simpler public functions:

    def damage_for_attack(attack_rating, defense_rating)
      scratch(defend(randomize_damage(attack_rating), defense_rating))
    end
