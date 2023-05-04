# SDK iOS Beta Documentation

> :bangbang: **The Bridgefy SDK** currently supports **P2P connections**, but we're adding **Mesh Networking, Broadcast, and Signal Protocol** encryption by mid-October! You can begin integrating the current version and add the updates as they're published.

## Overview

The Bridgefy Software Development Kit (SDK) is a state-of-the-art, plug-and-play package of awesomeness that will let people use your mobile app when they don’t have access to the Internet by using mesh networks.

Integrate the Bridgefy SDK into your Android and iOS app to reach the 3.5 billion people that don’t always have access to an Internet connection, and watch engagement and revenue grow!

**Website**. https://bridgefy.me/sdk/

 **Email**. contact@bridgefy.me

**Twitter**. https://twitter.com/bridgefy 

**Facebook**. https://www.facebook.com/bridgefy

## Operation mode

All the connections are handled seamlessly by the SDK to create a mesh network. The size of this network depends on the number of devices connected and the environment as a variable factor, allowing you to join nodes in the same network or nodes in different networks.

![https://github.com/bridgefy/bridgefy-ios-sdk-sample/blob/master/img/Mobile_Adhoc_Network.gif?raw=true](https://github.com/bridgefy/bridgefy-ios-sdk-sample/blob/master/img/Mobile_Adhoc_Network.gif?raw=true)

## Installation

### Swift Package Manager

The Swift Package Manager is a tool for managing the distribution of Swift code. It’s integrated with the Swift build system to automate the process of downloading, compiling, and linking dependencies.

1. Follow the [Apple documentation](https://developer.apple.com/documentation/xcode/adding-package-dependencies-to-your-app) on how to add a package to your project.
2. Paste the following URL `https://github.com/bridgefy/sdk-ios-beta`, into the Package Dependencies search bar.
3. Use the version-based Package Requirements, and set the value to the latest version of the Bridgefy SDK.

### Cocoapods

> The official BridgefySDK pod hasn't been released yet, but you can add it to your project using the Beta repository.

CocoaPods is a dependency manager that lets you add third-party frameworks and libraries to projects. For installation instructions, see [Install CocoaPods](https://guides.cocoapods.org/using/getting-started.html#toc_3).

Add the Bridgefy SDK to your Podfile and install it in your project as follows:

1. Add pod 'BridgefySDK' to the do block:

    ```cocoapods
    platform :ios, '13.0'

    target 'SDK test' do
    use_frameworks!

    pod 'BridgefySDK', :git => 'https://github.com/bridgefy/sdk-ios-beta', :branch => 'main'
    end
    ```

2. Save the Podfile.
3. Run the following command in the Terminal:

    ```terminal
    $ pod install
    ```

### Manually

1. Clone this repository or download the `BridgefySDK.framework` from this [link](https://bridgefy.me/beta/BridgefySDK.xcframework.zip).
2. Drag the `BridgefySDK.framework` folder to the top of your project's hierarchy in Xcode.

    <p align=center>
    <img src="img/Installation1.png"/>
    </p>

3. Select the option `Copy items if needed` from the dialog window that appears after releasing the folder into your project.

    <p align=center>
    <img src="img/Installation2.png"/>
    </p>

4. In the general section of your projects target, go to the section `Frameworks, Libraries and Embedded Content` look for the `BridgefySDK.framework`, and in the `Embed` column select the option ***Embed & Sign***

    <p align=center>
    <img src="img/Installation3.png"/>
    </p>

5. Import the Bridgefy SDK using the following code:

    ```swift
    import BridgefySDK
    ```

### Permissions

The BridgefySDK requires permission to use the Bluetooth antenna of the devices where it's installed; to achieve this you have to add a couple of entries on the `Info.plist` of your project, the entries are depicted in the following image:

<p align=center>
<img src="img/Permissions.png"/>
</p>

## Usage

### Initialization

The init method initializes the Bridgefy SDK with the given API key and propagation profile. The delegate parameter is required and should be an object that conforms to the BridgefyDelegate protocol. The verboseLogging parameter is optional and enables more detailed logging if set to true.

The following code shows how to start the SDK (using your API key) and how to assign the delegate.

```swift
do {
      bridgefy = try Bridgefy(withApiKey: apiKey,
                              propagationProfile: PropagationProfile,
                              delegate: BridgefyDelegate,
                              verboseLogging: Bool)
} catch {
    // Handle the error
}

```

The string **apiKey** represents a valid API key. An Internet connection is needed at least for the first time in order to validate the license.
The **delegate** is the class that will implement all the delegate methods from the BridgefySDK.

### PropagationProfile
The PropagationProfile parameter in the init method allows developers to define a series of properties and rules for the propagation of messages. It's an optional parameter with a default value of .standard.

Once the service is started, the following delegate function is called:

```swift
func bridgefyDidStart(with userId: UUID)
```

The **currentUserID** is the id used to identify the current user/device in the BridgefySDK.

In the case an error occurs while starting the BridgefySDK, the following delegate function is called:

```swift
func bridgefyDidFailToStart(with error: BridgefyError)
```

To stop the SDK, use the following function:

```swift
  func stop()
```

### Nearby peer detection

The following method is invoked when a peer has established connection:

```swift
func bridgefyDidConnect(with userId: UUID)
```

**userID**: Identifier of the user that has established a connection.

When a peer is disconnected(out of range), the following method will be invoked:

```swift
func bridgefyDidDisconnect(from userId: UUID)
```

**userID**: Identifier of the disconnected user.

### Sending data

The following method is used to send data using a transmission mode. This method returns a UUID to identify the message sent.

```swift
do {
    let messageID = try bridgefy.send(Data,
                                 using: TransmissionMode)
            
} catch {
    // Handle the error
}
```

**messageID**: Unique identifier related to the message.

If the message was successfully sent the following delegate method is called:

```swift
func bridgefyDidSendMessage(with messageId: UUID)
```

**messageID**: The unique identifier of the message sent.

***Note:*** Is important to notice that the call of this delegate method doesn't mean that the message was delivered, this is due to the nature of how the mesages travel through the mesh network created by the BridgefySDK. The ONLY scenario where you can assume that the message was delivered is when it was sent using the `p2p` transmission mode; otherwise it only means that there's no pre-validation error and the SDK will start propagating the message.

If an error occurs while sending a message, the following delegate method is called:

```swift
func bridgefyDidFailSendingMessage(with messageId: UUID,
                                   withError error: BridgefyError)
```

### Receiving Data

When a packet has been received, the following method will be invoked:

```swift
func bridgefyDidReceiveData(_ data: Data,
                            with messageId: UUID,
                            using transmissionMode: TransmissionMode)
```

**data**: Received data.
**messageId**: The id of the message that was received
**transmissionMode**: The transmission mode used to propagate a message

**Transmission Modes**:

```swift
public enum TransmissionMode {
    case p2p(userId: UUID)
    case mesh(userId: UUID)
    case broadcast(senderId: UUID)
}
```

The mode used to propagate a message through nearby devices:

**p2p(userId: UUID)**: Sends the message data only when the receiver is in range, otherwise an error is reported.
**mesh(userId: UUID))**: Sends the message data using the mesh created by the SDK. It doesn’t need the receiver to be in range. 
**broadcast(senderId: UUID)**: Sends a packet using mesh without a defined receiver. The packet is broadcast to all nearby users that are or aren’t in range.

### Direct and mesh transmission

Direct transmission is a mechanism used to deliver packets to a user that is nearby or visible (a connection has been detected).

Mesh transmission is a mechanism used to deliver offline packets even when the receiving user isn’t nearby or visible. It can be achieved taking advantage of other nearby peers; these receive the package, hold it, and forward to other peers trying to find the receiver.
