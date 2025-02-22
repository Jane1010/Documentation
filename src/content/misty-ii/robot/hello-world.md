---
title: Hello World Tutorial Series
layout: coding.hbs
columns: three
order: 3
---

# {{title}}

Welcome to the Misty II Hello World tutorial series! This document provides a series of brief tutorials that teach how to write and upload a JavaScript skill that brings Misty (and your code) to life. When you finish this series, you can call yourself an experienced roboticist!

## Overview

The Hello World tutorial series is divided into six parts:

1. [Moving Misty's Head](./#moving-misty-39-s-head)
2. [Changing Misty's LED](./#changing-misty-39-s-led)
3. [Playing Sounds](./#playing-sounds)
4. [Driving Misty](./#driving-misty)
5. [Teaching Misty to Wave](./#teaching-misty-to-wave)
6. [Using Face Recognition](./#using-face-recognition)

In the first part of the series, you learn how to create and upload the files Misty needs to run a skill. As you progress through the series, you add new lines of code to the original skill file and update the skill on Misty to see how the additions change her behavior. When you finish all of the sections, you'll have programmed your first fully working skill for the Misty robotics platform.

If you'd like to refer to the code files for this skill, you can [do so at any time on GitHub](https://github.com/MistyCommunity/Tutorials/tree/master/Hello%20World).

## Moving Misty's Head

This part of the Hello World tutorial series teaches how to create skill files and upload them to Misty. You'll also write your first lines of code and teach Misty to move her head in a lifelike way.

---

Each skill you write with Misty's JavaScript SDK requires the following elements:

* a [JavaScript "code" file](../../../misty-ii/coding-misty/local-skill-architecture/#code-file) with the logic and commands that Misty executes when the skill runs
* a [JSON "meta" file](../../../misty-ii/coding-misty/local-skill-architecture/#meta-file) that provides the initial settings and parameters Misty needs to run the skill

To begin, open your favorite text editor. (If you don't have a preference, we suggest Visual Studio Code.) Create a new JavaScript file called `HelloWorld.js`, and save this file to a new directory called `HelloWorld`. Now you can start writing the code to bring Misty to life.

To appear lifelike, Misty's head movement should be spontaneous and random. You can achieve this by registering for a "timer" event and configuring it to invoke a callback function after a random interval of time. In our Hello World skill, we code this callback function to send Misty a head movement command with randomized movement values.

Because the Hello World skill uses random integers throughout, you can start by declaring a helper function called `getRandomInt()`. This function uses the built-in [`Math.floor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) and [`Math.random()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random) JavaScript methods to return a random integer. Declare this function at the top of your `HelloWorld` code file:

```JavaScript
// Returns a random integer between min and max
function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
```

To issue any command to Misty in the local environment, we call methods on the `misty` object. Use the [`misty.RegisterTimerEvent()`](../../../misty-ii/reference/javascript-api/#misty-registertimerevent-alpha) method to register for a new timed event. The syntax for this method is as follows:

```JavaScript
// Syntax
misty.RegisterTimerEvent(string eventName, int callbackTimeInMs, [bool keepAlive], [string callbackRule], [string skillToCall], [int prePauseMs], [int postPauseMs]);
```
The `eventName` argument sets the name of the registered event. The `callbackTimeInMs` argument sets the duration before the event triggers, and `keepAlive` sets whether Misty should remain registered for this event after it triggers a callback function.

Copy the following into your `HelloWorld` skill code to register for a timer event with the name `look_around` that invokes a callback after 5000 - 10000 milliseconds. Set `keepAlive` to `false`, and ignore the rest of the arguments:

```JavaScript
misty.RegisterTimerEvent("look_around", getRandomInt(5, 10) * 1000, false);
```

By default, timer events you create with the `misty.RegisterTimerEvent()` method invoke a callback function with the same `eventName`, prefixed with an underscore. In this case, the callback function is called `_look_around()`. Declare this function in your skill code:

```JavaScript
// The look_around timer event invokes this callback function.
function _look_around(repeat = true) {

}
```

You can place the code to move Misty's head inside the `_look_around()` callback function.

Misty's neck has three movement axes -- pitch (tilt up/down), roll (tilt left/right), and yaw (turn left/right). You can move Misty's head to a new position by using the [`misty.MoveHeadPosition()`](../../../misty-ii/reference/javascript-api/#misty-moveheadposition) method. The syntax for this method is as follows:

```JavaScript
// Syntax
misty.MoveHeadPosition(double pitch, double roll, double yaw, double velocity, [int prePauseMs], [int postPauseMs]);
```

The value range for the `pitch`, `roll`, and `yaw` position arguments is `-5` to `5`. To achieve spontaneous head movement, we use the `getRandomInt()` helper function to return unique values for these position arguments each time Misty calls the `misty.MoveHeadPosition()` method.

Copy the `misty.MoveHeadPosition()` argument into the `_look_around()` callback function:

```JavaScript
function _look_around(repeat = true) {

    // Moves Misty's head to a random position. Adjust the min/max
    // values passed into getRandomInt() to change Misty's range of
    // motion when she calls this method.
    misty.MoveHeadPosition(
        getRandomInt(-5, 5), // Random pitch position between -5 and 5
        getRandomInt(-5, 5), // Random roll position between -5 and 5
        getRandomInt(-5, 5), // Random yaw position between -5 and 5
        100); // Head movement velocity. Decrease for slower movement.
}
```

Misty should continue moving her head until the Hello World skill ends. To do this, you can use the `misty.RegisterTimerEvent()` method inside the `look_around()` event callback to create a loop where Misty re-registers for the `look_around` event each time the callback executes.

Copy the conditional block below into your `_look_around()` callback function. As long as `repeat` is set to `true`, Misty runs the head movement command on a loop until the skill ends.

```JavaScript
function _look_around(repeat = true) {

    // Moves Misty's head to a random position. Adjust the min/max
    // values passed into getRandomInt() to change Misty's range of
    // motion when she calls this method.
    misty.MoveHeadPosition(
        getRandomInt(-5, 5), // Random pitch position between -5 and 5
        getRandomInt(-5, 5), // Random roll position between -5 and 5
        getRandomInt(-5, 5), // Random yaw position between -5 and 5
        100); // Head movement velocity. Decrease for slower movement.

        // If repeat is set to true, re-registers for the look_around
        // timer event, and Misty moves her head until the skill ends.
        if (repeat) misty.RegisterTimerEvent(
        "look_around",
        getRandomInt(5, 10) * 1000,
        false);
}
```

The next step is to call the `misty.Debug()` method at the beginning of your skill code with a message to indicate the skill is starting. The `misty.Debug()` method prints messages to debug listeners, and you can use it to locate issues as you develop Misty's skills.

Copy the following into the beginning of your skill code:

```JavaScript
// Sends a message to debug listeners
misty.Debug("The HelloWorld skill is starting!")
```

The code file is complete! At this point, the contents of your `HelloWorld.js` file should look look like this:

```JavaScript
// Sends a message to debug listeners
misty.Debug("The HelloWorld skill is starting!")

// Returns a random integer between min and max
function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

// The look_around timer event invokes this callback function. Change
// the value of repeat to false if Misty should only move her head once.
function _look_around(repeat = true) {

    // Moves Misty's head to a random position. Adjust the min/max
    // values passed into getRandomInt() to change Misty's range of
    // motion when she calls this method.
    misty.MoveHeadPosition(
        getRandomInt(-5, 5), // Random pitch position between -5 and 5
        getRandomInt(-5, 5), // Random roll position between -5 and 5
        getRandomInt(-5, 5), // Random yaw position between -5 and 5
        100); // Head movement velocity. Decrease for slower movement.

        // If repeat is set to true, re-registers for the look_around
        // timer event, and Misty moves her head until the skill ends.
        if (repeat) misty.RegisterTimerEvent(
        "look_around",
        getRandomInt(5, 10) * 1000,
        false);
}

// Registers for a timer event with called look_around, and invokes the
// _look_around() callback after 5000 - 10000 milliseconds.
misty.RegisterTimerEvent("look_around", getRandomInt(5, 10) * 1000, false);
```

### Generating the Meta File

Next we create the JSON meta file and install the skill on Misty. Follow these steps to generate a JSON file for your Hello World skill:

1. Open the [Skill Runner](http://sdk.mistyrobotics.com/skill-runner/) web page in a new browser window and find the **Generate** section. ![Skill Runner generate meta](../../../assets/images/skill-runner-generate.png)
2. Type `HelloWorld` in the **New Skill Name** field.
3. Select the option to **Download** option and click **Generate JSON Meta Template**.
4. Locate the downloaded `HelloWorld.json` file save it to the `HelloWorld` directory you created for this skill earlier.
5. Open the `HelloWorld.json` file in your text editor. It should look something like this, with a different value for the `"UniqueID"` key:

```JSON
{
    "Name": "HelloWorld",
    "UniqueId": "f1a3b79a-4942-4133-9e4c-92aa9643c378",
    "Description": "My skill is amazing!",
    "StartupRules": [
        "Manual",
        "Robot"
    ],
    "Language": "javascript",
    "BroadcastMode": "verbose",
    "TimeoutInSeconds": 600,
    "CleanupOnCancel": false,
    "WriteToLog": false,
    "Parameters": {
        "int": 10,
        "double": 20.5,
        "string": "twenty"
    }
}
```

Make sure the value of the `"Name"` parameter is `"HelloWorld"`. You can ignore the rest of the parameters for now.

{{box op="start" cssClass="boxed noteBox"}}
**Note:** value of `"UniqueId"` is randomized for each meta file the Skill Runner generates, and should be unique for each skill on your robot. The `"TimeOutInSeconds"` parameter describes how long the skill runs (in seconds) before Misty automatically cancels it.
{{box op="end"}}

### Installing the Hello World Skill

With the meta file saved, you're ready to install your skill on Misty. Follow these steps to do so:

1. In the [Skill Runner](http://sdk.mistyrobotics.com/skill-runner/) web page, type Misty’s IP address into the Robot IP Address field in the upper right hand corner. (You can find Misty's IP address in the Misty App.) Click **Connect**.
2. Once the connection is established, find the **Install** section on the Skill Runner page. Click **Choose files** and navigate to the `HelloWorld` directory where you saved the `HelloWorld.js` and `HelloWorld.json` files. Select both files and click **Open**.
3. The skill takes a few seconds to upload. When the upload is complete, your new `HelloWorld` skill appears under the **Manage** section of the Skill Runner page. Find it and click **Start** to begin execution!

{{box op="start" cssClass="boxed noteBox"}}
**Note:** The Skill Runner web page prints additional information from the skills running on Misty (including debug messages) to your browser's web console. To open the web console in Chrome, use **Ctrl + Shift + J** (Windows/Linux) or **Cmd + Option + J** (Mac).
{{box op="end"}}

## Changing Misty's LED

This part of the Hello World tutorial series teaches how to fade Misty's chest LED fade on and off in a looping cycle. If you've already completed the [first part of this tutorial series](./#moving-misty-39-s-head), you can add this code to your original `HelloWorld.js` code file, beneath where you wrote the code for moving Misty's head. 

{{box op="start" cssClass="boxed tipBox"}}
**Tip:** While we recommend completing the Hello World Tutorial Series in order, the code you write in this section runs just fine without the code from earlier parts of the series. Just edit your code in a new JavaScript file and [generate a new JSON meta file](./#generating-the-meta-file) with the same skill name and filename.
{{box op="end"}}

When we fade Misty's chest LED on and off, the visual effect is such that Misty appears to be "breathing". To achieve this in your skill code, start by calling the [`misty.RegisterTimerEvent()`](../../../misty-ii/reference/javascript-api/#misty-registertimerevent-alpha) method. Use `"breathingLED"` as the name of the timer event, and pass in a value of `1` for the `callbackTimeInMs` argument. Copy this method into your `HelloWorld.js` skill file, beneath where you wrote the code for moving Misty's head.

```JavaScript
// Registers for a timer event called breathingLED, and invokes the
// _breathingLED() callback after 1 millisecond.
misty.RegisterTimerEvent("breathingLED", 1, false);
```

By default, timer events you create with the `misty.RegisterTimerEvent()` method invoke a callback function with the same `eventName`, prefixed with an underscore. In this case, the callback function is called `_breathingLED()`. Declare this function in your skill code:

```JavaScript
// The breathingLED timer event invokes this callback function.
function _breathingLED() {

}
```

The first thing this callback function does is create variables to store the starting RGB values for the color of Misty's chest LED. We modify these these values in `for` loops to rapidly phase through a series of different colors. Copy the following into your `_breathingLED()` callback function:

```JavaScript
// The breathingLED timer event invokes this callback function.
function _breathingLED() {

    // Values used to modify the RGB intensity of Misty's chest LED.
    // Change these to use a different starting color for the LED.
    var red = 140 / 10.0;
    var green = 0 / 10.0;
    var blue = 220 / 10.0;

}
```

Next, we create the two `for` loops that cause Misty's chest LED to "breathe". The first loop uses the values stored in the `red`, `green`, and `blue` variables to send a sequence of [`misty.ChangeLED()`](../../../misty-ii/reference/javascript-api/#misty-changeled) commands and incrementally decrease intensity of each color in the LED. The second `for` loop does the inverse and slowly fades the LED back to its original color. The syntax of the `misty.ChangeLED()` method is as follows:

```JavaScript
// Syntax
misty.ChangeLED(int red, int green, int blue, [int prePauseMs], [int postPauseMs]);
```

Each iteration of the `for` loops in the `_breathingLED()` callback should send a `misty.changeLED()` command with slightly altered values for the `red`, `green`, and `blue` arguments. We use the built-in `Math.floor()` JavaScript method to help with this. We also call the [`misty.Pause()`](../../../misty-ii/reference/javascript-api/#misty-pause-alpha) method to briefly pause execution between iterations. Increasing the pause time decreases breathing speed, and decreasing pause time makes Misty breath faster. 

Copy these `for` loops into your `_breathingLED()` callback function:

```JavaScript
// The breathingLED timer event invokes this callback function.
function _breathingLED(){

    // Values used to modify the RGB intensity of Misty's chest LED.
    // Change these to use a different starting color for the LED.
    var red = 140 / 10.0;
    var green = 0 / 10.0;
    var blue = 220 / 10.0;

    // Incrementally DECREASES the intensity of each color in the LED
    for (var i = 10; i >= 0; i = i - 1) {
        misty.ChangeLED(
            Math.floor(i * red), // red intensity
            Math.floor(i * green), // green intensity
            Math.floor(i * blue)); // red intensity
        // Pause before next iteration. Increase value for slower
        // breathing; decrease for faster breathing.
        misty.Pause(150);
    }
    
    // Incrementally INCREASES the intensity of each color in the LED
    for (var i = 0; i <= 10; i = i + 1) {
        misty.ChangeLED(
            Math.floor(i * red), // red intensity
            Math.floor(i * green), // green intensity
            Math.floor(i * blue)); // blue intensity
        // Pause before next iteration. Increase value for slower
        // breathing; decrease for faster breathing.
        misty.Pause(150);
    }
}
```

The `_breathingLED()` callback only runs a single "breathing" cycle, but we want Misty's LED to continue breathing until the Hello World skill ends. To do this, you can call the `misty.RegisterTimerEvent()` method within the callback function to re-register for `breathingLED` events. Copy this into the end of the `_breathingLED()` callback function:

```JavaScript
    // Re-registers for the breathingLED timer event, so Misty's LED
    // continues breathing until the skill ends.
    misty.RegisterTimerEvent("breathingLED", 1, false);
```

With the callback function complete, you're ready to upload your changes to Misty. The completed code for changing Misty's LED should look like this:

```JavaScript
// The breathingLED timer event invokes this callback function.
function _breathingLED() {
    // Values used to modify the RGB intensity of Misty's chest LED.
    // Change these to use a different starting color for the LED.
    var red = 140 / 10.0;
    var green = 0 / 10.0;
    var blue = 220 / 10.0;

    // Incrementally DECREASES the intensity of each color in the LED
    for (var i = 10; i >= 0; i = i - 1) {
        misty.ChangeLED(
            Math.floor(i * red), // red intensity
            Math.floor(i * green), // green intensity
            Math.floor(i * blue)); // red intensity
        // Pause before next iteration. Increase value for slower
        // breathing; decrease for faster breathing.
        misty.Pause(150);
    }

    // Incrementally INCREASES the intensity of each color in the LED
    for (var i = 0; i <= 10; i = i + 1) {
        misty.ChangeLED(
            Math.floor(i * red), // red intensity
            Math.floor(i * green), // green intensity
            Math.floor(i * blue)); // blue intensity
        // Pause before next iteration. Increase value for slower
        // breathing; decrease for faster breathing.
        misty.Pause(150);
    }
    // Re-registers for the breathingLED timer event, so Misty's LED
    // continues breathing until the skill ends.
    misty.RegisterTimerEvent("breathingLED", 1, false);
}

// Registers for a timer event called breathingLED, and invokes the
// _breathingLED() callback after 1 millisecond.
misty.RegisterTimerEvent("breathingLED", 1, false);
```

Save your changes. Use the Skill Runner to [upload your modified `HelloWorld.js` file to Misty](./#installing-the-hello-world-skill).

{{box op="start" cssClass="boxed noteBox"}}
**Note:** When you upload a JavaScript code file with the same name as a skill that already exists on Misty, Misty overwrites the existing file with your new code. If you did not change the name of the JavaScript code file for your Hello World skill, then there's no need to upload a new JSON meta file to Misty.
{{box op="end"}}

When the upload is complete, run the skill from the **Manage** section of the Skill Runner web page.

## Playing Sounds

This part of the Hello World tutorial series teaches how to write code to have Misty play sounds. If you started at the beginning of the Hello World series, you can add this code to the same `HelloWorld.js` code file, beneath where you wrote the code for changing Misty's LED.

{{box op="start" cssClass="boxed tipBox"}}
**Tip:** While we recommend completing the Hello World Tutorial Series in order, the code you write in this section runs just fine without the code from earlier parts of the series. Just edit your code in a new JavaScript file and [generate a new JSON meta file](./#generating-the-meta-file) with the same skill name and filename.
{{box op="end"}}

When we bring Misty to life, she should greet the world with a robot sound of her own. Misty comes with several default system audio files, which you can play in your skills by calling the [`misty.PlayAudio()`](../../../misty-ii/reference/javascript-api/#misty-playaudio) method. The syntax for this method is as follows:

```JavaScript
// Syntax
misty.PlayAudio(string fileName, [int volume], [int prePauseMs], [int postPauseMs]);
```

In our Hello World skill, Misty plays the `"001-EeeeeeE.wav"` file at 100% of max volume. This is the same file that Misty plays when she boots up. After she plays the sound, we use the [`misty.Pause()`](../../../misty-ii/reference/javascript-api/#misty-pause-alpha) method to have Misty wait a few seconds before executing the next command.

Copy the following into your `HelloWorld.js` code file:

```JavaScript
// Plays an audio file at max volume.
misty.PlayAudio("001-EeeeeeE.wav", 100);
// Pauses for 3000 milliseconds before executing the next command.
misty.Pause(3000);
```

You can have Misty play a different sound by passing in a different `fileName` when you call the `misty.PlayAudio()` method. You can use the Command Center to see the names of all Misty's default audio files.

Save your changes and use the Skill Runner to [upload your modified `HelloWorld.js` file to Misty](./#installing-the-hello-world-skill). Start the skill to hear Misty play the sound.

## Driving Misty

This part of the Hello World tutorial series teaches how to programmatically drive Misty. When Misty executes the code from this section of the series, she slowly turns left and then right for a better view of her new home. If you started at the beginning of the Hello World series, you can add this code to the same `HelloWorld.js` code file you've been working on, beneath where you wrote the code for playing audio.

{{box op="start" cssClass="boxed tipBox"}}
**Tip:** While we recommend completing the Hello World Tutorial Series in order, the code you write in this section runs just fine without the code from earlier parts of the series. Just edit your code in a new JavaScript file and [generate a new JSON meta file](./#generating-the-meta-file) with the same skill name and filename.
{{box op="end"}}

{{box op="start" cssClass="boxed warningBox"}}
**Important!** The code you write in this section of the tutorial activates Misty's drive motors. Make sure to place Misty on the ground before you run this code. **DO NOT** run this code while Misty is set on a high surface. Always set Misty on the foam block that arrived in her carrying case when working with her on a desk or table.
{{box op="end"}}

{{box op="start" cssClass="boxed noteBox"}}
**Note:** If Misty receives a [`misty.DriveTime()`](../../../misty-ii/reference/javascript-api/#misty-drivetime) command while her treads are elevated (for example, while she's sitting on her foam block stand), she perceives her lack of acceleration as a failure to supply adequate power to her motors. When this happens, the drive motors accelerate until they reach maximum velocity, and they continue to receive power until the time passed in for the `duration` argument has elapsed.
{{box op="end"}}

To drive Misty in the Hello World tutorial, we use the [`misty.DriveTime()`](../../../misty-ii/reference/javascript-api/#misty-drivetime) method. This method drives Misty forward or backward at a set speed, with a given rotation, for a specified amount of time (in milliseconds). The syntax is as follows:

```JavaScript
// Syntax
misty.DriveTime(double linearVelocity, double angularVelocity, int timeMs, [double degree], [int prePauseMs], [int postPauseMs]);
```

{{box op="start" cssClass="boxed tipBox"}}
**Tip:** The `misty.DriveTime()` argument expects the values used for `linearVelocity` and `angularVelocity` to be a percentage Misty's maximum velocity. When using the `misty.DriveTime()` method, it helps to understand how linear velocity (speed in a straight line) and angular velocity (speed and direction of rotation) work together:

* If linear velocity is -100 and angular velocity is 0, Misty drives straight backward at full speed.
* If linear velocity is 100 and angular velocity is 0, Misty drives straight forward at full speed.
* If linear velocity is 0 and angular velocity is -100, Misty rotates clockwise at full speed.
* If linear velocity is 0 and angular velocity is 100, Misty rotates counter-clockwise at full speed.
* If linear velocity is not zero and angular velocity is not zero, Misty drives in a curve.
{{box op="end"}}

In our Hello World skill, we want Misty to turn slowly counter-clockwise, pause, then turn slowly clockwise. We do this with a sequence of `misty.DriveTime()` commands. We insert [`misty.Pause()`](../../../misty-ii/reference/javascript-api/#misty-pause-alpha) commands after each call to `misty.DriveTime()` to prevent the next drive command from overriding the previous drive command before Misty has finished driving. We call the [`misty.Stop()`](../../../misty-ii/reference/javascript-api/#misty-stop) method to ensure Misty's drive motors stop at the end of the sequence.

Copy the following lines of code into your `HelloWorld.js` skill file, beneath where you wrote the code to play audio:

```JavaScript
misty.DriveTime(0, 30, 5000); // Turns Misty to her left
misty.Pause(5000); // Wait for turn to complete
misty.DriveTime(0, -30, 5000); // Turns Misty to her right
misty.Pause(5000); // Wait for turns to complete
misty.Stop(); // Stops driving motors
```

To change the speed and distance that Misty turns, modify the `angularVelocity` and `timeMs` values in your `misty.DriveTime()` commands. Pass in a positive value for `linearVelocity` to have Misty move forward, and use a negative value to have Misty move backward. We suggest starting at low speeds, like `30`/`-30`, and slowly incrementing these values to get a feel for how they affect Misty's movement.

You're now ready to update your skill. Save your changes, and use the Skill Runner to [upload your modified `HelloWorld.js` file to Misty](./#installing-the-hello-world-skill). **Remove Misty from her foam block and set her on the floor, with both her left and right treads firmly on the ground**. Start the skill to see Misty drive.

## Teaching Misty to Wave

This part of the Hello World tutorial series teaches how to programmatically move Misty's arms. When the code you write in this section executes, Misty raises her right arm and waves. If you started at the beginning of the Hello World series, you can add this code to the same `HelloWorld.js` code file you've been working on, beneath where you wrote the code for driving Misty.

{{box op="start" cssClass="boxed tipBox"}}
**Tip:** While we recommend completing the Hello World Tutorial Series in order, the code you write in this section runs just fine without the code from earlier parts of the series. Just edit your code in a new JavaScript file and [generate a new JSON meta file](./#generating-the-meta-file) with the same skill name and filename.
{{box op="end"}}

In this tutorial, we use the [`misty.MoveArmDegrees()`](../../../misty-ii/reference/javascript-api/#misty-movearmDegrees) method to have Misty move her arms. This method controls which arm should move, the angle (`90` - fully down, and `-45` - angled up) that the arm should move to, and how quickly the arm should move. The syntax for the `misty.MoveArmDegrees()` method is as follows:

```JavaScript
// Syntax
misty.MoveArmDegrees(string arm, double degrees, double velocity, [int prePauseMs], [int postPauseMs])
```

To have Misty wave, we start by setting the angle of both arms to `90`, down by Misty's sides. We then send a command to move Misty's right arm up (wave), then pause before returning the arm to Misty's side. We wrap all this in a function called `waveRightArm()`, so we can use it later in the skill. 

Copy the following into your `HelloWorld.js` skill code:

```JavaScript
// Waves Misty's right arm!
function waveRightArm() {
    misty.MoveArmDegrees("left", 90, 45); // Left arm fully down
    misty.Pause(50);
    misty.MoveArmDegrees("right", 90, 45); // Right arm fully down
    misty.Pause(3000); // Pause for 3 seconds
    misty.MoveArmDegrees("right", -45, 45); // Right arm up
    misty.Pause(5000); // Pause with arm up for 5 seconds (wave!)
    misty.MoveArmDegrees("right", 90, 45); // Right arm fully down
}

waveRightArm();
```

You're now ready to update your skill. Save your changes, and use the Skill Runner to [upload your modified `HelloWorld.js` file to Misty](./#installing-the-hello-world-skill). **Remove Misty from her foam block and set her on the floor, with both her treads firmly on the ground**. Start the skill to see Misty wave!

## Using Face Recognition

This part of the Hello World tutorial series teaches how to use face recognition data in your skill code. When the code you write in this section executes, Misty attempts to detect and recognize faces. If you've trained Misty on your own face, then Misty waves when she sees you. If Misty sees a person she doesn't know, she raises her eyebrows and plays a sound.

{{box op="start" cssClass="boxed noteBox"}}
**Note:** If you haven't already trained Misty to recognize your face, [use the Command Center to do so](../../../tools-&-apps/web-based-tools/command-center/#face-training-amp-recognition) before working through this section of the tutorial.
{{box op="end"}}

If you started at the beginning of the Hello World series, you can add this code to the same `HelloWorld.js` code file you've been working on, beneath where you wrote the code that taught Misty to wave.

To begin, declare a function called `_registerFaceRed()`. We'll wrap the commands to start Misty looking for faces inside this function.

```JavaScript
// Invoke this function to start Misty recognizing faces.
function _registerFaceRec() {

}
```

We call the [`misty.StartFaceRecognition()`](../../../misty-ii/reference/javascript-api/#misty-startfacerecognition) method to start Misty's face recognition process. Copy this function (along with a call to [`misty.StopFaceRecognition()`](../../../misty-ii/reference/javascript-api/#misty-stopfacerecognition), to cancel any face recognition that's already underway) into your `_registerFaceRec()` function.

```JavaScript
// Invoke this function to start Misty recognizing faces.
function _registerFaceRec() {
    // Cancels any face recognition that's currently underway
    misty.StopFaceRecognition();
    // Starts face recognition
    misty.StartFaceRecognition();
}
```

We can subscribe to data from Misty's face recognition system by registering for events from the [`FaceRecognition`](../../../misty-ii/reference/sensor-data/#facerecognition) named object. We use the [`misty.RegisterEvent`](../../../misty-ii/reference/javascript-api/#misty-registerevent-alpha) method to do this. This method invokes a callback function each time the registered event occurs. The syntax for `misty.RegisterEvent` is as follows:

```JavaScript
// Syntax
misty.RegisterEvent(string eventName, string messageType, int debounce, [bool keepAlive = false], [string callbackRule = “synchronous”], [string skillToCall = null]);
```

Copy this method into your `_registerFaceRec()` function. Use `FaceRec` for `eventName`, `FaceRecognition` for `messageType`, and `1000` for `debounce`. Set `keepAlive` to `false`.

```JavaScript
// Invoke this function to start Misty recognizing faces.
function _registerFaceRec() {
    // Cancels any face recognition that's currently underway
    misty.StopFaceRecognition();
    // Starts face recognition
    misty.StartFaceRecognition();

    // Registers for FaceRecognition events. Sets eventName to FaceRec,
    // debounceMs to 1000, and keepAlive to false.
    misty.RegisterEvent("FaceRec", "FaceRecognition", 1000, false);
}
```

By default, when the face recognition system sends an event, data from that event is passed into a callback function with the same name, prefixed by an underscore (in our case, `_FaceRec()`). The face recognition system can send events that do not include meaningful face recognition data - for example, the system returns a notification message without any event data each time a skill successfully registers for an event. To ignore these messages (and invoke the callback function ONLY when meaningful data is present), we pass the data through a property test.

The [`misty.AddPropertyTest()`](../../../misty-ii/reference/javascript-api/#misty-addpropertytest-alpha) method checks event data against a custom property test, so you can control when the callback function should be invoked. The syntax for the `misty.AddPropertyTest()` is as follows:

```JavaScript
// Syntax
misty.AddPropertyTest(string eventName, string property, string inequality, string valueAsString, string valueType, [int prePauseMs], [int postPauseMs]);
```

In our skill, Misty should only invoke the `_FaceRec()` callback when she has actually detected a face. We can use the `misty.AddPropertyTest()` method to check the event data and make sure there is a value for the `PersonName` property before triggering the callback function. To do so, pass in `"PersonName"` as the `property` to check for, and `"exists"` as the inequality to use. Copy this into your `_registerFaceRec()` function.

```JavaScript
// Invoke this function to start Misty recognizing faces.
function _registerFaceRec() {
    // Cancels any face recognition that's currently underway
    misty.StopFaceRecognition();
    // Starts face recognition
    misty.StartFaceRecognition();
    // If a FaceRecognition event includes a "PersonName" property,
    // then Misty invokes the _FaceRec callback function.
    misty.AddPropertyTest("FaceRec", "PersonName", "exists", "", "string");
    // Registers for FaceRecognition events. Sets eventName to FaceRec,
    // debounceMs to 1000, and keepAlive to false.
    misty.RegisterEvent("FaceRec", "FaceRecognition", 1000, false);
}
```

Now we can define the `_FaceRec()` callback function. Each time a `FaceRec` event passes our property test, that event data is passed into the `_FaceRec()` callback. Define this function in your skill code.

```JavaScript
// FaceRec events invoke this callback function.
function _FaceRec(data) {

}
```

We place the code that describes how Misty should react when she sees a face inside this `_FaceRec()` callback function. First, let's print the value of the `"PersonName"` property to [`misty.Debug()`](../../../misty-ii/reference/javascript-api/#misty-debug-alpha) listeners, so we can compare Misty's reaction to the data she receives. Because we used a property test to check for the `"PersonName"` property, we can find the value of that property in the `data` object the callback function receives, at `data.PropertyTestResults[0].PropertyValue`.

Copy the following code into your `_FaceRec()` callback function:

```JavaScript
/ FaceRec events invoke this callback function.
function _FaceRec(data) {
    // Stores the value of the detected face
    var faceDetected = data.PropertyTestResults[0].PropertyValue;
    // Logs a debug message with the label of the detected face
    misty.Debug("Misty sees " + faceDetected);
}
```

Next, we define how Misty should react when she recognizes your face. We can use the `waveRightArm()` function we wrote [in an earlier part of this tutorial](./#teaching-misty-to-wave) series to have Misty wave to you. We can use the [`misty.DisplayImage()`](../../../misty-ii/reference/javascript-api/#misty-displayimage) and [`misty.PlayAudio()`](../../../misty-ii/reference/javascript-api/#misty-playaudio) methods to have Misty show her happy eyes and play a greeting sound.

Let's package this code inside an `if` statement that runs when the value of the `faceDetected` variable is equal to your name:

```JavaScript
/ FaceRec events invoke this callback function.
function _FaceRec(data) {
    // Stores the value of the detected face
    var faceDetected = data.PropertyTestResults[0].PropertyValue;
    // Logs a debug message with the label of the detected face
    misty.Debug("Misty sees " + faceDetected);

    // Use the Command Center to train Misty to recognize your face.
    // Then, replace <Your-Name> below with your own name! If Misty
    // sees and recognizes you, she waves and looks happy.
    if (faceDetected == "<Your-Name>") {
        misty.DisplayImage("Happy.png");
        misty.PlayAudio("005-Eurra.wav");
        waveRightArm();
    }
}
```

If Misty sees someone she doesn't know, use the `misty.DisplayImage()` and `misty.PlayAudio()` methods to have her raise an eyebrow and play a confused sound. Copy this code into your `_FaceRec()` callback function.

```JavaScript
// FaceRec events invoke this callback function.
function _FaceRec(data) {
    // Stores the value of the detected face
    var faceDetected = data.PropertyTestResults[0].PropertyValue;
    // Logs a debug message with the label of the detected face
    misty.Debug("Misty sees " + faceDetected);

    // Use the Command Center to train Misty to recognize your face.
    // Then, replace <Your-Name> below with your own name! If Misty
    // sees and recognizes you, she waves and looks happy.
    if (faceDetected == "<Your-Name>") {
        misty.DisplayImage("Happy.png");
        misty.PlayAudio("005-Eurra.wav");
        waveRightArm();
    }
    // If misty sees someone she doesn't know, she raises her eyebrow
    // and plays a different sound.
    else if (faceDetected == "unknown person") {
        misty.DisplayImage("Disdainful.png");
        misty.PlayAudio("001-OooOooo.wav");
    };

    // Register for a timer event to invoke the _registerFaceRec
    // callback function loop through the _registerFaceRec() again
    // after 7000 milliseconds pass.
    misty.RegisterTimerEvent("registerFaceRec", 7000, false);
}
```

At the end of the `_FaceRec()` function, call the [`misty.RegisterTimerEvent()`](../../../misty-ii/reference/javascript-api/#misty-registertimerevent-alpha) method to trigger the `_registerFaceRec()` function after 7000 milliseconds. This ensures that Misty will continue looking for faces, and will greet whomever she sees until the skill ends.

The full `_FaceRec()` function should look like this:

```JavaScript
// Invoke this function to start Misty recognizing faces.
function _registerFaceRec() {
    // Cancels any face recognition that's currently underway
    misty.StopFaceRecognition();
    // Starts face recognition
    misty.StartFaceRecognition();
    // If a FaceRecognition event includes a "PersonName" property,
    // then Misty invokes the _FaceRec callback function.
    misty.AddPropertyTest("FaceRec", "PersonName", "exists", "", "string");
    // Registers for FaceRecognition events. Sets eventName to FaceRec,
    // debounceMs to 1000, and keepAlive to false.
    misty.RegisterEvent("FaceRec", "FaceRecognition", 1000, false);
}

// FaceRec events invoke this callback function.
function _FaceRec(data) {
    // Stores the value of the detected face
    var faceDetected = data.PropertyTestResults[0].PropertyValue;
    // Logs a debug message with the label of the detected face
    misty.Debug("Misty sees " + faceDetected);

    // Use the Command Center to train Misty to recognize your face.
    // Then, replace <Your-Name> below with your own name! If Misty
    // sees and recognizes you, she waves and looks happy.
    if (faceDetected == "<Your-Name>") {
        misty.DisplayImage("Happy.png");
        misty.PlayAudio("005-Eurra.wav");
        waveRightArm();
    }
    // If misty sees someone she doesn't know, she raises her eyebrow
    // and plays a different sound.
    else if (faceDetected == "unknown person") {
        misty.DisplayImage("Disdainful.png");
        misty.PlayAudio("001-OooOooo.wav");
    };

    // Register for a timer event to invoke the _registerFaceRec
    // callback function loop through the _registerFaceRec() again
    // after 7000 milliseconds pass.
    misty.RegisterTimerEvent("registerFaceRec", 7000, false);
}
```

We're almost finished! At the end of your skill code, call the `_registerFaceRec()` function to start Misty recognizing faces and kick off the loop.

```JavaScript
// Starts Misty recognizing faces!
_registerFaceRec();
```

You're now ready to update your skill. Save your changes, and the Skill Runner to [upload your modified `HelloWorld.js` file to Misty](./#installing-the-hello-world-skill). **Remove Misty from her foam block and set her on the floor, with both her treads firmly on the ground**.

When you run the full Hello World skill, Misty starts looking for faces after she waves for the first time.

{{box op="start" cssClass="boxed noteBox"}}
**Note:** Misty's face recognition works best in well-lit environments, and she recognizes faces best when they are directly in front of her visor at a range of closer than about six feet. Because Misty is likely to be on the floor when you run this skill, you may need to kneel down to get within range of her visor. You may also need to wait for Misty to turn her head to look at you, as the head movement commands created in the first part of this series continue to execute until the end of the skill.
{{box op="end"}}

## What's Next?

After you finish the Hello World tutorial series, try customizing the skill to make it your own. Here are a few ideas:
* Consider teaching Misty new faces, and adjusting the face recognition callback to have her greet different individuals in a unique way.
* Try modifying the code from the Changing Misty's LED section to use a different color for the chest LED.
* Experiment with different driving commands to have Misty drive around while she runs your skill.
* In the JSON meta file, increase the value of the `"TimeoutInSeconds"` key to keep the skill running longer.

The elements covered in this tutorial series just scratch the surface of what's possible with Misty. When you're ready, [read our introduction to coding Misty](../../../misty-ii/coding-misty/introduction) to learn more about all the different ways she can bring your code to life.