# Integrating a Camera Function into an Android App

For a lot of apps, taking a picture can be a secondary function and is used as only a small part of the app's main functionality. For situations like this, using the internal camera on Android phones is the solution that makes the most sense, as it allows the app to use the pre-existing camera app to take pictures.

With this in mind, a simple solution for using the built-in camera on Android phones can be made using a relatively simple setup that can be integrated into most apps whether they are brand new or already have an existing structure

## Setting up Permissions

Android permissions are set out in the app's manifesto, which specifies the meta-data for the app (such as the name and icon on phones), the hardware that the app needs to run, and the permissions that the user needs to grant the app. In order to allow an app to use the camera on an android phone, two things must be specified in the `AndroidManifesto.xml` file that all apps have in their main folder

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

If an app can still run without the camera, this can be changed to `android:required="false"`. It is then possible to check if a phone has a camera on runtime by running `hasSystemFeature(PackageManager.FEATURE_CAMERA)` upon app startup, and then disabling any features that require a camera to avoid any issues while running the app. It should also be noted that specifying that your app needs a camera means that it will not be shown on the Google Play store for devices that don't have a camera, thereby reducing any risk of devices without cameras trying to use your app.

## Creating Unique Files

Before taking a photo, it's important to think of how the photo will be stored and find a method of giving each photo a unique name. While for an app that instantly uses a photo taken this may not be a problem, many apps store and reference photos later on, and so one of the key priorities is creating unique filenames which can then be referenced without having any collisions.  This is solved by using the current date and time in the file name, as these will always be unique due to being measured down to the second.

An example of a function that will create a unique file name for the photo is shown below

```java
String mCurrentPhotoPath;

private File createImageFile() throws IOException {

    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(imageFileName, ".jpg", storageDir);

    mCurrentPhotoPath = image.getAbsolutePath();
    return image;
}
```

In order to write a photo to the phone's storage, permission for this should be requested in the app's manifest file. This, like for the camera, will get the user's permission to write to storage that first time the app is run

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

It should be noted that requesting a write permission also implicitly requests a read permission on android, so with this line it is now possible to both read and write to a directory on the phone's storage. On newer versions of Android (i.e. those run on most modern phones) the directory is not accessible to other apps, so in fact there is not always a need to request this permission, as it is often already granted to the app by default

## Taking a Photo

### Understanding Intents

In order to take a picture, an `Intent` must be created. Intents are a key concept in Android development, and are essentially the Android way of describing something which must be done. This intent can then be passed onto another function and then handle the returned file. Android's built in infrastructure means that many functions to do with Intent protocol are automatically triggered during the running of the program.

The two key functions to see here are `startActivityForResult` and `onActivityResult`. When `startActivityForResult` is called, it will launch the activity specified in the provided Intent (the Intent must be defined as returning a result)

```java
void startActivityForResult (Intent intent, int requestCode)
```

Once `startActivityForResult` is finished, it will immediately called `onActivityResult`, providing it with the data from the Intent, the result code (which can be used to check where the result came from), and a result code (to specify if the activity was successfully executed)

```java
void onActivityResult (int requestCode, int resultCode, Intent data)
```

### Writing a Simple Intent

With these two functions in mind, a simple Intent for capturing an image is shown below (this can be expanded to include more than simply taking a photo). While it is not shown here, it should be noted that `onActivityResult` will be a useful function for doing more with the return file from the Intent

```java
static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
```

## Getting the Image Back

Once the photo has been taken (and stored), most apps will want to then get the photo back. This can be done using the  `getUriForFile` function and the `FileProvider` type to provide a secure way of storing photos

### Configuring the File Path Data

The `FileProvider` needs to be configured in the app's manifest file. This

```xml
<provider
  android:name="android.support.v4.content.FileProvider"
  android:authorities="example.android.fileprovider"
  android:exported="false"
  android:grantUriPermissions="true">
  <meta-data
     android:name="android.support.FILE_PROVIDER_PATHS"
     android:resource="@xml/file_paths">
  </meta-data>
</provider>
```

From the `meta-data` tag above it is clear that the provider expects paths to be configured in a specific file, so this must also be made as an `xml` file with the filepath `res/xml/file_paths.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="Android/data/example.package.name/files/Pictures" />
</paths>
```

### Writing the New intent

With these now configured, a new intent can be written, which is able to use these to get back a full-size photo

```java
static final int REQUEST_TAKE_PHOTO = 1;

Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

if (takePictureIntent.resolveActivity(getPackageManager()) != null) {

    File photoFile = null;
    try {
        photoFile = createImageFile();
    } catch (IOException ex) {
        //Catch error
    }
    if (photoFile != null) {
        Uri photoURI = FileProvider.getUriForFile(this, "example.android.fileprovider", photoFile);
        takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
        startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
    }
}
```

## Pic Example

Look it is a cat

![Picture of cute cat](/cat.jpg)
