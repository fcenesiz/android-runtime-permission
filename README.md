# android-runtime-permission

You need to extend this activity on your activity

### with JAVA
````java
import android.content.DialogInterface;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Bundle;
import android.provider.Settings;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

public abstract class RuntimePermissionActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }


    public void askPermission(final String[] askedPermissions, final int requestCode) {


        int permissionCheck = PackageManager.PERMISSION_GRANTED;
        boolean makeExcuses = false;


        //permissionCheck=0  -> OK
        //else -> No permission grant
        //makeExcuses = false -> First asking attempt
        //makeExcuses = true  -> user disagree permissions, need to make an excuse
        for (String permission : askedPermissions) {

            permissionCheck = permissionCheck + ContextCompat.checkSelfPermission(this, permission);
            makeExcuses = makeExcuses || ActivityCompat.shouldShowRequestPermissionRationale(this, permission);
        }

        if (permissionCheck != PackageManager.PERMISSION_GRANTED) {

            if (makeExcuses) {

                AlertDialog.Builder builder = new AlertDialog.Builder(this);
                builder.setTitle("Why should you grant?");
                builder.setMessage("If you want to search you need to give this permission.");
                builder.setNegativeButton("No permission", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.cancel();
                    }
                });

                builder.setPositiveButton("I want to grant", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        ActivityCompat.requestPermissions(RuntimePermissionActivity.this, askedPermissions, requestCode);
                    }
                });

                builder.show();

            } else {
                ActivityCompat.requestPermissions(RuntimePermissionActivity.this, askedPermissions, requestCode);
            }

        } else {

            permissionGranted(requestCode);

        }

    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);

        int permissionCheck = PackageManager.PERMISSION_GRANTED;


        //permissionCheck=0  -> OK
        for (int permissionResult : grantResults) {

            permissionCheck = permissionCheck + permissionResult;

        }

        if ((grantResults.length > 0) && permissionCheck == PackageManager.PERMISSION_GRANTED) {

            permissionGranted(requestCode);
        } else {

            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.setTitle("Need Permission");
            builder.setMessage("You need to give all permissions in settings.");
            builder.setNegativeButton("No", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    dialog.cancel();
                }
            });

            builder.setPositiveButton("I want to grant", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {

                    Intent intent = new Intent();
                    intent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                    intent.addCategory(Intent.CATEGORY_DEFAULT);
                    intent.setData(Uri.parse("package:" + getPackageName()));
                    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                    intent.addFlags(Intent.FLAG_ACTIVITY_NO_HISTORY);
                    intent.addFlags(Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS);
                    startActivity(intent);

                }
            });
            builder.show();
        }
    }
    public abstract void permissionGranted(int requestCode);
}
````

### with Kotlin

````kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.content.pm.PackageManager
import androidx.core.content.ContextCompat
import androidx.core.app.ActivityCompat
import android.content.DialogInterface
import android.content.Intent
import android.net.Uri
import android.provider.Settings
import androidx.appcompat.app.AlertDialog

abstract class RuntimePermissionActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }

    fun askPermission(askedPermissions: Array<String?>, requestCode: Int) {
        var permissionCheck = PackageManager.PERMISSION_GRANTED
        var makeExcuses = false


        //permissionCheck=0  -> OK
        //else -> No permission grant
        //makeExcuses = false -> First asking attempt
        //makeExcuses = true  -> user disagree permissions, need to make an excuse
        for (permission in askedPermissions) {
            permissionCheck += ContextCompat.checkSelfPermission(this, permission!!)
            makeExcuses =
                makeExcuses || ActivityCompat.shouldShowRequestPermissionRationale(this, permission)
        }
        if (permissionCheck != PackageManager.PERMISSION_GRANTED) {
            if (makeExcuses) {
                val builder = AlertDialog.Builder(this)
                builder.setTitle("Why should you grant?")
                builder.setMessage("If you want to search you need to give this permission.")
                builder.setNegativeButton("No permission") { dialog, which -> dialog.cancel() }
                builder.setPositiveButton("I want to grant") { dialog, which ->
                    ActivityCompat.requestPermissions(
                        this@RuntimePermissionActivity,
                        askedPermissions,
                        requestCode
                    )
                }
                builder.show()
            } else {
                ActivityCompat.requestPermissions(
                    this@RuntimePermissionActivity,
                    askedPermissions,
                    requestCode
                )
            }
        } else {
            permissionGranted(requestCode)
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        var permissionCheck = PackageManager.PERMISSION_GRANTED


        //permissionCheck=0  -> OK
        for (permissionResult in grantResults) {
            permissionCheck += permissionResult
        }
        if (grantResults.isNotEmpty() && permissionCheck == PackageManager.PERMISSION_GRANTED) {
            permissionGranted(requestCode)
        } else {
            val builder = AlertDialog.Builder(this)
            builder.setTitle("Need Permission")
            builder.setMessage("You need to give all permissions in settings.")
            builder.setNegativeButton("No") { dialog, which -> dialog.cancel() }
            builder.setPositiveButton("I want to grant") { dialog, which ->
                val intent = Intent()
                intent.action = Settings.ACTION_APPLICATION_DETAILS_SETTINGS
                intent.addCategory(Intent.CATEGORY_DEFAULT)
                intent.data = Uri.parse("package:$packageName")
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
                intent.addFlags(Intent.FLAG_ACTIVITY_NO_HISTORY)
                intent.addFlags(Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS)
                startActivity(intent)
            }
            builder.show()
        }
    }

    abstract fun permissionGranted(requestCode: Int)
}
````
