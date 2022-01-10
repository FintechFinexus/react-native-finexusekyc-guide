# react-native-finexusekyc

Developer Documentation

### Table of Contents

 - [Installation](#installation)
 - [Firebase](#firebase)
 - [Android Modifications](#android)
 - [iOS Modifications](#ios)
 - [Usage](#usage)
 - [Theming](#theme)
 - [Creating Custom Form](#custom-form)
 - [Validation](#validation)
 - [Returned results](#returned-results)
 - [Known issues](#known-issues)
 - [License](#license)

## Installation

### Required packages
```sh
npm i react-native-blob-util react-native-vector-icons react-native-video react-native-camera react-native-ffmpeg react-native-svg @react-native-community/image-editor
```
or
```sh
yarn add react-native-blob-util react-native-vector-icons react-native-video react-native-camera react-native-ffmpeg react-native-svg @react-native-community/image-editor
```

then run

```sh
npx react-native link
```

```sh
yarn add git+https://<token>:x-oauth-basic@github.com/FinexusGroup/react-native-finexusekyc.git
```


## Firebase
If your application already has a firebase setup then you can skip this step.

If your application does not have a firebase setup then let us know your application id. eg. com.company.applicationname.

Finexus will provide a firebase file named google-services.json for android and GoogleService-Info.plist iOS for you to include in the installation.
Depending on your file structure you may need to place this file in different locations.
Default location to place this file will be in

 -- Android: ./android/app

 -- iOS: ./ios : iOS extra steps, 

 1. Open the ios folder of your project with XCode.
 2. Right click on Runner in the left-hand column then select Add Files to Runner from the menu.
 3. Select the GoogleServices-Info. plist file from your computer and click on the Add button.
 4. Now you'll see GoogleServices-Info. plist listed under Runner.

## Android

android/build.gradle
```gf
buildscript {
    ext {
      ...
      minSdkVersion = 24
      reactNativeFFmpegPackage = "min"
    }
    repositories {
      ...
    }
    dependencies {
      ...
      classpath 'com.google.gms:google-services:4.0.1' //minimum version
    }
}

allprojects {
    repositories {
        mavenLocal()
        mavenCentral()
        ...
    }
}

```

android/app/build.gradle
```gf
android {
  ...
  defaultConfig {
    ...
    missingDimensionStrategy 'react-native-camera', 'mlkit'
  }
}

...
// place these at the bottom of the file
apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
apply plugin: 'com.google.gms.google-services' 

```

android/app/src/main/AndroidManifest.xml
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />

<application ...>
...
  <meta-data
      android:name="com.google.firebase.ml.vision.DEPENDENCIES"
      android:value="ocr, face" /> <!-- choose models that you will use -->
</application>
```

## iOS

ios/podfile
```pod

...
platform :ios, '12.0'

target 'AwesomeTSProject' do

  pod 'react-native-ffmpeg/min', :podspec => '../node_modules/react-native-ffmpeg/react-native-ffmpeg.podspec' <--- Note add this before `use_native_modules!`

  ...

  pod 'Firebase/Core'
  pod 'react-native-camera', path: '../node_modules/react-native-camera', subspecs: [
  'FaceDetectorMLKit'
  ]

```

ios/{appname}/AppDelegate.m
```m
#import <Firebase.h> // <--- add this
...

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [FIRApp configure]; // <--- add this
  ...
}
```

ios/{appname}/Info.plist
```plist
<dict>
  ...
  <key>NSCameraUsageDescription</key>
	<string>Allow $(PRODUCT_NAME) to take pictures and record video?</string>

  <!-- Include these when encountering problems -->
  <key>NSPhotoLibraryAddUsageDescription</key>
  <string>Allow $(PRODUCT_NAME) to access Photo Library?</string>
  <key>NSPhotoLibraryUsageDescription</key>
  <string>Allow $(PRODUCT_NAME) to access Photo Library?</string>
  <key>NSMicrophoneUsageDescription</key>
  <string>Allow $(PRODUCT_NAME) to access microphone?</string>
  ...
</dict>
```

## Usage

```js
import Finexusekyc, { resetStates } from "react-native-finexusekyc";
import license from './licensename.json';

  <Finexusekyc
    onComplete={(values) => {
      console.log('Completed', values);
    }}
    backNavigation={() => {
      console.log('Back button pressed')
    }}
    language={"English"}
    license={license}
  />

```
Property | Type | Description
--- | --- | ---
onComplete | callback | Callback will be called at the end of a successful AIV call. This will return values captured within the package & aiv results. Returned values can refer to table below
backNavigation | callback | Callback that is called when the back button of the first page. Use this to return to your previous screen.
language | "English", "Malay", "Chinese", "Tamil" | Pass in 1 of the 4 strings to adjust the language.
license | json | Please pass in the license file provided here so the package can work. 
highRiskFlag | "Y", "N" | Please pass the high risk country flag from walletUpgradeEligibility here. 
env | string | Please pass in "UAT" for testing purposes and "PRODUCTION" for production purposes.
theme (optional) | object | Color theme that follows React Native Paper's colour scheme, or you can enter your own object according to their respective key value pairs. Check theme construction below
defaultValues (optional) | object | Default values for AIV required form values: { 'fullName'?: string, 'idNumber'?: string, 'nationality'?: 'MY' (alpha-2 country codes), 'idType'?: 'P' (passport) or 'N' (mykad) } 
validationSchema (optional) | object | Validation schema follows validate.js's constraint format. Check constraint construction section below for more info
CustomForm (optional) | React.ReactElement | Render prop to allow to render anything below the 4 default fields
onSubmitForm (optional) | callback | Callback that is called when button is pressed in User details screen. Return any key value pair object e.g. { age: 22 }, to indicate successful form. Return empty object {} to indicate failed form submission and will not navigate to the next screen.

### Theme

```js
import {
  DefaultTheme as PaperDefaultTheme,
} from 'react-native-paper';

  const theme = {
    ...PaperDefaultTheme,
    roundness: 4,
    mode: 'adaptive' as const,
    colors: {
      ...PaperDefaultTheme.colors,
      primary: '#FECA34',
      secondary: '#3C4045',
      background: '#FAFAFA',
      surface: '#F9F9F9',
      error: '#B00020',
      disabled: '#D3D3D3',
    },
  };

  import Finexusekyc, { resetStates } from "react-native-finexusekyc";
  import license from './licensename.json';

  function Screen(){
    //...
    return (
      <Finexusekyc
        onComplete={(values) => {
          console.log('Completed', values);
        }}
        backNavigation={() => {
          console.log('Back button pressed')
        }}
        language={"English"}
        license={license}
        env={"UAT"}
        highRiskFlag={"N"}
        theme={theme}
      />
    )
  }

```
For more details, you can refer to https://callstack.github.io/react-native-paper/theming.html.


### Custom Form

```js
  import React from 'react';
  //You can also use the InputComponent from the package
  import Finexusekyc, { resetStates, InputComponent } from "react-native-finexusekyc";
  import license from './licensename.json';

  function Screen(){
    const [values, setValues] = React.useState({});
    return (
      <Finexusekyc
        ...
        CustomForm={() => RenderForm(values, setValues)}
        onSubmitForm={() => {
          if (values) {
            //when submit button is pressed in form, it will assume custom form is complete if returned with object containing any key
            return values
          }
          else return {}
        }}
      />
    )
  }

  function RenderForm({ values; setValues }){

    const _onChangeText = (text, name) => {
      setValues({
        ...values,
        [name]: text
      })
    }

    return(
      <>
        <InputComponent value={values.name} onChangeText={(text) => _onChangeText(text, 'name')} />
      </>
    )
  }

```

### Validation
Validation library is using validate.js
E.g. of a validation constraint
```js
const FormConstraints = {
  //this validates the value of key = fullName in an object for presence which doesn't allow empty string, 
  //and idNumber validation for Mykad format only if idType chosen is N value for MyKad
  fullName: {
    presence: {
      allowEmpty: false,
      message: lang?.required,
    },
    format: {
      pattern: "^[a-zA-Z @'/-]+$",
      message: lang?.onlyAlphabet,
    }
  },
  idNumber: {
    presence: {
      allowEmpty: false,
      message: lang?.required,
    },
    format: {
      pattern: '^[a-zA-Z0-9]*$',
      message: lang?.onlyAlphabetNumber
    },
    equality: {
      attribute: "idType",
      message: lang?.invalidNRIC,
      comparator: function (v1, v2) {
        const invalidRegions = ['17', '18', '19', '20', '69', '70', '72', '73', '80', '81', '94', '95', '96', '97', '98', '99', '00']
        let region = v1.substring(6, 8);
        const isValidRegion = !invalidRegions.includes(region);
        if (v1 && v2 === 'N') {
          if (!isValidRegion) return false;
          let matched = v1
            .toString()
            .match(
              /([0-9][0-9])((0[1-9])|(1[0-2]))((0[1-9])|([1-2][0-9])|(3[0-1]))([0-9][0-9])([0-9][0-9][0-9][0-9])/
            );
          if (!matched) return false;
          return true;
        } else {
          return true;
        }
      }
    }
  },
}
```

However if using custom form, it is recommended to do your own validations and controlling it at button press using the onSubmitForm prop.

For more details, you can refer to https://validatejs.org/

### Returned results
After completing AIV, the onComplete will return an object with these values. 
*Update*Returned values now includes AIV api results.

Property | Type | Description
--- | --- | ---
fullName | String | User details to submit to AIV
idNumber | String | User details to submit to AIV
idType | String | User details to submit to AIV
nationality | String | User details to submit to AIV
agreementToTerms | boolean | Agreement can only be true to complete the process.
frontmykad | string | Path to retrieve captured assets, calling resetStates will delete these files
backmykad | string | Path to retrieve captured assets, calling resetStates will delete these files
passportPhoto | string | Path to retrieve captured assets, calling resetStates will delete these files
passportPhoto2 | string | Path to retrieve captured assets, calling resetStates will delete these files
suppDocument | string | Path to retrieve captured assets, calling resetStates will delete these files
suppDocument2 | string | Path to retrieve captured assets, calling resetStates will delete these files
videoSelfie | string | Path to retrieve captured assets, calling resetStates will delete these files
aivRefId | string | Reference for the AIV api
<!-- aivStatus | [string] | Array containing 2 string with status of "SUCCESS" or "FAIL"
aivErrors | [{ msgCode: string, msgText: string }] | Array containing error code and its respective text -->
form[keys] | any | Extra fields added by developer in key value pairs


### Known issues

App can crash for android 10 due to image editor - https://github.com/callstack/react-native-image-editor/issues/73. You can apply the following fix at:
node_modules/@react-native-community/image-editor/android/src/main/java/com/reactnativecommunity/imageeditor/ImageEditorModule.java
```java
    //Add this function at line 258
    private String getMimeType() throws IOException {

      BitmapFactory.Options outOptions = new BitmapFactory.Options();
      outOptions.inJustDecodeBounds = true;

      InputStream inputStream = openBitmapInputStream();

      try {
        BitmapFactory.decodeStream(inputStream, null, outOptions);
        return outOptions.outMimeType;
      } finally {
        if (inputStream != null) {
          inputStream.close();
        }
      }
    }

    //then change this in line 291 after adding the above function or 274 before
    String mimeType = outOptions.outMimeType;
    if (mimeType == null || mimeType.isEmpty()) {
      throw new IOException("Could not determine MIME type");
    }
    //to
    String mimeType = outOptions.outMimeType;
    if (mimeType == null || mimeType.isEmpty()) {
      mimeType = getMimeType();
          
      if (mimeType == null || mimeType.isEmpty()) {
        throw new IOException("Could not determine MIME type");
      }
    }
```


### License

This project is using react-native-ffmpeg which is licensed under the LGPL v3.0. 
FFmpeg is only used in this way -y -safe 0 -f concat -i ${documentDir}/list.txt -c copy ${documentDir}/output.mp4 in this project.

### See Also

- [FFmpeg](https://www.ffmpeg.org)
- [Mobile FFmpeg Wiki](https://github.com/tanersener/mobile-ffmpeg/wiki)
- [FFmpeg License and Legal Considerations](https://ffmpeg.org/legal.html)