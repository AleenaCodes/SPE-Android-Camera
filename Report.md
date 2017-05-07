# Integrating a Camera Function into an Android App

For a lot of apps, taking a picture can be a secondary function &&. For situations like this, using the internal camera on Android phones is the solution that makes the most sense, as it allows the app to use the pre-existing camera app to take pictures

## Setting up Permissions

Android permissions are set out in the app's manifesto. In order to allow an app to use the camera on an android phone, two things must be specified in the `AndroidManifesto.xml` file that all app's have in their main folder

The first is to ask the user's permission to use the phone's camera. This will cause a popup the first time the user uses the app to get them to give the app permission to use the camera

```xml
<manifest ...>
  <uses-permission android:name="android.permission.CAMERA" />
  ...
</manifest>
```

The second is to specify that the app needs a camera to run.

```xml
<manifest ..>
  <uses-feature android:name="android.hardware.camera" android:required="true" />
...
</manifest>
```

If an app can still run without the camera, this can be changed to `android:required="false"`. It is then possible to check if a phone has a camera on runtime, and then modify the app accordingly.

## Taking a Photo
