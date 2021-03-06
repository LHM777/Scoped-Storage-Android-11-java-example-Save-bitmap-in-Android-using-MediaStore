# What is Scoped Storage?
Android 10 put a greater emphasis on privacy and security. Android 11 continues this emphasis and gives you many tools to achieve this. One of those tools is scoped storage. This feature impacts your app in a big way if you’re leveraging local file access.

With scoped storage, an app no longer has direct access to all the files in external storage. If an app wants to manipulate a file that it didn’t create, it has to get explicit authorization from the user. These request prompts appear for each file. This provides a new level of control for the user. They can now decide what an app can or cannot do with their files.

Because this can have a potentially heavy impact on existing apps, Android 10 provides an opt-out mechanism. When enabled, it allows an app to work without any of these requirements. The caveat here is that when your app targets API 30 (Android 11), scoped storage starts to be mandatory.

So if your app uses device storage, it’s time to start preparing it for scoped storage.



# Public file storage on Android 10
In the past before Android 10, saving a content, let’s say an image, on a device from our app will make that content publicly available to all the apps installed.
Those implementations, together with WRITE_EXTERNAL_STORAGE permissions, work until Android 9, while they throw a SecurityException on Android 10.

On Android 10 things slightly changed: we can still save content in external media directories, but only through the content resolver.

Here is an example of how to do it:

### RELATIVE_PATH :
Can be used to specify a subfolder in the directory of destination e.g. if we are saving into the pictures directory and we want our content to be saved into Our_subdirectory we can just pass ${DIRECTORY_PICTURES}/Our_subdirectory as in the example shown below.


### IS_PENDING : 
Used to tell the content resolver there is an operation going on. Once we copied the data in the destination file (done in the copyFileData() function), we can set this value to false like shown at the end of the example below. Copying the data into the destination file can be easily done.




# Other types of files
I made an example showing how to save an image file. What about other types of files? The MediaStore class contains different subclasses for this purpose. In this example, you could see we were using MediaStore.Images.xyz.

The other available ones are:
- MediaStore.Video
- MediaStore.Audio
- MediaStore.Downloads

As a general rule, I tend to save all the media content in the related subtype folder (so a .mp3 file will use the MediaStore.Audio class) while everything else goes in the downloads folder using the MediaStore.Downloads class.



# Notes
Unfortunately, the APIs used in Android 10 are not available in the older versions of Android, which means we need to maintain two different ways of saving content externally, which I will also cover in this example



# Add an Imageview to layout
We'll get this image and save that using mediastore.

```xml

<?xml version="1.0" encoding="utf-8"?>

<ScrollView android:layout_marginTop="30dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <ImageView
            android:id="@+id/imageView"
            android:layout_width="match_parent"
            android:layout_height="600dp"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:srcCompat="@drawable/img" />

        <Button
            android:id="@+id/btnSaveImage"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Save Image"
            android:textSize="26dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/imageView" />


    </androidx.constraintlayout.widget.ConstraintLayout>


</ScrollView>

```



![xxx](https://user-images.githubusercontent.com/86467782/138275959-a898b137-9afe-48e5-93d7-832072b0ac30.png)




# Get bitmap from Imageview
To save it in gallery

```java

package com.example.saveimage2021;

import androidx.appcompat.app.AppCompatActivity;

import android.content.ContentResolver;
import android.content.ContentValues;
import android.graphics.Bitmap;
import android.graphics.drawable.BitmapDrawable;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.os.Environment;
import android.provider.MediaStore;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.Objects;

public class MainActivity extends AppCompatActivity {

    ImageView ivBackGround;
    Button btnSave;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ivBackGround = findViewById(R.id.imageView);
        btnSave = findViewById(R.id.btnSaveImage);

        btnSave.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                BitmapDrawable bitmapDrawable = (BitmapDrawable) ivBackGround.getDrawable();
                Bitmap bitmap = bitmapDrawable.getBitmap();
                saveImageToGallery(bitmap);
            }
        });


    }



}

```



# Saving image to gallery in Android Q and above.

In this example, we're going to save our image bitmap to a folder called "TestFolder" that is located in the gallery.

```java

    private void saveImageToGallery(Bitmap bitmap){

        OutputStream fos;

        try{

            if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q){

                ContentResolver resolver = getContentResolver();
                ContentValues contentValues =  new ContentValues();
                contentValues.put(MediaStore.MediaColumns.DISPLAY_NAME, "Image_" + ".jpg");
                contentValues.put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg");
                contentValues.put(MediaStore.MediaColumns.RELATIVE_PATH, Environment.DIRECTORY_PICTURES + File.separator + "TestFolder");
                Uri imageUri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues);

                fos = resolver.openOutputStream(Objects.requireNonNull(imageUri));
                bitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
                Objects.requireNonNull(fos);

                Toast.makeText(this, "Image Saved", Toast.LENGTH_SHORT).show();
            }

        }catch(Exception e){

            Toast.makeText(this, "Image not saved \n" + e.toString(), Toast.LENGTH_SHORT).show();
        }


    }
```



# Saving image to gallery in Android versions below Q

There are many implementations that you could do when it comes to saving an image bitmap to gallery for Android versions that are below Q.
Below, is just one of those implementations. This is the one that works for me.

### Implementation #1:
```java

    private void saveImageToGallery(Bitmap bitmap){

        OutputStream fos;

        try{

            if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q){

                ContentResolver resolver = getContentResolver();
                ContentValues contentValues =  new ContentValues();
                contentValues.put(MediaStore.MediaColumns.DISPLAY_NAME, "Image_" + ".jpg");
                contentValues.put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg");
                contentValues.put(MediaStore.MediaColumns.RELATIVE_PATH, Environment.DIRECTORY_PICTURES + File.separator + "TestFolder");
                Uri imageUri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues);

                fos = resolver.openOutputStream(Objects.requireNonNull(imageUri));
                bitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
                Objects.requireNonNull(fos);

                Toast.makeText(this, "Image Saved", Toast.LENGTH_SHORT).show();
            }
            else{
            
                // Save image to gallery
                String savedImageURL = MediaStore.Images.Media.insertImage(getContentResolver(), bitmap, "Bird", "Image of bird");

                // Parse the gallery image url to uri
                Uri savedImageURI = Uri.parse(savedImageURL);

                Toast.makeText(this, "Image saved to internal!!", Toast.LENGTH_SHORT).show();
                resetOpTimes();
                
            }

        }catch(Exception e){

            Toast.makeText(this, "Image not saved \n" + e.toString(), Toast.LENGTH_SHORT).show();
        }


    }
```





# New Permission for Android Q and above.
"Write external storage" permission is not needed

```xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.saveimage2021">

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_MEDIA_LOCATION"/>


    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.SaveImage2021">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>

```
