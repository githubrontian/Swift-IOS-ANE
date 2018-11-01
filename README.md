# FreSwift

Example Xcode projects showing how to create AIR Native Extensions for iOS, tvOS & OSX using Swift.   
It supports iOS 9.0+, tvOS 9.2+, OSX 10.10+

#### Xcode 9.4.1 (9F2000) must be used with Apple Swift version 4.1.2 (swiftlang-902.0.54 clang-902.0.39.2)
It is not possible to mix Swift versions in the same app. Therefore all Swift based ANEs must use the same exact version.
ABI stability is planned for Swift 5 in late 2018

This project is used as the basis for the following ANEs   
[Google Maps ANE](https://github.com/tuarua/Google-Maps-ANE)       
[AdMob ANE](https://github.com/tuarua/AdMob-ANE)   
[WebViewANE](https://github.com/tuarua/WebViewANE)    
[AR-ANE](https://github.com/tuarua/AR-ANE)     
[ML-ANE](https://github.com/tuarua/ML-ANE)


-------------

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=5UR2T52J633RC)

### Getting Started

A basic Hello World [starter project](/starter_projects) is included for each target

An iOS based walkthrough video is available on [Youtube](https://www.youtube.com/watch?v=pjZPzo1A6Ro)

### How to use

[Full documentation](https://tuarua.github.io/swiftdocs/freswift/index.html) is provided   

The following table shows the primitive as3 types which can easily be converted to/from Swift types

| AS3 type | Swift type | AS3 param->Swift | return Swift->AS3 |
|:--------:|:--------:|:--------------|:-----------|
| String | String | `let str = String(argv[0])` | `return str.toFREObject()`|
| int | Int | `let i = Int(argv[0])` | `return i.toFREObject()`|
| Boolean | Bool | `let b = Bool(argv[0])` | `return b.toFREObject()`|
| Number | Double | `let dbl = Double(argv[0])` | `return dbl.toFREObject()`|
| Number | CGFloat | `let cfl = CGFloat(argv[0])` | `return cfl.toFREObject()`|
| Date | Date | `let date = Date(argv[0])` | `return date.toFREObject()`|
| Rectangle | CGRect | `let rect = CGRect(argv[0])` | `return rect.toFREObject()` |
| Point | CGPoint | `let pnt = CGPoint(argv[0])` | `return pnt.toFREObject()` |
| Vector int | [Int] | `let a = [Int](argv[0])` | `return a.toFREObject()`|
| Vector Boolean | [Bool] | `let a = [Bool](argv[0])` | `return a.toFREObject()`|
| Vector Number | [Double] | `let a = [Double](argv[0])` | `return a.toFREObject()`|
| Vector String | [String] | `let a = [String](argv[0])` | `return a.toFREObject()`|
| Object | [String, Any]? | `let dct = Dictionary.init(argv[0])` | N/A |
| null | nil | | return nil |


#### Basic Types

```swift
let myString: String? = String(argv[0])
let myInt = Int(argv[1])
let myBool = Bool(argv[2])

let swiftString = "I am a string from Swift"
return swiftString.toFREObject()
```

#### Creating new FREObjects

```swift
let newPerson = try FREObject(className: "com.tuarua.Person")

// create a FREObject passing args
// 
// The following param types are allowed: 
// String, Int, UInt, Double, Float, CGFloat, NSNumber, Bool, Date, CGRect, CGPoint, FREObject
let frePerson = FREObject(className: "com.tuarua.Person", args: "Bob", "Doe", 28, myFREObject)
```

#### Calling Methods

```swift
// call a FREObject method passing args
// 
// The following param types are allowed: 
// String, Int, UInt, Double, Float, CGFloat, NSNumber, Bool, Date, CGRect, CGPoint, FREObject
let addition = try freCalculator.call(method: "add", args: 100, 31) {
```

#### Getting / Setting Properties

```swift
let oldAge = Int(person["age"])
let newAge = oldAge + 10

// Set property using braces access
person["age"] = (oldAge + 10).toFREObject()

// Set property using setProp
try person.setProp(name: "age", value: oldAge + 10)

```

#### Arrays

```swift
let airArray: FREArray = FREArray(argv[0])
// convert to a Swift [String]
let airStringVector = [String](argv[0])

// create a Vector.<com.tuarua.Person> with fixed length of 5
let myVector = try FREArray(className: "com.tuarua.Person", length: 5, fixed: true)
let airArrayLen = airArray.length

// loop over FREArray
for fre in airArray {
    trace(Int(fre))
}

// set element 0 to 123
airArray[0] = 123.toFREObject()

// return Int Array to AIR
let swiftArr: [Int] = [99, 98, 92, 97, 95]
return swiftArr.toFREObject()
```

#### Sending Events back to AIR

```swift
trace("Hi", "There")

// with interpolation
trace("My name is: \(name)")

dispatchEvent("MY_EVENT", "this is a test")
```

#### Bitmapdata

```swift
if let img = UIImage(freObject: argv[0]) {
    if let rootViewController = UIApplication.shared.keyWindow?.rootViewController {
        let imgView: UIImageView = UIImageView(image: img)
        imgView.frame = CGRect(x: 0, y: 0, width: img.size.width, height: img.size.height)
        rootViewController.view.addSubview(imgView)
    }
}
```

#### ByteArrays

```swift
let asByteArray = FreByteArraySwift(freByteArray: argv[0])
if let byteData = asByteArray.value { // NSData
	let base64Encoded = byteData.base64EncodedString(options: .lineLength64Characters)
}
```
  
#### Error Handling

```swift
do {
    _ = try person.getProp(name: "doNotExist") //calling a property that doesn't exist
} catch let e as FreError {
    if let aneError = e.getError(#file, #line, #column) {
        return aneError //return the error as an actionscript error
    }
} catch {}
```

#### Advanced Example - Extending. Convert to/from SCNVector3
```swift
public extension SCNVector3 {
    init?(_ freObject: FREObject?) {
        guard let rv = freObject else {
            return nil
        }
        self.init(Float(rv["x"]) ?? 0,
                  Float(rv["y"]) ?? 0,
                  Float(rv["z"]) ?? 0)
    }
    func toFREObject() -> FREObject? {
        do {
            let ret = try FREObject(className: "flash.geom.Vector3D",
                                    args: self.x, self.y, self.z)
            return ret
        } catch {
        }
        return nil
    }
}
```
----------

#### applicationDidFinishLaunching
The static library contains a predefined `+(void)load` method in FreMacros.h. This method can safely be declared in different ANEs.
It is also called once for each ANE and very early in the launch cycle. In here the SwiftController is inited and `onLoad()` called.
This makes an ideal place to add observers for applicationDidFinishLaunching and any other calls which would normally be added as app delegates, thus removing the restriction of one ANE declaring itself as the "owner".   
Note: We have no FREContext yet so calls such as trace, sendEvent will not work.

```swift
@objc func applicationDidFinishLaunching(_ notification: Notification) {
   appDidFinishLaunchingNotif = notification //save the notification for later
}
func onLoad() {
NotificationCenter.default.addObserver(self, 
            selector: #selector(applicationDidFinishLaunching),
            name: NSNotification.Name.UIApplicationDidFinishLaunching, 
            object: nil)      
}
```
----------

### Required AS3 classes
com.tuarua.fre.ANEUtils.as and com.tuarua.fre.ANEError.as are required by FreSwift and should be included in the AS3 library of your ANE


### Prerequisites

You will need

- Xcode 9.4
- Xcode 9.1 for iOS Simulator
- IntelliJ IDEA
- AIR 29
- wget
