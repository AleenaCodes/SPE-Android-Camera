# This is a tester header

## Subheading

Here is some java

```java
import static android.content.Context.LAYOUT_INFLATER_SERVICE;

/**
 * Created by Rae on 11/02/2017.
 */

public class ImageAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    private static final int TYPE_HEADER = 0;
    private static final int TYPE_ITEM = 1;
    private EditText mURLField;

    private String[] titles = {"Take Photo",
            "My Location",
            "My Images",
            "Enter URL"};
}
```

Here is some xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="eu.kudan.ar">

    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-feature android:name="android.hardware.location.gps" />
    <uses-feature android:name="android.hardware.screen.portrait" />
    <uses-feature android:name="android.hardware.camera" android:required="true" />

    <application

       android:allowBackup="true"
        android:icon="@drawable/anthromos"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/MyMaterialTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".BeginActivity"></activity>
        <activity android:name=".MapActivity"></activity>
        <activity android:name=".ArJigsaw"></activity>
        <activity android:name=".PickImageActivity"></activity>
        <activity android:name=".PickSizeActivity"></activity>
        <activity android:name=".LoadImageActivity"></activity>
        <activity android:name=".TakePictureActivity"></activity>

        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="main.java.eu.kudan.ar.android.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"></meta-data>
        </provider>
        <!--change from example app name-->
    </application>

</manifest>
```

##Another subheading

Here is a longer paragraph. Writing this because I'd hope that this report has some words that come in sentences that are longer than 5 words

```
<?xml version="1.0" encoding="utf-8"?>

<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="Android/data/eu.kudan.ar/files/Pictures" />
</paths>
```

Well here is some more java

```
private ViewHolder(View itemView) {
    super(itemView);
    mImageButton = (ImageView) itemView.findViewById(R.id.get_info);
    mItemTitle = (TextView) itemView.findViewById(R.id.retrieve_from);

    itemView.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            int position = getAdapterPosition();

//                    Snackbar.make(v, "Click detected on item " + position,
//                            Snackbar.LENGTH_LONG)
//                            .setAction("Action", null).show();

            if(position == 1){
                //TODO: Get image from camera feed
                Intent intent = new Intent(v.getContext(), TakePictureActivity.class);
                v.getContext().startActivity(intent);
            }
            else if (position == 2){
                Intent intent = new Intent(v.getContext(), MapActivity.class);
                v.getContext().startActivity(intent);
            }
            else if (position == 3){
                //TODO: Load image from gallery
                Intent intent = new Intent(v.getContext(), LoadImageActivity.class);
                v.getContext().startActivity(intent);
            }
            else if(position == 4){
                getURL(v);
                }
        }
    });
}
```
