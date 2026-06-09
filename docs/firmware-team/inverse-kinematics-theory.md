# Inverse Kinematics Theory for a (2)-Link Robot Arm

## Purpose of this file

This file explains the math theory behind the inverse kinematics (IK) we plan to use for the robot arm firmware

Inverse kinematics answers this question:

> Given a target position, what shoulder and elbow angles are needed for the robot arm to reach it?

For now, we are starting with 2D version of our arm and later, this can be expanded to include full 3D

---

## 1. Coordinate convention

For the first version, we treat the arm as a 2-link planar arm moving in`y-z` plane

- `z` = forward/back distance from the shoulder
- `y` = up/down distance from the shoulder
- Target position = `(z, y)`

---

## 2. Link and angle definitions

Let:

- `L1` = length of shoulder-to-elbow link
- `L2` = length of elbow-to-wrist link
- `r` = straight-line distance from shoulder to target
- `θ1` = shoulder angle
- `θ2` = elbow angle

Goal:

```text
Given target (z, y), find θ1 and θ2.
```

Our arm can be visualized as:

```text
shoulder ---- L1 ---- elbow ---- L2 ---- target
```

Straight-line distance from shoulder to target is `r`

---

## 3. First Important Eq: Distance to the target

First, calculate the distance from the shoulder to the target using Pythag theorem

```text
r = sqrt(z^2 + y^2)
```

This forms a triangle with side lengths:

```text
L1 = upper arm link
L2 = forearm link
r  = shoulder-to-target distance
```

This triangle is what allows us to use trigonometry to solve for the joint angles

---

## 4. Check if target is reachable

Before calculating angles, check whether the target is physically reachable

The farthest the arm can reach is:

```text
L1 + L2
```

The closest the arm can reach is:

```text
abs(L1 - L2)
```

Therefore, target is reachable only if:

```text
abs(L1 - L2) <= r <= L1 + L2
```

If the target is outside this range, arm cannot reach it exactly

In firmware, this should become like a constant checking function before sending any motor commands

---

## 5. Elbow angle using the law of cosines

To solve for the elbow angle, we use the law of cosines

- Trigonometric formula that relates the lengths of a triangle's sides to the cosine of one of its angles
- generalizes the Pythagorean theorem, meaning it applies to any arbitrary triangle

For the triangle made by `L1`, `L2`, and `r`:

```text
cos(θ2) = (r^2 - L1^2 - L2^2) / (2 * L1 * L2)
```

Then solve for `θ2 `:

```text
θ2 = arccos((r^2 - L1^2 - L2^2) / (2 * L1 * L2))
```

This gives elbow angle for one valid configuration

There are usually two possible elbow configurations:

```text
elbow-up
elbow-down
```

Mathematically, this can be represented by using either:

```text
θ2 = +arccos(...)
```

or:

```text
θ2 = -arccos(...)
```

For our implementation, it's the elbow-up configuration since:

- `L1` goes upward from the shoulder
- then `L2` goes downward/forward from the elbow to the target

So we would use: `θ2 = -arccos(...)`

for elbow-up

---

## 6. Shoulder angle overview

The shoulder angle i would say is more complicated than the elbow angle

The shoulder does not simply point directly at the target because the elbow bends

So, the shoulder angle is split into two parts:

```text
θ1 = φ - a
```

Where:

- `φ` = direct angle from shoulder to target
- `a` = offset angle caused by elbow bend

In other words:

```text
shoulder angle = target direction angle - elbow offset angle
```

---

## 7. Direct target angle φ

`φ` is the angle of the straight-line direction from the shoulder to the target.

Because our 2D plane uses:

```text
z = forward/back
 y = up/down
```

we calculate:

```text
φ = atan2(y, z)
```

We use `atan2` instead of regular `atan` b/c `atan2` considers the signs of both inputs and returns the angle in the correct quadrant

This matters because a target could be above, below, forward, or behind the shoulder depending on coordinate values

---

## 8. Offset angle a

This calculation is where I believe it needs a bit of interpretation/understanding

`a` is the angle between:

```text
the first link L1
```

and

```text
the straight line from the shoulder to the target (r)
```

This angle tells us how much the upper arm is rotated away from the direct shoulder-to-target line because the elbow is bent

The formula is:

```text
a = atan2(L2 * sin(θ2), L1 + L2 * cos(θ2))
```

This comes from breaking the second link `L2` into two components relative to the direction of the first link, basically a forward part adding to L1, and a upward/downward part adding to L1:

```text
forward part           = L2 * cos(θ2)
upwards/downwards part = L2 * sin(θ2)
```

Putting together the forward part and the upward/downward part:
- From shoulder to elbow you already moved forward by: `L1`
- Then the second link adds more forward distance: `L2 cos(θ2)`
- So the total forward distance from shoulder to target is: `L1 + L2 cos(θ2)`
- The upward distance is just: `L2 sin(θ2)`


Simply put, the total forward distance is:

```text
L1 + L2 * cos(θ2)
```

The perpendicular distance is:

```text
L2 * sin(θ2)
```

So the offset angle is:

```text
a = atan2(perpendicular distance, forward distance)
```

which gives:

```text
a = atan2(L2 * sin(θ2), L1 + L2 * cos(θ2))
```

---

## 9. Shoulder angle equation

Once `φ` and `a` are known, calculate the shoulder angle:

```text
θ1 = φ - a
```

So the full shoulder calculation is:

```text
φ = atan2(y, z)

a = atan2(L2 * sin(θ2), L1 + L2 * cos(θ2))

θ1 = φ - a
```

This gives shoulder angle for the chosen elbow configuration

---

## 10. Full (2)-link IK formula summary

Given:

```text
L1 = shoulder-to-elbow link length
L2 = elbow-to-wristlink length
z  = forward/back target coordinate
y  = up/down target coordinate
```

Calculate:

```text
r = sqrt(z^2 + y^2)
```

Check reachability:

```text
abs(L1 - L2) <= r <= L1 + L2
```

Calculate elbow angle:

```text
θ2 = -arccos((r^2 - L1^2 - L2^2) / (2 * L1 * L2))
```

Calculate target direction:

```text
φ = atan2(y, z)
```

Calculate shoulder offset:

```text
a = atan2(L2 * sin(θ2), L1 + L2 * cos(θ2))
```

Calculate shoulder angle:

```text
θ1 = φ - a
```

---

## 11. Worked example

Suppose:

```text
L1 = 10
L2 = 10
target: z = 10, y = 10
```

### Step 1: Calculate r

```text
r = sqrt(z^2 + y^2)
r = sqrt(10^2 + 10^2)
r = sqrt(200)
r = 14.14
```

### Step 2: Calculate θ2

```text
cos(θ2) = (r^2 - L1^2 - L2^2) / (2 * L1 * L2)
```

Substitute values:

```text
cos(θ2) = (14.14^2 - 10^2 - 10^2) / (2 * 10 * 10)
cos(θ2) = (200 - 100 - 100) / 200
cos(θ2) = 0
```

Therefore:

```text
θ2 = -arccos(0)
θ2 = -90 degrees
```

### Step 3: Calculate φ

```text
φ = atan2(y, z)
φ = atan2(10, 10)
φ = 45 degrees
```

### Step 4: Calculate a

```text
a = atan2(L2 * sin(θ2), L1 + L2 * cos(θ2))
```

Substitute values:

```text
a = atan2(10 * sin(-90), 10 + 10 * cos(-90))
a = atan2(-10, 10)
a = -45 degrees
```

### Step 5: Calculate θ1

```text
θ1 = φ - a
θ1 = 45 degrees - (-45) degrees
θ1 = 90 degrees
```

Final result:

```text
θ1 = 90 degrees
θ2 = -90 degrees
```

So shoulder goes upward first, then the second link goes forward/downward toward the target

---

## 12. Forward kinematics check

A useful way to verify the IK result is to plug the angles back into the forward kinematics equations (will not go too into FK as it's somewhat irrelevant for what we need at the moment for this project)

For the `z-y` plane:

```text
z = L1 * cos(θ1) + L2 * cos(θ1 + θ2)

y = L1 * sin(θ1) + L2 * sin(θ1 + θ2)
```

Using the example result:

```text
θ1 = 0 degrees
θ2 = 90 degrees
L1 = 10
L2 = 10
```

Check `z`:

```text
z = 10 * cos(0) + 10 * cos(0 + 90)
z = 10 * 1 + 10 * 0
z = 10
```

Check `y`:

```text
y = 10 * sin(0) + 10 * sin(0 + 90)
y = 10 * 0 + 10 * 1
y = 10
```

So the calculated angles correctly reach:

```text
(z, y) = (10, 10)
```

---

## 13. Notes for firmware implementation (*******)

The equations above give theoretical math angles, but actual servo angles may not be the same

For our real robot arm, we will eventually need to account for:

- servo zero positions
- servo rotation direction
- mechanical mounting orientation
- joint limits
- safe angle ranges
- unreachable targets
- smoothing between movements

A possible servo conversion format could be:

```text
shoulder_servo_angle = shoulder_offset + shoulder_direction * θ1
elbow_servo_angle    = elbow_offset    + elbow_direction    * θ2
```

Where:

```text
shoulder_offset / elbow_offset
```

represent the physical calibration offsets, and:

```text
shoulder_direction / elbow_direction
```

represent whether the servo angle increases or decreases relative to the math angle

This conversion is separate from IK theory and should be tested on physical arm

---

## 14. What we have currenty (what this file covers)

For now, this IK-theory covers:

```text
2D IK using the y-z plane
shoulder angle θ1
elbow angle θ2
reachability checking and elbow-up/down
```

Important steps for first theoretical stage of our robot arm firmware work

Later work might possibly include things like:

```text
full x,y,z coordinate handling (update this file if we do so?)
pseudo-code implementation of our 2D IK theory above
conversion of our theoretical math angles into servo angles
```

---

## References

- ChatGPT
- https://livephysics.com/cheat-sheets/robotics-inverse-kinematics-reference/
- https://medium.com/%40manuelmort/inverse-kinematics-of-two-link-planar-arm-geometric-approach-5f3ffdfde16d
- https://www.youtube.com/watch?v=_3Dy30ltDA0