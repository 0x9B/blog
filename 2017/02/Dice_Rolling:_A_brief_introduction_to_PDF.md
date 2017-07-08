# Dice Rolling: A brief introduction to PDF

Dice, a multiple faced throwable device that provides a random number
since thousands of years ago. Dices represent the randomness of selection
on discrete results. In the following sessions, I am going to briefly
talk about the algorithm and implementation on calculation of dice
related probabilities.

### Everything start from basic

Since 1974 by the release of Dungeons & Dragons (DND),
people started to simulate many action result by using dices.

For example, you are Chris Wong, a Elven archer aiming at a goblin
30m from him. Chris is just a newbie who has just taken some lessons
on archery so his aiming ability is low. Fortunately the world of DND
depends on dice, so eventually Chris will have chance to hit the goblin.

Now to determine whether Chris will hit the goblin, we need to roll
a dice to see. Chris has an Attack Roll of D20+2, and the goblin has
an Armor Class of 15 .

By each unbiased dice of 20 faces, we will have equal chance to obtain
an integer from 1 to 20, and adding 2 to the roll will becomes from 3 to 22.
Let’s count all the possible numbers which is equal or larger than 15.
Now the hit chance can be represented as

```
P(D20+2/15) = n(D20+2 >= 15) / n(D20) = 8 / 20 = 40%
```

But obviously not all the dimensions will use the same rule.
Let’s see how it works in another world.

### A little more complicated case

Log Horizon Series, a lightnovel written by Mamare Touno(橙乃ままれ),
describing players in a MMORPG “Elder Tale”, are transported to the
virtual world and have to live in the medieval-styled fantasy world.
A set of TRPG, namely Log Horizon TRPG is published for readers to
enjoy the story in another point of view.

In the mechanism of LHTRPG, each character have their own hit roll
according to their attribute. For example, the main character
Shiroe(シロエ), has a hit roll of 2D6+2.

The probability mass of his hit roll, regardless of the value to against,
is no longer linear. If we roll a single dice, the probability mass is simple:
every possible outcome have the same number of possible combination.
But the case is different when there is more dices. Calculating the outcome
is easy, but calculating the probability mass is not. For example, if we roll
a 2D6 and we wants a sum of 7, what will be the probability of this?
We need to count all combination of dices of sum equals
to 7: 1+6,2+5,3+4,4+3,5+2,6+1. So the probability will be

```
P(2D6=7) = Count(2D6 = 7) / Count(2D6) = 6 / 36 = 16.7%
```

### Things get more and more complicated

There are many skill effects in LHTRPG that allows characters to temporarily
add dice to their hit roll. Suppose the teammates of Shiroe has buffed him
with better accuracy, which equivalent to adding 1 dice to his hit roll
so now he can roll 3D6+2. But now you find problem: counting combination
of dices become troublesome. Maybe you can use a lot of time counting all
the combinations of 3 Dices, but you cannot count that much when Shiroe
becomes stronger or gaining more buff from his teammate, that the number
of dice may become 5 or 10 or even 100. We need to know how to calculate
it in an efficient way.

Consider an array containing the distribution as below:

```
[0 1 1 1 1 1 1]
```

Which represents the number of combinations of outcome from 0 to 6,
the expected outcome distribution of a 6 faced dice.

By the probability theorem, the probability density function of sum of two
independent variable equals to the convolution of probability density
functions of each of the variable. As every two dices must be independent
to each other, we may calculate the PDF of 2D6 as below:

```
PDF(2D6) = PDF(D6) ⊗ PDF(D6) 
= [0 1 1 1 1 1 1] ⊗ [0 1 1 1 1 1 1] 
= [0 0 1 2 3 4 5 6 5 4 3 2 1]
```

Which ⊗ represents convolution.

Again, since each dice is independent to each other, the third dice must be
independent to the previous two and thus:

```
PDF(3D6) 
= PDF(2D6) ⊗ PDF(D6) 
= PDF(D6) ⊗ PDF(D6) ⊗ PDF(D6)
= [0 0 0 1 3 6 10 15 21 25 27 27 25 21 15 10 6 3 1]
```

And so on and so on. In fact, this is called Convolution Power, similar to
the conventional “multiplication” power, is the number of time that
a function convolute itself.

To summerize, the probability of an arbitrary value x by rolling nDm is:

```
P(nDm = x) = PDF(nDm)(x) / Count(nDm)
```

### Time to get things work

In the following session, I will demonstrate on the implementation
of the calculation above in Javascript.

First we start with the convolution function:

```js
function conv(arr1,arr2){
  let length = arr1.length+arr2.length-1;
  let res = [];
  for(let n = 0;n<length;n++){
    res[n] = 0;
    //Def: f⊗g = ∑f[n-m]g[m] = ∑f[m]g[n-m]
    for(let m = 0;m<arr1.length;m++){
      res[n] += (arr1[m] || 0) * (arr2[n-m] || 0)
    }
  }
  return res;
}
```

Then we can start working on the PDF function of Dices:

```js
function probDist(n,m){
  //n dices m faces
  let f1 = [0];
  for(let i = 1;i<=m;i++){
    f1[i] = 1;
  }
  let fn = [0];
  fn[1] = f1;
  for(let ni = 2;ni<=n;ni++){
    fn[ni] = conv(fn[1],fn[ni-1]);
  }
  return fn[n];
}
```

Finally we can call this function to find out the probability of
specific outcome:

```js
//3D6 >= 10
var target = 10;
var dist = probDist(3,6);
var output = 0;
var sum = Math.pow(6,3); 
for(let i = target;i<dist.length;i++){
  output += dist[i]
}
console.log("The required probability equals to "+(output / sum));
```

```
The required probability equals to 0.625
```

### Optional: Make things even better

In the implementation mentioned, there are two ways that improve
the calculation speed:

1. Dynamic Programming
2. Circular Convolution

When we calculate a PDF of nDm, we must first calculate (n-1)Dm.
If we actually will need to access those previous result, it is better
for us to store all the calculated result first, and lookup the stored
result when we need to access. In such implementation, the function will be like:

```js
function probDist(n,m){
  let fn = {};
  function get(n,m){
    if(!fn[m]){
      fn[m] = [0];
      fn[m][1] = [0]
      for(let i = 1;i<=m;i++){
        fn[m][1][i] = 1;
      }
    }
    if(!fn[m][n]){
      fn[m][n] = conv(get(n-1,m),fn[m][1]);
    }
    return fn[m][n];
  }
  return get(n,m);
}
```

When we calculate a PDF of a higher order or some dices of a lot of faces,
we may encounter convolutions with large arrays. In such situation,
we may use Fast Fourier Transform to enhance the efficiency.
(Not to be included in this article, for more information on FFT,
[Click Here][https://en.wikipedia.org/wiki/Fast_Fourier_transform].)

### Ref:

http://www.dandwiki.com/wiki/SRD:Attack_Roll

http://www.d20srd.org/srd/monsters/goblin.htm

http://www.dandwiki.com/wiki/Archer_(3.5e_Class)

https://en.wikipedia.org/wiki/Probability_density_function#Sums_of_independent_random_variables

https://en.wikipedia.org/wiki/Convolution_power

