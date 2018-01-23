---
title: VR Locomotion tips, 2017 Edition
date: 2018-01-23 03:07:01 Z
layout: post
image: https://i.imgur.com/Hylve3w.jpg
---

For a medium that is only 2 years old in the hand of the mainstream customer, we can say that virtual reality's user interaction and experience is evolving very fast. And more with the introduction of AAA studios in the game like Bethesda.
Let's pick some of 2017 UX ideas and add them to more classic locomotion methods.

*Design ideas inspired by Windows MR's Cliffhouse (the VR menu) as well as Bethesda's Doom/Fallout VR.*

## **Pre-Requisites:**
Repo is available [here](https://github.com/Guendeli/GearVr-motion-plus)
Foremost, we will be using Oculus GearVR for these, it should work without hassle for the Rift since it is using the Oculus Utilitie's CameraRig, and could easily be rewritten for Daydream and/or SteamVR as most of the interaction is in [ArcTeleporter.cs](https://github.com/Guendeli/GearVr-motion-plus/blob/master/Assets/Scripts/Locomotion/SharedCode/ArcTeleporter.cs).


## **What is in there ?:**
Our main orientation method will be the Arc (also known as Bezier) teleporter that let us point to where we are going using hand tracked controllers. and the two classical locomotions would obviously be the Blink Teleport, and the Dash. Then we'll see what could we add to make them better.
but first, let's tackle a boring part about curves.

## **Bezier Curves**
A Quadratic Bézier curve would be the easiest way to implement an arc teleporter, the other being a parabolic curve.
It consists of three points: start/end points and a control point. And two line segments: a first one 'S0' between the start and control point, and a second 'S1' between the control and the end points.

![altText](https://i.imgur.com/8MtqOGU.png)

Two additional points, Q0 and Q1 will be obtained by interpolating along S0 and S1 by some value t with a range between 0 and 1. And another segment S2 will be created by connecting Q0 and Q1, then by interpolating along this segment, a point 'Q2' would be the result that creates our curve.

![altText](https://scontent.fmad3-7.fna.fbcdn.net/v/t39.2365-6/22879649_542336362777632_507106561005453312_n.gif?oh=5b6c3b1502c70f2a6a26ac8915ce68e3&oe=5AF2D279)

This can be expressed with this snippet:

```Java
Vector3 SampleCurve(Vector3 start, Vector3 end, Vector3 control, float t) {
      // Interpolate along line S0: control - start;
      Vector3 Q0 = Lerp(start, control, t);
      // Interpolate along line S1: S1 = end - control;
      Vector3 Q1 = Lerp(control, end, t);
      // Interpolate along line S2: Q1 - Q0
      Vector3 Q2 = Lerp(Q0, Q1, t)
      return Q2; // Q2 is a point on the curve at time t
  }
```

## **Blink Teleport**
This method was the first to be adopted as a motion sickness-free viable method of locomotion in most early VR experiences,
either with a fading in/out or immediate. by either pointing to static teleport points or -what we are using- the Arc teleporter.

**_Sidenote/History:_** In VR, the developer gives full control of the camera to the player's...neck, that is part of the deal. But for the sake of **[accessibility](http://gameaccessibilityguidelines.com/)**, some way to support seated vr play was necessary. Oculus started to ask for a "snap rotation" using the joysticks as a best practice for their games and thus it became common nowadays. That bring us to our issue: mobile VR controllers only have a trigger and a touchpad.

Microsoft's implemented this just right in their cliffhouse menu by letting the user calibrate it's facing rotation while teleporting. A modest implementation looks like:

<blockquote class="imgur-embed-pub" lang="en" data-id="3SthBls"><a href="//imgur.com/3SthBls">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

```Java
//......SOME CODE.......

// Unity Methods
void Update () {
		if (!HasController) {
			return; 
		}
		bool currentTriggerState = OVRInput.Get (OVRInput.Button.PrimaryIndexTrigger);

		// If the trigger was released this frame
		if (lastTriggerState && !currentTriggerState) {
			Vector3 forward = objectToMove.forward;
			Vector3 up = Vector3.up;

			// If there is a valid raycast
			if (arcRaycaster!= null && arcRaycaster.MakingContact) {
				if (objectToMove != null) {
					if (teleportedUpAxis == UpDirection.TargetNormal) {
						up = arcRaycaster.Normal;
					}
          objectToMove.position = arcRaycaster.HitPoint + up * height;
				}
			}

			if (OVRInput.Get (OVRInput.Touch.PrimaryTouchpad)) {
				forward = TouchpadDirection;
			}
      objectToMove.rotation = Quaternion.LookRotation(forward, up);
		}
		lastTriggerState = currentTriggerState;
}

// Helper Methods
Matrix4x4 ControllerToWorldMatrix {
		get {
			if (!HasController) {
				return Matrix4x4.identity;
			}

			Matrix4x4 localToWorld = arcRaycaster.trackingSpace.localToWorldMatrix;

			Quaternion orientation = OVRInput.GetLocalControllerRotation(Controller);
			Vector3 position = OVRInput.GetLocalControllerPosition (Controller);

			Matrix4x4 local = Matrix4x4.TRS (position, orientation, Vector3.one);

			Matrix4x4 world = local * localToWorld;

			return world;
		}
	}

Vector3 TouchpadDirection {
		get {
			Vector2 touch = OVRInput.Get(OVRInput.Axis2D.PrimaryTouchpad);
			Vector3 forward = new Vector3 (touch.x, 0.0f, touch.y).normalized;
			forward = ControllerToWorldMatrix.MultiplyVector (forward);
			forward = Vector3.ProjectOnPlane (forward, Vector3.up);
			return forward.normalized;
		}
	}

  //...........MORE CODE...............
```

## **Dash Teleport:**
Very cool for action fast paced games/experiences, the dash gives the user a sense of power and was well designed in DOOM VR (as well as the flat screen Doom game btw), but if improperly implemented can induce sickness.
The culprit here is the *[Field of View](https://en.wikipedia.org/wiki/Field_of_view)*, while it can be pretty wide if you're in your couch watching TV, it get reduced if you're walking, running or focusing on a task in front of you.
that bring us to the *tunneling effect*.

<blockquote class="imgur-embed-pub" lang="en" data-id="u9h5xWY"><a href="//imgur.com/u9h5xWY">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

Implementation makes use of a mask attached to a Quad/Plane/UIPanel child of our main camera. then by tweaking the alpha or the scale it creates a cool vignette effect that reduces the FOV while moving. a sample of a mask can be created in 3 clicks in photoshop and looks like:

![altText](https://i.imgur.com/fM3NHuu.png)

**Conclusion:**
>VR is still a great terrain for R&D and experimentation, and what is accepted today as good practice will surely become tomorrow's mistake. so don't take these for granted and feel free to experiment, tweak and playtest(A LOt) as you want.










