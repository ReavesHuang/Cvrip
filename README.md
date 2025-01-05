# A Visual GUI Ripping Framework - Cvrip
## 🔆 Introduction for Cvrip

We introduce cvrip, a visual GUI ripping framework, to make the automated exploration. Cvrip visually analyzes the GUI screen for ripping and does not rely on the underlying GUI frameworks. We introduce many new techniques to enable efficient visual GUI ripping, e.g., a YOLO v5 based model to detect executable widgets, a state recognition acceleration method for fast model updating, and several GUI exploration strategies taking the characteristics of imperfect visual analysis into account.

## 💡 Downloads<a name="tips"></a>

Here is the link of cvrip: https://pan.nuaa.edu.cn/share/e880b7da655709421057487e3a.

### 📝 1. Requirements
- Python：python 3.6-3.8

  ```shell
  pip install -r requirements.txt
  ```

- CUDA support

  - Without CUDA support, cvrip may run very slow

    - YOLO:  based on pytorch

    - resnet: based on pytorch

    - UIED: based on tensflow

    - vgg: based on tensflow

    - paddleocr: based on paddlepaddle

      The installed paddlepaddle-gpu and CUDA versions shall be compatible

      - The paddlepaddle-gpu installed in default is only compatible with Cuda 10.x
        
      	```
      	python -m pip install paddlepaddle-gpu==2.2.2 -i https://mirror.baidu.com/pypi/simple
      	```
       
      - For Cuda 11.1, paddlepaddle-gpu shall be installed in the following way
        
      	```
  	pip install paddlepaddle-gpu==2.2.2.post111 -f https://www.paddlepaddle.org.cn/whl/windows/mkl/avx/stable.html
      	```
  
- AI models

  - models/yolo_v5_best.pt  
  - cv/yolov3/yolov3_ckpt.pth      

## 🚀 Quick Start
## 2. How to Run

### 2.1 Install *adb*

- Install AndroidStudio and the Android SDK (Android SDK Tools, Android SDK Platform-tools, Android SDK Build-tools)

  Ref.：https://developer.android.com/studio/releases/platform-tools?hl=zh-cn

- Put the adb binary into the environment variable `PATH`

### 2.2 Start the real or virtual device

An AVD/bluestack virtual machine is also possible.

The device has an ID like `127.0.0.1:5555` or `emulator-5554`.

### 2.3 Install the subject app

    # connect the device
    adb connect 127.0.0.1:7555
    # install the app
    adb install demo\android\com.ddnapalon.calculator.gp.apk

### 2.4 Get the app parameters

- Get the app `package` and `activity`

	Start the app on the device. Run `adb shell dumpsys window | findstr mCurrentFocus`, there will be a string like the following in the result.

	 `mCurrentFocus=Window{e660c1b u0 com.ddnapalon.calculator.gp/com.ddnapalon.calculator.gp.ScienceFragment}`

	The package name is  `com.ddnapalon.calculator.gp`，and the activity is：`com.ddnapalon.calculator.gp.ScienceFragment`

- Get the app window  (exclude the notification and status bars on the screen)

  Run `shell dumpsys window displays`. In the result, `rng=720x684-1080x1044` may show the effective area of the app screen


### 2.5 Run cvrip

```shell
python cvrip\visualripper.py -device <device_serial> -app <app_name> -package <app_package> -start_activity <activity> -max_action_times <max_actions> [-o output_dir] [-debug] [-eval] [-replay] [-monkey] [-window <"left,top,right,bottom">] [-recognition <recog_strategy>] [-input <input_strategy>] [-state_compare <state_compare_strategy>] [-exclude_screens <excluded_cross_app_screen_folder>]
```

- -device `device_id`:   The device id, which can be in a form like `127.0.0.1:5555` or `emulator-5554`

- -app `app_name`: The short app name for display    

- -package `app_package`: The app package

- -start_activity `activity`:  The start activity for testing

- -max_action_times `max_actions`: the budget of the triggered actions

- -o `output_dir`: The output directory containing the produced UTG, the evaluation data, etc. The default is `output`.

- -debug: when enabling this option, the widget detection results and the action triggering process will be logged.

- -replay: Replay the previously recorded exploration process for debugging.

- -eval: Enable this option to acquire the covered GUI hierarchy information from the device under test

- -monkey: Optional choice to use `monkey` to start an app.

- -utg_template: Optional choice to use the historic UTG template for action generation.

- -window "left,top,right,bottom" : The window rect of an app screen (exclude the status and notification bars of the OS). 

  If this option is not set, the full screen is considered the app screen.

  `left`, `top`, `right`, `bottom` shall be replaced with concrete values or an empty string

- -recognition `recog_strategy`:  The widget recognition strategy. `recog_method` can be：`yolo` （stands for yolov3), `yolov5`, `canny`, `saliency`, `uied`, `ocr`, or the first five choices plus ocr, e.g., `yolo+ocr`. The default choice is `canny`.

- -input `input_strategy`: The action input strategy, which can be: `random`, `art`, `saliency`, `non-text-first`, `equivalence`, or`combined`. The default choice is `combined`

- -action_profile `action_profile`: The probability weight of actions, which could be:

  ```
  click:0.5;swipe:0.2;key_input:0.1;long_press:0.05;drag:0.05;restart:0.1
  ```
  
- -state_compare `state_compare_strategy`: 

  The state recognition method, expressed using a form `acceleration strategy` + `state comparison strategy`. Some examples include: `hash+ssim`, `ahash:1024+ssim:0.5`, `dhash:1024+ssim:0.5:0.9`, `resnet+ssim`, `resnet18+ssim`, `resnet50+ssim_orb`.
  

  `acceleration strategy` can be hash strategy (`ahash`, `phash`, `dhash`), ResNet model (`resnet18`、`resnet50`), or VGG model (`vgg16`). Some strategies can have options, for example, `ahash:1024` denotes using ahash with a length about 1024 for state recognition acceleration.

  If no `acceleration strategy` is provided, and only the `state comparison strategy` is given, e.g., `-state_compare ssim`, that means there is no state recognition acceleration.

  `state comparison strategy`  can be `ssim`, `ssim+orb`,`phash`, or `none`. The strategy can have an additional resize option, e.g., ssim:0.5，denoting resize the screenshot to 1/2 size when doing state recognition. If no such option, no resizing is performed.  The strategy can also have two additional options, e.g., ssim:0.5:0.9，denoting resize the screenshot to 1/2 size when doing state recognition and use 0.9 as the image comparison threshold.  
  

  If options `-state_compare none` is specified, cvrip performs random model-less GUI exploration.

  If no `-state_compare` option is specified, the default state recognition option is `dhash+ssim`.

- -exclude_screens `excluded_cross_app_screen_folder`: The folder of cross-app home screens for preventing cross-app exploration.


## ✨ 3. Demo
```shell
python visualripper.py -o experiment/subject/android/calculator -device emulator-5554 -app calculator -package com.ddnapalon.calculator.gp -start_activity com.ddnapalon.calculator.gp.ScienceFragment -window "0,63,1080,1794" -recognition canny -input random -max_action_times 200

python visualripper.py  -debug -eval-o experiment/subject/android/calculator -device 127.0.0.1:7555 -app calculator -package com.ddnapalon.calculator.gp -start_activity com.ddnapalon.calculator.gp.ScienceFragment -window "0,40,," -recognition canny -input random -max_action_times 200

python visualripper.py -debug -eval -o output -device emulator-5554 -app settings -package com.android.settings -recognition canny -input random -max_action_times 100 -window 0,0,1080,1920

python visualripper.py -o experiment/subject/android/douban -device 127.0.0.1:5555 -app douban -package com.douban.frodo -start_activity com.douban.frodo.MainActivity -window "0,72,,1776" -recognition canny -input random -max_action_times 200 -debug -eval -exclude_screens experiment\cross-app

# random explore
python visualripper.py -o experiment/subject/android/douban -device 127.0.0.1:5555 -app douban -package com.douban.frodo -start_activity com.douban.frodo.MainActivity -window "0,72,,1776" -recognition canny -state_compare none -max_action_times 200 -debug -eval -exclude_screens experiment\cross-app
```


