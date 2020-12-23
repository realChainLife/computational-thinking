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

## Method Chaining
Method chaining is an object-oriented alternative to function chaining.
A method chain is where a series of methods is called on an object without repeating the name of the object.
This is done by having each chained method change the state of the object it’s called on,
then return that object so it’s available for the next link in the chain:

    damage = Damage.new(attack_rating)
                   .randomize
                   .apply_defense(defense_rating)
                   .apply_scratch_damage
                   .amount

    class Damage
        def initialize(damage_amount)
          @damage_amount = damage_amount
        end

        def randomize
          @damage_amount *= Random.new.rand(0.9..1.0)
            self
        end

        def apply_defense(defense_rating)
          @damage_amount *= (256.0 - defense_rating) / 256.0
            self
        end

        def apply_scratch_damage
          @damage_amount += 1
            self
        end

        def amount
          @damage_amount
        end
    end
    
Method chaining provides the same level of re-use as a public function chain,
as well as the same cost in terms of coupling to implementation details.
If you want to hide these details, rather than making a method chain private, there’s another option.

## Private Method Tree
A calculation can be split up into a “tree” of private methods that each performs part of the calculation. Each method might use data from instance variables or from other private methods. Ruby makes this decomposition especially readable by allowing you to call methods without parentheses (sometimes referred to as “barewords”):

    def damage
      (attack_damage * defense_modifier) + scratch_damage
    end

    private

    def attack_damage
      attack_rating * attack_modifier
    end

    def attack_modifier
      rng.rand(0.9..1.0)
    end

    def defense_modifier
      (max_defense_rating - defense_rating) /
         max_defense_rating
    end

    def max_defense_rating
      256.0
    end

    def scratch_damage
     1
    end

The “tree” is really a tree of multiple levels of abstraction.
For example, looking at damage, we can see it is calculated from 
`attack_damage`, `defense_modifier`, and `scratch_damage`,but we 
can’t see what those are calculated from without stepping down a 
level. These levels of abstraction can be an advantage or a 
disadvantage, depending on how visible that information needs to be.

This example of code decomposition was first published by [Josh Justice](https://www.bignerdranch.com/blog/category/authors/josh-justice/)
