# Situm iOS Swift Example

**This is a basic example showing how to integrate your Swift aplication with the Situm SDK. For a full-fledged example of the Situm SDK features see our [Objective-C Code Samples](https://github.com/situmtech/situm-ios-code-samples)**

## Table of contents

[Introduction](#introduction)

[Setup](#setup)

1. [Configure our SDK in your iOS project](#configuration)
2. [Set API Key](#apikey)

[Samples](#samples)
1. [Fetch buildings](#fetchBuildings)
2. [Fetch building info](#fetchBuildingInfo)
3. [Fetch nearest building](#fetchNearestBuilding)
4. [Activate the positioning](#positioning)
5. [Navigation](#navigation)

[More information](#moreinfo)
[Support information](#supportinfo)

## Introduction <a name="introduction"></a>

In this tutorial, we will guide you step by step to set up your first Swift application using Situm SDK. Before starting to write code, we recommend you to set up an account in our [Dashboard](https://dashboard.situm.es), retrieve your APIKEY and configure your first building.

1. Go to the [sign in form](http://dashboard.situm.es/accounts/register) and enter your username and password to sign in.
2. Go to the [account section](https://dashboard.situm.es/accounts/profile) and on the bottom, click on "generate one" to generate your API KEY.
3. Go to the [buildings section](http://dashboard.situm.es/buildings) and create your first building.
4. Download [Situm Mapping Tool](https://play.google.com/store/apps/details?id=es.situm.maps). With this application, you will be able to configure and test Situm's indoor positioning system in your buildings (coming soon on iOS).

Perfect! Now you are ready to develop your first indoor positioning application. Following you'll find some examples of how to retrieve buildings, information and position updates using SitumSDK from Swift. **While all this examples are already implemented in this example project, you can use it to get a better understanding of the code**.

## <a name="setup"></a> Setup

### <a name="configuration"></a> Step 1: Configuration guide

To learn how to configure our SDK, please visit our [developers page](http://developers.situm.es/pages/ios/quick_start_guide_swift.html).

### Step 2: Set API Key <a name="apikey"></a>

Now that you have correctly configured your Swift project, you can start writting your application's code. All you need to do is introduce your credentials. You can do that in your `AppDelegate.swift` file. There are two ways of doing this:

##### Using your email address and APIKEY.

This is the recommended option and the one we have implemented in this project. You will find the function in your `AppDelegate.swift` file, just add the line to set your credentials.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        SITServices.provideAPIKey("SET YOUR APIKEY HERE", forEmail: "SET YOUR EMAIL HERE")
        return true
}
```

##### Using your user and password

This is the other available option to provide your credentials, with your username and password. As in the previous case, just add the line to set your credentials.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        SITServices.provideUser("SET YOUR USER HERE", password: "SET YOUR PASSWORD HERE")
        return true
}
```

## Samples <a name="samples"></a>
### <a name="fetchBuildings"></a> Fetch buildings

At this point, you should be able to retrieve the list of buildings associated with your user's account. To do so, include the following code snippet, that will also receive an error object in case the retrieve operation fails.

```swift
var sharedManager: SITCommunicationManager = SITCommunicationManager.shared()

sharedManager.fetchBuildings(options: nil, success: { (mapping: [AnyHashable : Any]?) -> Void in
          var buildings: NSArray = mapping!["results"] as! NSArray
    }, failure: { (error: Error?) in
        // Handle error accordingly
    })
```

### <a name="fetchBuildingInfo"></a> Fetch building info
Once we have the buildings, it is straightforward to get their information. For instance, in order to obtain all the floors of the first building retrieved, we just have to select the required building:

```swift
var building: SITBuilding = buildings[0] as! SITBuilding

sharedManager.fetchBuildingInfo(building.identifier, withOptions: nil, success: { (mapping: [AnyHashable: Any]?) -> Void in

        var buildingInfo: SITBuildingInfo = mapping!["results"] as! SITBuildingInfo

    }, failure: { (error: Error?) in
        // Handle error accordingly
    })
```

As we can see, all requests are very similar, and remain being so for the other resources (points of interest, floorplans, etc.).

### <a name="fetchNearestBuilding"></a> Fetch nearest building
In order to detect what's the nearest building to your current location, you need to fetch the buildings list, obtain your current location using CoreLocation, and then calculate the distance between your current location and each building on your account. This can be done as follows:

```swift
let locationManager: CLLocationManager = CLLocationManager()

···

    func getNearestBuilding() {
    
        //Configure CLLocationManager
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.distanceFilter = kCLDistanceFilterNone
        locationManager.requestAlwaysAuthorization()
        
        //Start and stop location updates in order to get only one update
        locationManager.startUpdatingLocation()
        locationManager.stopUpdatingLocation()
    }
    
    //CLLocationManagerDelegate method
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        let location: CLLocation = locations[0]
        let buildingLocation: CLLocation = getLocationFromLocation2D(buildings[0].center())
        var minDistance: CLLocationDistance = location.distance(from: buildingLocation)
        var selectedBuilding = buildings[0]
        for building in self.buildings {
            let buildingLocation = getLocationFromLocation2D(building.center())
            let distance: CLLocationDistance = location.distance(from: buildingLocation)
            if (distance < minDistance) {
                minDistance = distance
                //At the end of the loop, selectedBuilding contains the nearest building
                selectedBuilding = building
            }
        }
    }
```

### <a name="positioning"></a> Activate the positioning

The last step is to initiate the indoor positioning on a certain building. This will allow the app to retrieve the location of the smartphone within this building. In order to receive location updates, just add the following code to your project:

In your class, make sure to conform to the protocol SITLocationDelegate protocol:

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

### <a name="navigation"></a> Navigation

Situm SDK provides a way to show the indications while you are going from one point to another. Since we have already seen how to get your location and how to plan a route between two points, here we will talk only about how to get the indications. This is a two-steps-functionallity, first we have to tell the route we have planned to do and then update every time we move our position in the route.

* The route planned:

```swift
  let navigationRequest : SITNavigationRequest = SITNavigationRequest.init(route: route)
  navigationManager.requestNavigationUpdates(navigationRequest)
  
  ···
  
    //NavigationManagerDelegate methods
    func navigationManager(_ navigationManager: SITNavigationInterface!, didFailWithError error: Error!) {
        print(error)
    }
    
    func navigationManager(_ navigationManager: SITNavigationInterface!, didUpdate progress: SITNavigationProgress!, on route: SITRoute!) {
        //This contains navigation updates
        print(progress.currentIndication)
    }
    
    func navigationManager(_ navigationManager: SITNavigationInterface!, destinationReachedOn route: SITRoute!) {
        print("Destination reached")
    }
    
    func navigationManager(_ navigationManager: SITNavigationInterface!, userOutsideRoute route: SITRoute!) {
        print("User outside route")
    }
        
```

* Updating your position through the route

```swift
    func locationManager(_ locationManager: SITLocationInterface, didUpdate location: SITLocation) {
        //Every time a new location is received, you must update the navigationManager with it
        if (navigationManager.isRunning()) {
            navigationManager.update(with: location)
        }
    }
```

If you want to know more about the indications, you can check the [SDK Documentation](http://developers.situm.es/sdk_documentation/ios/documentation/html/Classes/SITNavigationProgress.html).

## <a name="moreinfo"></a> More information

More info is available at our [Developers Page](https://des.situm.es/developers/pages/ios/).

## <a name="supportinfo"></a> Support information

For any question or bug report, please send an email to [support@situm.es](mailto:support@situm.es)
