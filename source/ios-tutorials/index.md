---
title: Creating a Camera Application (Swift)
version: v4.14
date: 2021-06-08
github: https://github.com/DJI-Mobile-SDK-Tutorials/iOS-FPVDemo-Swift
keywords: [iOS FPVDemo, capture, shoot photo, take photo, record video, basic tutorial, Swift]
---

*If you come across any mistakes in this tutorial feel free to open Github pull requests.*

---

This tutorial is designed for you to gain a basic understanding of the DJI Mobile SDK. It will implement the FPV view and two basic camera functionalities: **Take Photo** and **Record video**.

You can download the tutorial's final sample project from this [Github Page](https://github.com/DJI-Mobile-SDK-Tutorials/iOS-FPVDemo-Swift).

See [this Github Page](https://github.com/DJI-Mobile-SDK-Tutorials/iOS-FPVDemo) for an Objective C version. 

We used a Mavic Pro to make this demo.

## Importing and Activating the SDK

Now, let's create a new project in Xcode, choose **Single View Application** template for your project and press "Next", then enter "FPVDemo" in the **Product Name** field and keep the other default settings.

Once the project is created, let's delete the "ViewController.swift" file created by Xcode by default. Create a new ViewController named "FPVViewController".

Now, let's install the **DJISDK.framework** in the Xcode project using Cocoapods and implement the SDK activation process in the "FPVViewController.swift" file. If you are not familiar with the process of installing and activating DJI SDK, please check the Github source code and this tutorial: [Importing and Activating DJI SDK in Xcode Project](../application-development-workflow/workflow-integrate.html#Xcode-Project-Integration) for details. //TODO: check link, use Swift version if one exists.

## Application Activation and Aircraft Binding in China

 For DJI SDK mobile application used in China, it's required to activate the application and bind the aircraft to the user's DJI account.

 If an application is not activated, the aircraft not bound (if required), or a legacy version of the SDK (< 4.1) is being used, all **camera live streams** will be disabled, and flight will be limited to a zone of 100m diameter and 30m height to ensure the aircraft stays within line of sight.

 To learn how to implement this feature, please check this tutorial [Application Activation and Aircraft Binding](./ActivationAndBinding.html).

## Implementing the First Person View

### Importing the DJIWidget

  **1**. We use the **FFMPEG** decoding library (found at <a href="http://ffmpeg.org" target="_blank">http://ffmpeg.org</a>) to do software video decoding here. For the hardware video decoding, we provide a **H264VTDecode** decoding library. You can find them in the `DJIWidget/DJIWidget/VideoPreviewer` folder, which you can download it from <a href="https://github.com/dji-sdk/DJIWidget/tree/master/DJIWidget/VideoPreviewer" target="_blank">DJIWidget Github Repository</a>.

  DJIWidget is available through [CocoaPods](http://cocoapods.org). To install it, simply add the following line to your Podfile:

  ```
  pod 'DJIWidget', '~> 1.6.4'
  ```

  > Note: Remember to add `use_frameworks!` in the pod file to use DJIWidget as a dynamic framework in Swift project.
  >

### Working on the DJICameraViewController

 **1**. Let's open the `FPVDemo.xcworkspace` file in Xcode and open the Main.storyboard, then add a new View Controller and set **FPVViewController** as the **Class** for it:

  ![rootController](../images/tutorials-and-samples/iOS/FPVDemo/rootController.png)

Add a UIView inside the View Controller. Then, add two UIButtons and one UISegmentedControl at the bottom of the View Controller as shown below:

  ![Storyboard](../images/tutorials-and-samples/iOS/FPVDemo/Storyboard.png)

  Go to "FPVViewController.swift" file and import the **DJISDK** and **DJIVideoPreviewer** header files. Next implement four delegate protocols and set the IBOutlets and IBActions for the UI we just create in Main.storyboard as shown below:

~~~Swift
import UIKit
import DJISDK
import DJIWidget

class FPVViewController: UIViewController,  DJIVideoFeedListener, DJISDKManagerDelegate, DJICameraDelegate {

    @IBOutlet var recordButton: UIButton!
    @IBOutlet var workModeSegmentControl: UISegmentedControl!
    @IBOutlet var fpvView: UIView!

    @IBAction func captureAction(_ sender: UIButton) {
        // Implement Later
    }
    
    @IBAction func recordAction(_ sender: UIButton) {
        // Implement Later
    }
    
    @IBAction func workModeSegmentChange(_ sender: UISegmentedControl) {
        // Implement Later
    }
~~~

**2.** Furthermore, let's create the `setupVideoPreviewer` and `resetVideoPreviewer` methods as shown below:

~~~Swift
    func setupVideoPreviewer() {
        DJIVideoPreviewer.instance().setView(self.fpvView)
        let product = DJISDKManager.product();
        
        //Use "SecondaryVideoFeed" if the DJI Product is A3, N3, Matrice 600, or Matrice 600 Pro, otherwise, use "primaryVideoFeed".
        if ((product?.model == DJIAircraftModelNameA3)
            || (product?.model == DJIAircraftModelNameN3)
            || (product?.model == DJIAircraftModelNameMatrice600)
            || (product?.model == DJIAircraftModelNameMatrice600Pro)) {
            DJISDKManager.videoFeeder()?.secondaryVideoFeed.add(self, with: nil)
        } else {
            DJISDKManager.videoFeeder()?.primaryVideoFeed.add(self, with: nil)
        }
        DJIVideoPreviewer.instance().start()
    }

    func resetVideoPreview() {
        DJIVideoPreviewer.instance().unSetView()
        let product = DJISDKManager.product();
        
        //Use "SecondaryVideoFeed" if the DJI Product is A3, N3, Matrice 600, or Matrice 600 Pro, otherwise, use "primaryVideoFeed".
        if ((product?.model == DJIAircraftModelNameA3)
            || (product?.model == DJIAircraftModelNameN3)
            || (product?.model == DJIAircraftModelNameMatrice600)
            || (product?.model == DJIAircraftModelNameMatrice600Pro)) {
            DJISDKManager.videoFeeder()?.secondaryVideoFeed.remove(self)
        } else {
            DJISDKManager.videoFeeder()?.primaryVideoFeed.remove(self)
        }
    }
~~~

In the `setupVideoPreviewer` method, we set the `fpvView` instance variable as the superview of the `MovieGLView` in the **DJIVideoPreviewer** class. Then get a `DJIBaseProduct` object from the `DJISDKManager` and use its `model` property to display the correct video feed (Some larger drones have a dedicated FPV camera in addition to the primary detachable payload(s)). Lastly, we invoke the `start` method of `DJIVideoPreviewer` instance to start the video decoding.

In the `resetVideoPreview` method, we invoke the `unSetView` method of `DJIVideoPreviewer` instance to remove the `MovieGLView` of the `DJIVideoPreviewer` class from the `fpvPreviewView` instance first. Then also check the current `product`'s model using an if statement.

If the product is **A3**, **N3**, **Matrice 600** or **Matrice 600 Pro**, we invoke the `removeListener:` method of `DJIVideoFeeder` class to remove the `DJICameraViewController` listener from the `secondaryVideoFeed` for video feed, otherwise, we remove the `DJICameraViewController` listener from the `primaryVideoFeed` for video feed.

**3.** Once you finished the above steps, let's implement the `DJISDKManagerDelegate` delegate methods and the `viewWillDisappear` method as shown below:

~~~Swift
    func fetchCamera() -> DJICamera? {
        if let aircraft = DJISDKManager.product() as? DJIAircraft {
            return aircraft.camera
        }
        if let handheld = DJISDKManager.product() as? DJIHandheld {
            return handheld.camera
        }
        return nil
    }

    // MARK: DJISDKManagerDelegate Methods
    func productConnected(_ product: DJIBaseProduct?) {
        
        NSLog("Product Connected")
        if let camera = fetchCamera() {
            camera.delegate = self
        }
        self.setupVideoPreviewer()
        
        //If this demo is used in China, it's required to login to your DJI account to activate the application. Also you need to use DJI Go app to bind the aircraft to your DJI account. For more details, please check this demo's tutorial.
        DJISDKManager.userAccountManager().logIntoDJIUserAccount(withAuthorizationRequired: false) { (state, error) in
            if let _ = error {
                NSLog("Login failed: %@" + String(describing: error))
            }
        }
    }
    
    func productDisconnected() {
        NSLog("Product Disconnected")

        if let camera = fetchCamera(), let delegate = camera.delegate, delegate.isEqual(self) {
            camera.delegate = nil
        }
        self.resetVideoPreview()
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
    
        if let camera = fetchCamera(), let delegate = camera.delegate, delegate.isEqual(self) {
            camera.delegate = nil
        }
        
        self.resetVideoPreview()
    }
~~~

Firstly, we create the `- (DJICamera*) fetchCamera` method to fetch the updated DJICamera object. Before we get the return DJICamera object, we need to check if the product object of DJISDKManager is kind of **DJIAircraft** of **DJIHandheld** class. Since the camera component of the aircraft or handheld device may be changed or disconnected, we need to fetch the camera object everytime we want to use it to ensure we get the correct camera object.

Then invoke the `setupVideoPreviewer` method to setup the DJIVideoPreviewer in the `productConnected` delegate method. Lastly, reset the `camera` instance's delegate to nil and invoke the `resetVideoPreview` method to reset the videoPreviewer in the `productDisconnected` and `viewWillDisappear` methods.

**4**. Lastly, let's implement the "DJIVideoFeedListener" and "DJICameraDelegate" delegate methods, as shown below:

~~~Swift

#pragma mark - DJIVideoFeedListener

-(void)videoFeed:(DJIVideoFeed *)videoFeed didUpdateVideoData:(NSData *)videoData {
    [[DJIVideoPreviewer instance] push:(uint8_t *)videoData.bytes length:(int)videoData.length];
}

#pragma mark - DJICameraDelegate

-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState
{
}

~~~

Here, we use the `-(void)videoFeed:(DJIVideoFeed *)videoFeed didUpdateVideoData:(NSData *)videoData` method to get the live H264 video feed data and send them to the **DJIVideoPreviewer** to decode.

Moreover, the `-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState` method is used to get the camera state from the camera on your aircraft. It will be invoked frequently, so you can update your user interface or camera settings accordingly here.

## Connecting to the Aircraft or Handheld Device

Please check this [Connect Mobile Device and Run Application](../application-development-workflow/workflow-run.html#connect-mobile-device-and-run-application) guide to run the application and view the live video stream from your DJI product's camera based on what we've finished of the application so far!

## Enjoying the First Person View

If you can see the live video stream in the application, congratulations! Let's move forward.

  ![fpv](../images/tutorials-and-samples/iOS/FPVDemo/fpv.jpg)

## Implementing the Capture function

Let's implement the `captureAction` IBAction method as shown below:

~~~Swift
- (IBAction)captureAction:(id)sender {

    DJICamera* camera = [self fetchCamera];
    if (camera) {
        WeakRef(target);
        [camera setShootPhotoMode:DJICameraShootPhotoModeSingle withCompletion:^(NSError * _Nullable error) {
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                [camera startShootPhotoWithCompletion:^(NSError * _Nullable error) {
                    WeakReturn(target);
                    if (error) {
                        [target showAlertViewWithTitle:@"Take Photo Error" withMessage:error.description];
                    }
                }];
            });
        }];
    }
}
~~~

In the code above, we firstly invoke the following method of DJICamera to set the shoot photo mode to `DJICameraShootPhotoModeSingle`:

~~~Swift
- (void)setShootPhotoMode:(DJICameraShootPhotoMode)mode withCompletion:(DJICompletionBlock)completion;
~~~

  Normally, once an operation is finished, the camera still needs some time to finish up all the work. It's safe to delay the next operation after an operation is finished. So let's enqueue the block which may invoke the following method with 1 second delay to control the camera to shoot a photo:

`- (void)startShootPhotoWithCompletion:(DJICompletionBlock)completion;`

  You can check the shoot photo result from the `DJICompletionBlock`.

  Build and run your project and then try the shoot photo function. If the screen flash after your press the **Capture** button, your capture fuction should work now.

## Implementing the Record function

### 1. Switching Camera Mode

   Before we implement the record function, we need to switch the camera work mode firstly.

   Let's check the DJICameraMode enum in **DJICameraSettingsDef.h** file.

~~~Swift
/**
 *  Camera work modes.
 */
typedef NS_ENUM (NSUInteger, DJICameraMode){

    /**
     *  Capture mode. In this mode, the user can capture pictures.
     */
    DJICameraModeShootPhoto = 0x00,

    /**
     *  Record mode. In this mode, the user can record videos.
     */
    DJICameraModeRecordVideo = 0x01,

    /**
     *  Playback mode. In this mode, the user can preview photos and videos, and can
     *  delete files. It is supported by  Phantom 3 Profressional camera, X3, X5 and X5R
     *  cameras on aircraft and Phantom 4 camera. Playback mode is not  supported by
     *  Z30, X5S, X4S, Phantom 4 Pro, Mavic Pro, Phantom 3 Standard, Phantom 3 Advanced,
     *  Phantom 3 4K and  Osmo series.
     */
    DJICameraModePlayback = 0x02,

    /**
     *  In this mode, the user can download media to the Mobile Device. Not supported by
     *  X5 camera nor X5R camera while  mounted on aircraft.
     */
    DJICameraModeMediaDownload = 0x03,

    /**
     *  In this mode, live stream resolution and frame rate will be 1080i50 (PAL) or
     *  720p60 (NTSC). In this mode videos can  be recorded. Still photos can also be
     *  taken only when video is recording. The only way to exit broadcast mode is to
     *  change modes to `DJICameraModeRecordVideo`. Only supported by Inspire 2.
     */
    DJICameraModeBroadcast = 0x04,

    /**
     *  The camera work mode is unknown.
     */
    DJICameraModeUnknown = 0xFF
};
~~~

   You can see from above that there are 5 types of **DJICameraMode**. Here we use the first two types.

   Remember we create a UISegment Control in the storyboard? We can update the state of the segmented control when switching between **DJICameraModeShootPhoto** and **DJICameraModeRecordVideo** using the previous delegate method like this:

~~~Swift
-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState
{        
    //Update UISegmented Control's state
    if (systemState.mode == DJICameraModeShootPhoto) {
        [self.changeWorkModeSegmentControl setSelectedSegmentIndex:0];
    }else if (systemState.mode == DJICameraModeRecordVideo){
        [self.changeWorkModeSegmentControl setSelectedSegmentIndex:1];
    }
}

~~~

 Now we can implement the `changeWorkModeAction` IBAction method as follows:

~~~Swift

- (IBAction)changeWorkModeAction:(id)sender {

    UISegmentedControl *segmentControl = (UISegmentedControl *)sender;
    DJICamera* camera = [self fetchCamera];

    if (camera) {
        WeakRef(target);
        if (segmentControl.selectedSegmentIndex == 0) { //Take photo

            [camera setMode:DJICameraModeShootPhoto withCompletion:^(NSError * _Nullable error) {
                WeakReturn(target);
                if (error) {
                    [target showAlertViewWithTitle:@"Set DJICameraModeShootPhoto Failed" withMessage:error.description];
                }
            }];

        }else if (segmentControl.selectedSegmentIndex == 1){ //Record video

            [camera setMode:DJICameraModeRecordVideo withCompletion:^(NSError * _Nullable error) {
                WeakReturn(target);
                if (error) {
                    [target showAlertViewWithTitle:@"Set DJICameraModeRecordVideo Failed" withMessage:error.description];
                }
            }];

        }
    }

}

~~~

 In the code above, we invoke the
 `- (void)setMode:(DJICameraMode)mode withCompletion:(DJICompletionBlock)completion;` method of DJICamera to change the camera mode.  Here we add two UIAlertViews to show warnings when the user set `DJICameraMode` failed.

### 2. Working on the Record Action

  Firstly, let's go to Main.storyboard and drag a UILabel on top of the screen, set up the Autolayout constraints for it and create an IBOutlet named `currentRecordTimeLabel` in the **DJICameraViewController.m** file.

  Then add a BOOL variable `isRecording` in the class extension part of **DJICameraViewController**. Be sure to hide the `currentRecordTimeLabel` in the `viewDidLoad` method.

~~~Swift
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.currentRecordTimeLabel setHidden:YES];
}
~~~

We can update the bool value for `isRecording` and `currentRecordTimeLabel`'s text value in the following delegate method:

~~~Swift

-(void) camera:(DJICamera*)camera didUpdateSystemState:(DJICameraSystemState*)systemState
{
    self.isRecording = systemState.isRecording;

    [self.currentRecordTimeLabel setHidden:!self.isRecording];
    [self.currentRecordTimeLabel setText:[self formattingSeconds:systemState.currentVideoRecordingTimeInSeconds]];

    if (self.isRecording) {
        [self.recordBtn setTitle:@"Stop Record" forState:UIControlStateNormal];
    }else
    {
        [self.recordBtn setTitle:@"Start Record" forState:UIControlStateNormal];
    }

    //Update UISegmented Control's state
    if (systemState.mode == DJICameraModeShootPhoto) {
        [self.changeWorkModeSegmentControl setSelectedSegmentIndex:0];
    }else if (systemState.mode == DJICameraModeRecordVideo){
        [self.changeWorkModeSegmentControl setSelectedSegmentIndex:1];
    }   
}

~~~

   Because the text value of `currentRecordingTime` is counted in seconds, so we need to convert it to "mm:ss" format like this:

~~~Swift
- (NSString *)formattingSeconds:(int)seconds
{
    NSDate *date = [NSDate dateWithTimeIntervalSince1970:seconds];
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    [formatter setDateFormat:@"mm:ss"];
    [formatter setTimeZone:[NSTimeZone timeZoneForSecondsFromGMT:0]];

    NSString *formattedTimeString = [formatter stringFromDate:date];
    return formattedTimeString;
}
~~~

   Next, add the following codes to the `recordAction` IBAction method as follows:

~~~Swift
- (IBAction)recordAction:(id)sender {

    DJICamera* camera = [self fetchCamera];
    if (camera) {
        WeakRef(target);
        if (self.isRecording) {
            [camera stopRecordVideoWithCompletion:^(NSError * _Nullable error) {
                WeakReturn(target);
                if (error) {
                    [target showAlertViewWithTitle:@"Stop Record Video Error" withMessage:error.description];
                }
            }];
        }else
        {
            [camera startRecordVideoWithCompletion:^(NSError * _Nullable error) {
                WeakReturn(target);
                if (error) {
                    [target showAlertViewWithTitle:@"Start Record Video Error" withMessage:error.description];
                }
            }];
        }
    }
}
~~~

  In the code above, we implement the `startRecordVideoWithCompletion` and `stopRecordVideoWithCompletion` methods of the **DJICamera** class based on the `isRecording` property value. And show an alertView when an error occurs.

  Now, let's build and run the project and check the functions. You can try to play with the **Record** and **Switch Camera WorkMode** functions, if everything goes well, you should see the screenshot like this:

  ![Screenshot](../images/tutorials-and-samples/iOS/FPVDemo/record_screenshot.jpg)

  Congratulations! Your Aerial FPV iOS app is complete, you can now use this app to control the camera of your Mavic Pro.

### Summary

   In this tutorial, you’ve learned how to use DJI Mobile SDK to show the FPV View from the aircraft's camera and control the camera of DJI's Aircraft to shoot photo and record video. These are the most basic and common features in a typical drone mobile app: **Capture** and **Record**. However, if you want to create a drone app which is more fancy, you still have a long way to go. More advanced features should be implemented, including previewing the photo and video in the SD Card, showing the OSD data of the aircraft and so on. Hope you enjoy this tutorial, and stay tuned for our next one!
