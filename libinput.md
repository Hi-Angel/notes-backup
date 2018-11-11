# Internals

## touchpads

Touchpad device: `x` increases right, decreases left; `y` decreases *up*, increases down.

Scroll log: `vert` and `horiz` concur with `y` and `x` *(see higher)*.

`tp->device->abs.absinfo_x->resolution` *(and acc. for y)* looks like width of the touchpad in mm. That said, for mine y was reported as `80` by libinput, but as `94` by `evemu-describe`.

# hysteresis

Also is currently implemented in `evdev_hysteresis`.

Current implementation suffers of latency, plus there's some vague bugreport with a touchpad that have "dead zones". Idea for another impl.: detect and ignore checkered movement, and only allow "straight" moves.

## idea 2

How about using the wobbliness detection code to record the maximum *length* of wobbliness per given axis, and then… what? *how about filtering out such length every time?*

## wobbliness

Measurement units: pixels, ms.

Wobbliness is: quick movements back&forth along an axis. Can happen in same direction too, but is limited in length.

Mean value: upon real movement jittery might still be present, so it needs to be used for determining the endpoint of movement. But mean value would wobble too.

Current libinput algo looks fine, I'd like not to change it.

## wobbliness solution 1

Wobbliness radius has a point which means cursor position. Moving position a bit should(?) require an effort of length greater than the radius. So ignore movement part of that radius? May we detect that movement started, and stop ignoring?

### wobbliness detection

Record motions within threshold into a byte. Then search for pattern `101` at the byte's end (mask out the rest bits). If it matches, we got the suspect.

1. make the byte
2. add the code to do the recordings
3. add the code to enable hysteresis if motions says it wobbles (i.e. `101` matches).

## ignore movement upon finger up

Approach 1: buffer movement, and ignore the buffered content when finger is up. It'd introduce a latency, not good.

Approach 2: record movements, and undo the movement upon finger-up.

## how much to ignore?

It gotta be quick, and not to look like a swipe. So both length and time is in play.

Length: for starters use ½ touchpad_resolution. It's the length used by older hysteresis algo, which did remove movement upon finger up; at the same time it's too small to be visible upon swipe.

Time: the length alone can satisfy the case when one is aiming. So use in addition 50ms of time — it's too quick for one to both understand they got the target they aimed for, and lift the finger.

## Approach 2: record movements, and undo the movement upon finger-up.

Use circular buffer of only 5 elements, because upon finger up there won't be much movement.

1. Record every movement.
2. if finger up then
3. sum up through the list of movements their lengths until the last satisfying time threshold
4. if sum > 0 then undo movements till this last one

`undo movements` is probably easier by emulating the move in reverse order.

# wobbly 2-fingers scroll when fingers move in opposite directions

## data:

`tp_gesture_state` is `GESTURE_STATE_SCROLL` when scroll happens. Also, 2 `tp_touch`es are active.

"Opposite directions" means "more than perpendicular", i.e. angle > 90° ≥ π/2.

## solution

Even if angle is 91° — which direction you'd move? Taking the middle would just confuse the user. Not to mention bigger angles.

Best, probably, to ignore the movement.

# todos

* rename `device_delta` to `coords_delta`

If I gonna make the general algo for discarding movement, let's write that info somewhere into evemu or whatever.

# twitchy scroll (hw issue)

## Solution 1:

* find resolution per mm
* let `speed_max_mm` = largest possible movement distance in mm
* let `speed_max` = largest possible movement distance in touchpad points
* let `speed_avg` = calculate from `res_per_mm` an average movement speed in touchpad points
* `when speed > speed_max then speed_avg`

### reasoning

Analogous to existing assumptions e.g. that a human can't move left-right-left within 40ms, add an assumption how fast a human can consciously move.

# jump detection

The jump being driven by `ABS_Y` evemu event. Not the `ABS_MT_POSITION_Y`, which I can successfully remove from the testcase without influencing it; whilst removing `ABS_Y` results in record not doing anything at all.

Increasing timestamp to 30ms didn't help.
