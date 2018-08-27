# ios-swift-app-bridgingExample

### Table of contents

  1. [ Introduction ](#introduction)
  2. [ Configure our SDK in your Swift project (Manual installation) ](#configureprojectswift)
  3. [ Set API Key ](#apikeyswift)
  4. [ Download your buildings ](#fetchindoorbuildingsswift)
  5. [ Download building data ](#fetchbuildinginfoswift)
  6. [ Activate the positioning ](#positioningswift)

<a name="introduction"></a>
### Introduction 

In this tutorial, we will guide you step by step to set up your first Objective C application using Situm SDK. Before starting to write code, we recommend you to set up an account in our [Dashboard](https://dashboard.situm.es), retrieve your APIKEY and configure your first building.

1. Go to the [sign in form](http://dashboard.situm.es/accounts/register) and enter your username and password to sign in.
2. Go to the [account section](https://dashboard.situm.es/accounts/profile) and on the bottom, click on "generate one" to generate your API KEY.
3. Go to the [buildings section](http://dashboard.situm.es/buildings) and create your first building.
4. Download [SitumMaps](https://play.google.com/store/apps/details?id=es.situm.maps). With this application, you will be able to configure and test Situm's indoor positioning system in your buildings (coming soon on iOS).

Perfect! Now you are ready to develop your first indoor positioning application. Following you'll find some examples of how to retrieve buildings, information and position updates using SitumSDK from Swift. **While all this examples are already implemented in this example project, you can use it to get a better understanding of the code**.

<a name="configureprojectswift"></a>
### Step 1: Configure our SDK in your Swift project (Manual installation) 

First of all, you must configure Situm SDK in your iOS project.

In order to include the SitumSDK framework you should follow this instructions:

1- Drag SitumSDK.framework file to the xcode to the most convinient place in your project. Make sure to check the 'Copy items if needed' option.

2- Link the following libraries:

    - CoreLocation
    - CoreMotion
    - libc++.tbd
    - libz.tbd

3- In the configuration of your project, under Linking/Other Linking Flags add the flag "-ObjC"

4- Disable Bitcode. Go to the Build settings tab of your app, search for the option Enable Bitcode and select NO as the value for the setting.

5- In order to ask for permission to use the location of the user you should include the following keys in your Info.plist file:

    - NSLocationWhenInUseUsageDescription (in XCode, 'Privacy - Location When In Use Usage Description') and NSLocationAlwaysUsageDescription (in XCode, 'Privacy - Location Always and When In Use Usage Description') with the value 'Location is required to find out where you are' or a custom message that you like.

6- Create a header file (.h) and add the line:

```objc
#import <SitumSDK/SitumSDK.h>
```

7- In the configuration of your project, under Swift compiler - General/Objective-C Bridging Header add the route from the project root to the header file created previously.

You can now compile and check everything is working.

And that's all. From now on, you should be able to use Situm SDK in your app.

<a name="apikeyswift"></a>
### Step 2: Set API Key 

Now that you have correctly configured your Swift project, you can start writting your application's code. All you need to do is introduce your credentials. You can do that in your AppDelegate.swiftfile. There are two ways of doing this:

##### Using your email address and APIKEY.

This is the recommended option and the one we have implemented in this project. Write the following sentence on the -application:didFinishLaunchingWithOptions: method.

```swift
SITServices.provideAPIKey("SET YOUR APIKEY HERE", forEmail: "SET YOUR EMAIL HERE")
```

##### Using your user and password

This is the other available option to provide your credentials, with your username and password. As in the previous case, write the following sentence on the -application:didFinishLaunchingWithOptions: method.

```swift
SITServices.provideUser("SET YOUR USER HERE", password: "SET YOUR PASSWORD HERE")
```

<a name="fetchindoorbuildingsswift"></a>
### Step 3: Download your buildings 

At this point, you should be able to retrieve the list of buildings associated with your user's account. To do so, include the following code snippet, that will also receive an error object in case the retrieve operation fails.

```swift
var sharedManager: SITCommunicationManager = SITCommunicationManager.shared()

sharedManager.fetchBuildings(options: nil, success: { (mapping: [AnyHashable : Any]?) -> Void in
          var buildings: NSArray = mapping!["results"] as! NSArray
    }, failure: { (error: Error?) in
        // Handle error accordingly
    })
```
<a name="fetchbuildinginfoswift"></a>
### Step 4: Download building data
Once we have the buildings, it is straightforward to get their information. For instance, in order to obtain all the floors of the first building retrieved, we just have to select the required building:

```swift
var building: SITBuilding = buildings[0] as! SITBuilding

sharedManager.fetchBuildingInfo(building.identifier, withOptions: nil, success: { (mapping: [AnyHashable: Any]?) -> Void in

        var buildingInfo: SITBuildingInfo = mapping!["results"] as! SITBuildingInfo

    }, failure: { (error: Error?) in
        // Handle error accordingly
    })
```

As we can see, all requests are very similar, and remain being so for the other resources (events, points of interest, floorplans, etc.).

<a name="positioningswift"></a>
### Step 5: Activate the positioning

The last step is to initiate the indoor positioning on a certain building. This will allow the app to retrieve the location of the smartphone within this building. In order to receive location updates, just add the following code to your project:

In your class, make sure to conform to the protocol SITLocationDelegate

```swift
var sharedLocationManager: SITLocationManager = SITLocationManager.sharedInstance()

sharedLocationManager.delegate = self

let request: SITLocationRequest = SITLocationRequest.init(priority: SITLocationPriority.highAccuracy, provider: SITLocationProvider.hybridProvider, updateInterval: 1, buildingID: building.identifier!, operationQueue: nil, options: nil)

sharedLocationManager.requestLocationUpdates(request)
```

Implement SITLocationDelegate methods where you'll receive location updates, error notifications and state changes.

```swift
func locationManager(_ locationManager: SITLocationInterface, didUpdate location: SITLocation) {
    // Handle location update
}

func locationManager(_ locationManager: SITLocationInterface, didFailWithError error: Error) {
    // Handle error
}

func locationManager(_ locationManager: SITLocationInterface, didUpdate state: SITLocationState) {
    // Handle location manager state
}
```