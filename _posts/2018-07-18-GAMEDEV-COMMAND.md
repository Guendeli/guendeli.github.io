---
title: Patterns in gameplay programming, Command
date: 2018-07-18 03:00:00 Z
layout: post
image: https://i.imgur.com/sfV8KUW.jpg
---

I'll be starting a series on design patterns applied to real-life gameplay programming using Unity3D along with sample scene explaining one mechanic, by following Matt, our imaginary gameplay programmer who just joined the studio.

## CTRL+Z:
Matt came to his office this morning, motivated, he is working on Paladins Tactics, a turn based RPG for mobiles. When opening Slack, he discovers his product manager just assigned him this story

> As a player, i should be able to undo my actions and change them
> before ending the turn.

With an explanation image that shows the wanted result:

![enter image description here](https://media.giphy.com/media/Rd6sv6StGGedfnPy2b/giphy.gif)

Our developer scratches his head, all of the gameplay mechanics like movements and attacks were deeply hardcoded in the Player related components that decoupling them would be insanely hard. 
Well, not that much, enters the Command pattern, quoting wikipedia:

> In object-oriented programming, the **command pattern** is a "Behavioral pattern" pattern in which an object is used to encapsulate all information needed to perform an action or trigger an event at a later time.

'at a later time' may make it sound like a callback, a simple slugline for it would be "an object-oriented replacement for callbacks"

In every video game there is some functionnality that reads raw user input from the player and translate it into something of value inside the game world, and Matt's implementation was dead simple but working during conception:
```C#
     void HandleInput()
    {
        //....other inputs tied here
        if (Input.GetButton("BUTTON_X"))
        {
            Move();
        }
        //....other inputs tied here
    }
```
While this was working just fine during the early stages, it poses the problem of not letting the user configure his/her own input scheme, it forces us to write duplicate code for other moving units not controlled by the player and finally, makes implementing our undo feature a pain in the arse, enter our pattern.

## Command:
we start by defining a base class that represents any triggerable game action:
```C#
/// <summary>
/// The 'Command' abstract class that we will inherit from
/// </summary>
abstract class Command
{
    public abstract void Execute();
    public abstract void UnExecute();
}
```
We could then create a set of subclasses for each of our actions in game, in our case here, just movement:
```C#
class MoveCommand : Command
{
    private MoveDirection _direction;
    private float _distance;
    private GameObject _gameObject;
    //Constructor
    public MoveCommand(MoveDirection direction, float distance, GameObject gameObjectToMove)
    {
        this._direction = direction;
        this._distance = distance;
        this._gameObject = gameObjectToMove;
    }
    //Execute new command
    public override void Execute()
    {
        MoveOperation(_gameObject, _direction, _distance);
    }
    //Undo last command
    public override void UnExecute()
    {
        MoveOperation(_gameObject, InverseDirection(_direction), _distance);
    }

    // Rest of code, MoveOperation can just be a method that translates a Transform with a given distance
}
```
By passing it an actor we want move, we loosen restrictions and make that command also callable by an AI agent rather than only the player input controller. Then we can just implement the rest of our Player Movement Handler, it would be a Monobehaviour existing somewhere in our scene.
Note to mention, this implementation varies slightly from a standard one due to the Undo mechanic, commands here represent something that can be done *at a specific point in time* and thus we should keep track of them.

```C#
public class PlayerMovementHandler : MonoBehaviour
{
    public float moveDistance = 10f;
    public GameObject objectToMove;

    private List<Command> commands = new List<Command>();
    private int currentCommandNum = 0;

    void Start()
    {
        // some null checks
    }

    private void Move(MoveDirection direction)
    {
        MoveCommand moveCommand = new MoveCommand(direction, moveDistance, objectToMove);
        moveCommand.Execute();
        commands.Add(moveCommand);
        currentCommandNum++;
    }

    public void Undo()
    {
        if (currentCommandNum > 0)
        {
            currentCommandNum--;
            MoveCommand moveCommand = (MoveCommand)commands[currentCommandNum];
            moveCommand.UnExecute();
            commands.Remove(moveCommand);
        }
    }
    // Simple public move methods below
    // to attach to UI buttons, Gamepad buttons
    // EventSystem or whatever
    //....
}
```

a fully downloadable sample can be foud [here](https://github.com/Guendeli/unity3d-patterns).

### reference:
[Robert Nystrom's Game Programming Patterns](http://gameprogrammingpatterns.com) book, language and framework agnostic.

Credits to Free visual assets used in this post:
[RPG Hero HP](https://assetstore.unity.com/packages/3d/characters/humanoids/rpg-hero-hp-121480), [Medieval Town Exteriors](https://assetstore.unity.com/packages/3d/environments/fantasy/medieval-town-exteriors-27026), [Simple UI](https://assetstore.unity.com/packages/2d/gui/icons/simple-ui-103969)

