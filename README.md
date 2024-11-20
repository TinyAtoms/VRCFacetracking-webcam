
# VRChat Face Tracking Module Documentation

This program utilizes your webcam for face tracking, enabling eye and mouth tracking capabilities. It does not require a phone, allowing for a straightforward and convenient setup for VRChat users.

# Prerequisites
To get started with the VRChat Face Tracking Module, make sure you have the following:

1. **Python Installation**: Ensure that you have Python installed on your computer. If not, install from [here](https://www.python.org/downloads/windows/)
2. **Install Dependencies**: You’ll need to install the required dependencies. Open a command prompt and run:
   ```
   pip install -r .\requirements.txt
   ```
3. **VRC Face Tracking Module**: You can download the VRC Face Tracking module from [this link](https://github.com/TinyAtoms/VRCFaceTracking-MPmodule).


# Running the Program
Follow these steps to run the face tracking program:
1. **Clone this repository**: `git clone https://github.com/TinyAtoms/VRCFacetracking-webcam.git`
2. **Start VRCFT**: Launch the VRC Face Tracking (VRCFT) application.
3. **Run the Script**: Execute the following command in your terminal or command prompt:
   ```
   python face_webcam.py
   ```
4. if you don't see a camera feed in the window, change the camera device ID. Set it to 1,2 etc, click "change" and restart the script.
4. **Adjust Parameters**: Feel free to tweak the parameters in the script as needed for your setup.

# Tuning Parameters
Here are some guidelines to help you tune the performance. This may or may not be needed, as the current parameters have been set to my particular setup, which may be fine for most people.

1. **Mirror Feedback**: Stand in front of a VRChat mirror while the module is active. This will help you see the effects of your adjustments in real-time. The values sent to VRCFT are calculated as `offset + multiplier * raw_value`[^1].
2. Click "Show Tuning Options" to reveal the interface where you can change offsets and multipliers
3. **Reference Materials**: It’s helpful to have [this reference page](https://docs.vrcft.io/docs/tutorial-avatars/tutorial-avatars-extras/unified-blendshapes) open. It provides detailed explanations of each parameter's function.
4. **Testing Expressions**: Create different facial expressions and observe if your avatar reflects those expressions with similar intensity. For instance, check if fully opening your jaw results in your avatar’s jaw fully extending. If the movements don’t align, adjust the offset and multiplier values accordingly.
5. **Save Your Settings**: Remember to save your changes so that you don’t have to reconfigure everything the next time you use the module.


## Advanced Tuning Parameters
Instead of a simple offset and multiplier, you can provide your own custom function to process a particular parameter. To do that, you open the `param_tuning.jsonc` file and add a name to the `cfunct` option of the parameter which should  have a custom function Open the face_webcam.py file and add a method to the BlendshapeParams class with the same name you added 
For example, here is a custom function applied to EyeOpennessLeft/Right:

```
 "EyeOpennessLeft": {"cfunct": "eye_openness_left", "max": 1, "min": 0, "mul": 3.5, "offset": 3},
 "EyeOpennessRight": {"cfunct": "eye_openess_right", "max": 1, "min": 0, "mul": 3.5, "offset": 3},
```

and in the python file
```
class BlendShapeParams:
...
    def eye_openness_left(self, value: float) -> float:
        """
        Custom function for EyeOpennessLeft processing.
        instead of offset + mul* value, it uses mul* pow(value, 1/offset)
        which is a vaguely sigmoidal function
        """
        a = self.mul("EyeOpennessLeft")
        b = self.offset("EyeOpennessLeft")

        return self.bound(
            self.min("EyeOpennessLeft"),
            self.max("EyeOpennessLeft"),
            a * math.pow(value, 1 / b),
        )

    def eye_openess_right(self, value: float) -> float:
        """
        Custom function for EyeOpennessLeft processing.
        instead of offset + mul* value, it uses mul* pow(value, 1/offset)
        which is a vaguely sigmoidal function
        """
        a = self.mul("EyeOpennessRight")
        b = self.offset("EyeOpennessRight")
        return self.bound(
            self.min("EyeOpennessRight"),
            self.max("EyeOpennessRight"),
            a * math.pow(value, 1 / b),
        )
```

To disable the current default custom function, change EyeOpennessLeft/Right to something like this
```
 "EyeOpennessLeft": {"cfunct": "", "max": 1, "min": 0, "mul": 4, "offset": 0},
 "EyeOpennessRight": {"cfunct": "", "max": 1, "min": 0, "mul": 4, "offset": 0},
```


# Known Issues
1. **Closing VRCFT**: Please note that closing VRCFT will also terminate this script.
2. **Eye Tracking openess**:  Tracking how open your eyes are is kind of bad for people wearing glasses. The default tracking option is better suited for people with glasses, and as a result has trouble fully closing the eyes. If you don't wear glasses, see the Advanced parameter tuning section on how to change this behaviour.

3. **Eye Gaze tracking**: Due to how the parameters are processed, the directions where your eye looks might be a bit jittery when staring directly ahead.


[^1]: The calculation is based on an exponential moving average of recent frames, with minor movements filtered out for some parameters. However, these details are not currently adjustable, so that's not really relevant.