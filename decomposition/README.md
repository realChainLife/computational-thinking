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

