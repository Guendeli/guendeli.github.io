---
title: Views on Unity C#
date: 2018-01-08 23:31:01 Z
layout: post
image: https://media.licdn.com/media/gcrc/dms/image/C4E12AQF42VbkWmEIAw/article-cover_image-shrink_720_1280/0?e=2131315200&v=beta&t=PBbvLuFwyB7ZxvTZii7Fx1EEervpBdZhin6qBt2aTwo
---

Style and naming conventions might vary between every team / company / individual out there and Unity3D's API use a strange naming convention. So i had to modify mine at most gigs i took to make the code match theirs.

What follows is my own view, this is not to be taken for granted and please, feel free to give more feedback in the comments section.

**Formatting**

First and foremost, i prefer Visual Studio always. It's out in OSX now but if different i'll use Windows VS settings.

At default Unity make a new script look like this:

```C#
public class JohnDoe : MonoBehaviour {

	// Use this for initialization
    void Start () {
	  //Code here
	}
}
```

I'll use Microsoft's formatting style. The first step is to delete the end curly brace and retype it, so it formats the class like this:

```C#
public class JohnDoe : MonoBehaviour
{
  // Use this for initialization
  void Start()
  {
    //Code here
  }
}
```
Ensuring your spaces are also formatted appropriately. = and == should have spaces around them. The curly brace trick will fix these, too.

**Comments**

In default MonoBehaviours created, there is a stupid default comment made by Unity. If any comment is nonsense, _DELETE IT_!

I realy like comments above methods to have standard C# summaries , three /// will generate them in VS.

```C#
public class JohnDoe : MonoBehaviour
{
  ///summary
  /// This is where I would put a meaningful comment about JohnDoe starting up.
  ///summary
  void Start()
  {
    //Code here
  }
}
```
summaries may be _required_ in the those cases:

- every single class definition
- public methods expected to be called by other classes
- methods in base classes expected to extended later
- Anything confusing

These are helpful for other developers returning to your code later on, which might even be you! And if a game is successful expect to be making changes to it for a long time.

**Empty methods**

Delete empty methods that are not conforming to a base class or interface. If it is a Unity method like Start or Update it will actually slow the game down if you do not remove them.

**Commented code**

If it is a quick hack that is going to be fixed later, it is OK to _temporarily_ comment out the block but if there is commented code that you are not sure what it does, or has been there a for a long while, _DELETE IT_!

Some very smart people invented something called Source Control, there is no reason to commit commented code long term.

**Classes and files**

1 class per file! The only exception would be a file named Enumerations.cs or Structs.cs that contain many small enums or structs.

**Public fields**

The main difference for Unity's naming is public fields. Most devs would name a public field like this:

```C#
public class JohnDoe
{
  public float  Intensity;
  public string FirstName;
  public string LastName;
}
```
The problem is, Unity's APIs look like this:

```C#
public class JohnDoe
{
  public float  intensity;
  public string firstName;
  public string lastName;
}
```
So their convention for public fields is to start with a lower case letter, and camel case the rest. What the Unity Editor will do is convert firstName to First Name in the inspector, so camel casing is extremely important or it would say Firstname.

**Private fields**

My preferred shot is to remain unchanged from standard C#, i'll go naming private fields as such:

```C#
public class JohnDoe
{
  private float  _intensity;
  private string _firstName;
  private string _lastName;
}
```
And i'll _ALWAYS_ put the private keyword. It is way more readable. Also it can confuse past Java fellow devs, C# defaults to private while Java defaults to public.

**Methods**

These don't change from Microsoft's. They should look like this:

```C#
public class JohnDoe
{
  public void SlapTheGuyNotFollowingCodingStandards()
  {
    //I'll show you a SOLID design principle
  }
}
```

**Names**

Don't name a class JohnDoeBehavior or JohnDoeController. Just name it JohnDoe.

Don't abbreviate a field, method, or class name. A variable shouldn't be named lastP _ever_. It should be named lastPosition. The only _good_ short names are things like i, j, x, y, z, etc.

**Class visibility**

Always put the public keyword:

```C#
public class JohnDoe
{
  //Members here
}
```

If left off, it defaults to internal, and defaults to private if it is a nested class. We might not want that if we want to use the class in another project. If the class is actually internal or private use the keyword, also.

**Member ordering**

A class should not look like this:

```C#
public class JohnDoe
{
	public Vector3 Point1 = Vector3.zero;
	public Vector3 Point2 = Vector3.zero;
	public float Thickness = 1f;
	public float ScaleMod = .95f;
	Vector3 offset;
	public float OffsetMod = .01f;

	Vector3 lastP1;
	Vector3 lastP2;

	GameObject texture;
	bool decay = false;
	public bool DecayTrigger=false;
	float decayDrop = 1f;
}
```

Group public static, private static, public, private, and method declarations together. Do not mix and match like a crazy fool.

It should look like this:

```C#
public class JohnDoe
{
	public Vector3 point1 = Vector3.zero;
	public Vector3 point2 = Vector3.zero;
	public float thickness = 1f;
	public float scaleMod = .95f;
	public float offsetMod = .01f;
	public bool decayTrigger = false;

	private Vector3 _offset;
	private Vector3 _lastP1;
	private Vector3 _lastP2;
	private GameObject _texture;
	private bool _decay = false;
	private float _decayDrop = 1f;
}
```

**Compiler warnings**

No Reds, No Yellows. Your C# code should _not_ have warnings. If you are not using a variable, _DELETE IT_! I am setting the build server to fail on warnings, so i can make fun of myself if my code code has them.

**God classes**

i don't make ridiculously long classes anymore. If a class is longer than XXX lines, i consider breaking its functionality into multiple classes. I don't want to see another endless GameDataMainManager like the one i had in my previous game.

**Be SOLID**

Learn the [SOLID design principles](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29). Most importantly S and D:

- S - Single Responsibility Principal - each class should have a single "responsiblity" or purpose. A Player class would handle logic for the Player, but not handle saving your game, for example.
- D - Dependencies - depend upon abstractions, not concretions. Anything that must be flexible across platforms should use a base/abstract class or interface. Examples of this are a file system service, in-app purchasing, etc.

As always, thank you for reading. And feel free to jump in the comment section for your own feedbacks and style.
