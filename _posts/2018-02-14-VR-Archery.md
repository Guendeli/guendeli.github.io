---
title: VR Archery and two handed aiming
date: 2018-02-14 00:33:01 Z
layout: post
image: https://i.imgur.com/tppLzYn.jpg?1
---

Ones of the most common interactions we see in the VR medium are throwing things and shooting at things. And while holding a handgun or a knife might sound straightforward to implement, aiming with a two handed weapon like a sniper rifle or - in our case - a bow, comes with it sets of challenge.
With this in mind, i've tryed prototyping and here comes my findings.

*What follows have been implemented using Oculus Utilities for Unity, but can be rewritten to other VR platforms*
*Golem Model by Fluzo studios(c)*

<iframe width="560" height="315" src="https://www.youtube.com/embed/THfRLHMElkY" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## **Aiming Direction:**
When holding a one-handed item, it is pretty obvious that we are gonna raycast from the holding hand in its forward vector direction. But when using a weapon that requires two hands to operate, which one gets the upper hand ? well kinda both.
*Code below is handedness agnostic, lefties matter*
```Java
private void Aim(){
    transform.rotation = Quaternion.LookRotation(gripHand.position - arrowHand.position, gripHand.TransformDirection(Vector3.forward));
}
```
Our direction vector is determined by vector substraction (direction = target vector - origin vector) , and since our grip hand's (the hand holding the bow) Oculus Touch is facing upwards we have to pass that as a second argument to our Quaternion.LookRotation() method.
Of course our bow's position would obviously be the grip hand's one.

## **Bow String Pull Animation:**
Multiple possibilities here, but in VR optimization is mandatory so we are gonna keep it very simple.
my bow is a rigged mesh having one animation imported with a Legacy Animation component (gif below).

![altText](https://media.giphy.com/media/l4pTr7Sbznz4iTieI/giphy.gif)

goal here is to set the animation frame based on pull distance.

```Java
private void PullString(){

    // mathf.Clamp clamps a value between a min/max values
    pull = Mathf.Clamp(
        Vector3.Distance(gripHand.position,arrowHand.position),
        0,
        maxPullDistance 
    );

    // animation setup
    animation["pull"].speed = 0; // we won't be playing it automaticly
    animation["pull"].time = pull; // we set the frame
    animation.Play("pull");

    previousPull = pull;
}
```

## *Bonus Points:*
* Only relying on the grip button (OVRInput.Button.PrimaryHandTrigger) to interact made it natural since that's how we grab things in real life.
* Adding a Trigger collider behind the Main Camera that spawns arrows made it also feel right, most archers in the media wear their quiver on their back. tested on mom and dad, they didn't need any tutorial to understand that feature.



