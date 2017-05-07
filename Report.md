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

## Creating Unique Files

Before taking a photo, it's important to think of how the photo will be stored. Onne of the key issues with storing files as photos is creating unique filenames which can then be referenced without having any collisions. This is solved by using the current date and time in the file name, as these will always be unique.

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

It should be noted that in order to write a photo to the phone's storage, permission for this should be requested in the app's manifest file. This, like for the camera, will get the user's permission to write to storage that first time the app is run

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```

## Taking a Photo

In order to take a picture, an `Intent` must be created. This is the Android way of describing something which must be done, which can then be passed onto another function and then handle the returned file

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

### Configuring the file path data

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
    <external-path name="my_images" path="Android/data/com.example.package.name/files/Pictures" />
</paths>
```

### Writing the new intent

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

## Getting the Image Back - BAD VERSION??

Once the photo has been taken (and stored), most apps will want to then get the photo back. This can be done in the `onActivityResult` which the `Intent` will deliver the photo to. The photo can be decoded as a `Bitmap` using the `BitmapFactory`, and then used as required

```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    try {
        if (requestCode == REQUEST_TAKE_PHOTO && resultCode == RESULT_OK){
            Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath);
            ...
        }
        else {
            //No image taken
        }
    } catch (Exception e) {
        //Catch exception
    }

}
```
