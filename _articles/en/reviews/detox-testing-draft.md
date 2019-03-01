---
title: Detox testing - draft
redirect_from: []
date: 2019-02-20 14:19:03 +0000
published: false

---
Detox is a gray box end-to-end tests and automation library for mobile apps. Currently, it is only supported for iOS apps built with React Native. If you have a React Native app for iOS on Bitrise, you can run Detox tests.

To see an example configuration, check out [our sample app](https://github.com/bitrise-samples/sample-project-react-native)!

Running Detox requires:

* A Mac with a macOS (El Capitan 10.11 or newer version).
* Xcode 8.3 or newer version with Xcode command line tools.
* A working React Native app.

[Install and set up Detox for your project](https://github.com/wix/detox/blob/master/docs/Introduction.GettingStarted.md#getting-started). You will need to install Homebrew, Node.js and applesimutils, as well as the Detox command line tools. Add Detox to your project and then create and run Detox tests locally.

Once you are done, you can test your Detox-configured project on Bitrise:

1. Create a [release device configuration]() inside `package.json` under the `detox` section.

   **Example:**

       "detox": {
        "configurations": {
          "ios.sim.debug": {
            "binaryPath": "ios/build/Build/Products/Debug-iphonesimulator/SampleProjectReactNative.app",
            "build": "xcodebuild -project ios/SampleProjectReactNative.xcodeproj -scheme SampleProjectReactNative -configuration Debug -sdk iphonesimulator -derivedDataPath ios/build -UseNewBuildSystem=NO",
            "type": "ios.simulator",
            "name": "iPhone 8"
          },
          "ios.sim.release": {
            "binaryPath": "ios/build/Build/Products/Release-iphonesimulator/SampleProjectReactNative.app",
            "build": "xcodebuild -project ios/SampleProjectReactNative.xcodeproj -scheme SampleProjectReactNative -configuration Release -sdk iphonesimulator -derivedDataPath ios/build -UseNewBuildSystem=NO",
            "type": "ios.simulator",
            "name": "iPhone 8"
          }
        },
2. On [bitrise.io](https://app.bitrise.io/), go to your project and open the Workflow Editor.
3. Switch to the workflow you want to use.
4. Add a `Run npm command` Step to your workflow.
5. Add the Detox install command to the `The npm command with arguments to run` input:

       install -g detox-cli
6. Install a test runner. 

   For example, [our sample app](https://github.com/bitrise-samples/sample-project-react-native) uses `mocha`, installed with the `yarn` Step. To install yarn dependencies, just set the `The yarn command to run` input's value to `install`.
7. Add a Script Step to install the necessary utilities and then run Detox.

       #!/bin/bash
       
       # applesimutils is a collection of utils for Apple simulators
       brew tap wix/brew
       brew install applesimutils
       
       # we are building and testing a release device configuration
       detox build --configuration ios.sim.release
       detox test --configuration ios.sim.release --cleanup

   You can, of course, put each of these commands in separate Script Steps, for the sake of modularity. 
8. Run a build!