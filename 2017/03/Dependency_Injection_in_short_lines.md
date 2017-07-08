# Dependency Injection in short lines

You are a blacksmith, and your work is to make swords to anyone who has
ordered for a sword. In order to make a sword, you need to use a
tool to make the sword.

```js
Import {Tool,Hammer} from './Tool'
export class Sword{
  public tool:Tool;
  constructor(){
    this.tool = findAHammer();
    console.log('This sword is made by '+this.tool);
  }
}
```

Now your client thinks hammer-made swords are not enough and they wants
you to use a stamping machine to do it. You found the machine and
do the job...

```js
Import {Tool,StampingMachine} from './Tool'
export class BetterSword{
  public tool:Tool;
  constructor(){
    this.tool = new StampingMachine();
    console.log('This sword is made of '+this.tool);
  }
}
```

You found that there is not much different to make a sword by different
tools. You actually don’t care what tool you are using. So you told
your client to bring the tool themselves.

```js
Import {Tool} from './Tool'
export class Sword{
  public tool:Tool;
  constructor(public tool:Tool){
    console.log('This sword is made of '+this.tool);
  }
}
```

So now you don’t have to find the tool yourself and can be focused on
making a better sword.

For a even easier understanding, I’m quoting a line found from
Stackoverflow answer(which actually quoting from other pages, LOL):

```
Dependency injection means giving an object its instance variables.
```

DI is good when you have built the thing really big. Yeah, **really** big one.
It is good when you need to test a little piece of logic from a huge thing
that you would not want to run the whole thing, you can just slice a bit and
do your testing, so that the test should be still valid while you won’t messed
yourself up. But, if you are not making such a big thing or you don’t need to
do unit test(or you just don’t do it), then you don’t need DI to inflate your code.

### Ref:

http://huan-lin.blogspot.com/2011/10/dependency-injection-1.html

http://stackoverflow.com/questions/130794/what-is-dependency-injection

http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html
