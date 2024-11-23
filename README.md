
#### AndroidManifest.xml

```bash
<application
    .............>
   <receiver
        android:name=".others.RoutineNow"
        android:exported="false" >
        <intent-filter>
            <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
        </intent-filter>

        <meta-data
            android:name="android.appwidget.provider"
            android:resource="@xml/routine_now_info" />
    </receiver>
</application>
```
#### RoutineNow.java 
```bash
public class RoutineNow extends AppWidgetProvider {

    private static Handler handler = new Handler();
    private static Runnable runnable;

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        for (int appWidgetId : appWidgetIds) {
            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.routine_now);
            startUpdatingWidget(context, appWidgetManager, appWidgetId, views);
        }
    }

    private void startUpdatingWidget(Context context, AppWidgetManager appWidgetManager, int appWidgetId, RemoteViews views) {
        runnable = new Runnable() {
            @Override
            public void run() {
                String currentTime = getCurrentTime();
                views.setTextViewText(R.id.widget_text, currentTime);
                appWidgetManager.updateAppWidget(appWidgetId, views);

                // Schedule the next update
                handler.postDelayed(this, 1000); // Update every second
            }
        };

        // Start the updates
        handler.post(runnable);
    }

    private String getCurrentTime() {
        // Format the current time
        SimpleDateFormat sdf = new SimpleDateFormat("hh:mm:ss a", Locale.getDefault());
        return sdf.format(new Date());
    }

    @Override
    public void onDisabled(Context context) {
        // Stop the updates when the last widget is removed
        if (handler != null && runnable != null) {
            handler.removeCallbacks(runnable);
        }
        super.onDisabled(context);
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
    }
}
```
#### routine_now.xml 
```bash
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:background="@color/orange"
    android:padding="16dp">

    <TextView
        android:id="@+id/widget_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello Widget!"
        android:textSize="24sp" />

    <Button
        android:id="@+id/widget_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click Me" />
</LinearLayout>
```
#### MainActivity.java 
```bash
private void addToHomeScreen() {
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(this);
        ComponentName myWidget = new ComponentName(this, RoutineNow.class);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            // Request widget pinning on supported Android versions
            if (appWidgetManager.isRequestPinAppWidgetSupported()) {
                Intent pinnedWidgetCallbackIntent = new Intent(this, MainActivity.class);
                appWidgetManager.requestPinAppWidget(myWidget, null, pinnedWidgetCallbackIntent.getParcelableExtra(Intent.EXTRA_INTENT));
                Toast.makeText(this, "Pin the widget using the dialog.", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "Pinning widgets is not supported on your device.", Toast.LENGTH_SHORT).show();
            }
        } else {
            Toast.makeText(this, "This feature requires Android 8.0 or higher.", Toast.LENGTH_SHORT).show();
        }
    }
```
#### res/xml/routine_now_info.xml
```bash
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialLayout="@layout/routine_now"
    android:minWidth="150dp"
    android:minHeight="100dp"
    android:updatePeriodMillis="0"
    android:widgetCategory="home_screen" />
```
