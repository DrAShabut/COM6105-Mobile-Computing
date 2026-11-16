# Lab 2: Foreground Worker and Live Notification (Jetpack Compose)

## Prerequisite
Complete Lab 1: Cat Agent Tracker with WorkManager Chains.

# Learning Outcomes

By the end of this lab you will be able to:

- Create a foreground `CoroutineWorker`.
- Create and register a notification channel.
- Build foreground notifications using `NotificationCompat`.
- Create `ForegroundInfo` objects.
- Publish progress updates from a worker.
- Observe progress using `WorkInfo`.
- Handle Android 13 notification permissions.
- Build a Jetpack Compose screen that reacts to worker progress.

---

# Existing Project Structure

Your project should already contain a structure similar to:

```text
com.example.catagenttracker
│
├── MainActivity.kt
│
├── ui
│   └── ... existing Compose screens
│
└── worker
    ├── CatStretchingWorker.kt
    ├── CatFurGroomingWorker.kt
    ├── CatLitterBoxSittingWorker.kt
    └── CatSuitUpWorker.kt
```

At the end of this lab add:

```text
worker
└── CatDispatchWorker.kt

ui
└── CatAgentScreen.kt
```

---

# Before You Start

Add the following dependencies if they are not already present.

```kotlin
implementation("androidx.work:work-runtime-ktx:2.10.5")
implementation("androidx.lifecycle:lifecycle-runtime-compose:2.9.4")
implementation("androidx.activity:activity-compose:1.11.0")
implementation("androidx.compose.material3:material3")
```

Sync Gradle.

---

# Scenario

The Cat Agent has completed:

- Stretching
- Grooming
- Litter-box inspection
- Suiting up

The final stage is deployment.

When the user selects **Dispatch Cat Agent**, a foreground worker starts.

The worker:

- Displays a notification.
- Counts down from 10 seconds.
- Updates the notification every second.
- Sends progress updates to the activity.
- Displays an arrival message when complete.

---

# Part 1 – Create CatDispatchWorker

Create:

```text
worker/CatDispatchWorker.kt
```

Replace the complete implementation with the gollowing code:

```kotlin
package com.example.catagenttracker.worker

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.content.pm.ServiceInfo
import android.os.Build
import androidx.core.app.NotificationCompat
import androidx.core.app.NotificationManagerCompat
import androidx.work.CoroutineWorker
import androidx.work.ForegroundInfo
import androidx.work.WorkerParameters
import androidx.work.workDataOf
import com.example.catagenttracker.MainActivity
import com.example.catagenttracker.R
import kotlinx.coroutines.delay

class CatDispatchWorker(
    appContext: Context,
    params: WorkerParameters
) : CoroutineWorker(appContext, params) {

    companion object {

        const val NOTIFICATION_ID = 1001

        const val INITIAL_COUNTDOWN = 10

        const val CHANNEL_ID = "CatDispatch"

        const val KEY_CAT_AGENT_ID = "catAgentId"

        const val KEY_SECONDS_REMAINING = "secondsRemaining"
    }

    override suspend fun doWork(): Result {

        val agentId =
            inputData.getString(KEY_CAT_AGENT_ID)
                ?: "Unknown Agent"

        createNotificationChannel()

        setForeground(
            createForegroundInfo(
                agentId,
                INITIAL_COUNTDOWN
            )
        )

        startCountdown(agentId)

        return Result.success()
    }

    private suspend fun startCountdown(
        agentId: String
    ) {

        val manager =
            NotificationManagerCompat.from(
                applicationContext
            )

        for (seconds in INITIAL_COUNTDOWN downTo 0) {

            setProgress(
                workDataOf(
                    KEY_CAT_AGENT_ID to agentId,
                    KEY_SECONDS_REMAINING to seconds
                )
            )

            if (Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU ||
                androidx.core.content.ContextCompat.checkSelfPermission(
                    applicationContext,
                    android.Manifest.permission.POST_NOTIFICATIONS
                ) == android.content.pm.PackageManager.PERMISSION_GRANTED
            ) {
                try {
                    manager.notify(
                        NOTIFICATION_ID,
                        buildNotification(
                            agentId,
                            seconds
                        )
                    )
                } catch (e: SecurityException) {
                    // Ignore if permission denied
                }
            }

            delay(1000)
        }
    }

    private fun createNotificationChannel() {

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

            val channel =
                NotificationChannel(
                    CHANNEL_ID,
                    "Cat Agent Dispatch",
                    NotificationManager.IMPORTANCE_LOW
                )

            val manager =
                applicationContext.getSystemService(
                    NotificationManager::class.java
                )

            manager.createNotificationChannel(channel)
        }
    }

    private fun createPendingIntent(): PendingIntent {

        val intent = Intent(
            applicationContext,
            MainActivity::class.java
        )

        return PendingIntent.getActivity(
            applicationContext,
            0,
            intent,
            PendingIntent.FLAG_UPDATE_CURRENT or
                    PendingIntent.FLAG_IMMUTABLE
        )
    }

    private fun buildNotification(
        agentId: String,
        secondsRemaining: Int
    ): Notification {

        return NotificationCompat.Builder(
            applicationContext,
            CHANNEL_ID
        )
            .setSmallIcon(R.mipmap.ic_launcher)
            .setContentTitle("Cat Agent Tracking")
            .setContentText(
                "Agent $agentId arriving in $secondsRemaining seconds"
            )
            .setContentIntent(
                createPendingIntent()
            )
            .setOngoing(true)
            .build()
    }

    private fun createForegroundInfo(
        agentId: String,
        secondsRemaining: Int
    ): ForegroundInfo {

        val notification =
            buildNotification(
                agentId,
                secondsRemaining
            )

        return if (
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q
        ) {

            ForegroundInfo(
                NOTIFICATION_ID,
                notification,
                ServiceInfo.FOREGROUND_SERVICE_TYPE_DATA_SYNC
            )

        } else {

            ForegroundInfo(
                NOTIFICATION_ID,
                notification
            )
        }
    }
}
```

The worker must:

- Extend `CoroutineWorker`.
- Create a notification channel.
- Create a foreground notification.
- Create a `ForegroundInfo` object.
- Update progress every second.
- Return `Result.success()`.

### Checkpoint

When finished, Android Studio should recognise:

```kotlin
CatDispatchWorker
```

without any compilation errors.

---

# Part 2 – Update AndroidManifest.xml

Add the following permissions above the `<application>` tag.

```xml
<uses-permission
    android:name="android.permission.POST_NOTIFICATIONS" />

<uses-permission
    android:name="android.permission.FOREGROUND_SERVICE" />

<uses-permission
    android:name="android.permission.FOREGROUND_SERVICE_DATA_SYNC" />
```

Inside the `<application>` tag add:

```xml
<service
    android:name="androidx.work.impl.foreground.SystemForegroundService"
    android:foregroundServiceType="dataSync"
    tools:node="merge" />
```

Sync the project.

---

# Part 3 – Create the Compose Screen

Reuse the existing:

```text
ui
```

package.

Create:

```text
ui/CatAgentScreen.kt
```

Paste the complete `CatAgentScreen` composable provided below@:

```kotlin
package com.example.catagenttracker.ui.screen

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.Card
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun CatAgentScreen(
    statusText: String,
    onDispatchClick: () -> Unit
) {

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),

        horizontalAlignment =
            Alignment.CenterHorizontally,

        verticalArrangement =
            Arrangement.Center
    ) {

        Card(
            modifier = Modifier.fillMaxWidth()
        ) {

            Text(
                text = statusText,
                modifier = Modifier.padding(16.dp)
            )
        }

        Button(
            onClick = onDispatchClick,
            modifier = Modifier.padding(top = 16.dp)
        ) {

            Text("Dispatch Cat Agent")
        }
    }
}
```
This screen should contain:

- A status display.
- A Dispatch Cat Agent button.
- Progress updates.
- Completion messages.

### Important

The composable does **not** communicate directly with WorkManager.

The composable only:

- Displays state.
- Raises button click events.

`MainActivity` handles all WorkManager interactions.

---

# Part 4 – Replace MainActivity Contents

Open:

```text
MainActivity.kt
```

Replace the existing content with the following code:
```kotlin
package com.example.catagenttracker

import android.Manifest
import android.content.pm.PackageManager
import android.os.Build
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.core.content.ContextCompat
import androidx.lifecycle.lifecycleScope
import androidx.work.OneTimeWorkRequestBuilder
import androidx.work.WorkInfo
import androidx.work.WorkManager
import androidx.work.workDataOf
import com.example.catagenttracker.ui.screen.CatAgentScreen
import com.example.catagenttracker.worker.CatDispatchWorker
import kotlinx.coroutines.flow.collectLatest
import kotlinx.coroutines.launch
import java.util.UUID

class MainActivity : ComponentActivity() {

    private var statusText by mutableStateOf(
        "Press Dispatch Cat Agent"
    )

    private val notificationPermissionLauncher =
        registerForActivityResult(
            ActivityResultContracts.RequestPermission()
        ) { granted ->

            if (granted) {
                startTracking()
            } else {
                statusText =
                    "Notification permission denied."
            }
        }

    override fun onCreate(
        savedInstanceState: Bundle?
    ) {
        super.onCreate(savedInstanceState)

        setContent {

            MaterialTheme {

                Surface {

                    CatAgentScreen(
                        statusText = statusText,
                        onDispatchClick = {
                            checkNotificationPermission()
                        }
                    )
                }
            }
        }
    }

    private fun checkNotificationPermission() {

        if (
            Build.VERSION.SDK_INT >=
            Build.VERSION_CODES.TIRAMISU
        ) {

            if (
                ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.POST_NOTIFICATIONS
                ) == PackageManager.PERMISSION_GRANTED
            ) {

                startTracking()

            } else {

                notificationPermissionLauncher.launch(
                    Manifest.permission.POST_NOTIFICATIONS
                )
            }

        } else {

            startTracking()
        }
    }

    private fun startTracking() {

        val inputData =
            workDataOf(
                CatDispatchWorker.KEY_CAT_AGENT_ID
                        to "CatAgent007"
            )

        val request =
            OneTimeWorkRequestBuilder<CatDispatchWorker>()
                .setInputData(inputData)
                .build()

        WorkManager.getInstance(this)
            .enqueue(request)

        observeWork(request.id)
    }

    private fun observeWork(id: UUID) {

        lifecycleScope.launch {

            WorkManager.getInstance(this@MainActivity)
                .getWorkInfoByIdFlow(id)
                .collectLatest { info ->

                    if (info != null) {

                        val seconds =
                            info.progress.getInt(
                                CatDispatchWorker.KEY_SECONDS_REMAINING,
                                10
                            )

                        val agentId =
                            info.progress.getString(
                                CatDispatchWorker.KEY_CAT_AGENT_ID
                            ) ?: "Unknown"

                        when (info.state) {

                            WorkInfo.State.RUNNING -> {

                                statusText =
                                    "Agent: $agentId\n" +
                                            "Remaining: $seconds seconds"
                            }

                            WorkInfo.State.SUCCEEDED -> {

                                statusText =
                                    "Cat Agent arrived at destination!"
                            }

                            WorkInfo.State.FAILED -> {

                                statusText =
                                    "Dispatch failed"
                            }

                            else -> {

                                statusText =
                                    info.state.toString()
                            }
                        }
                    }
                }
        }
    }
}
```
The activity is responsible for:

1. Requesting notification permissions.
2. Creating the WorkManager request.
3. Starting the worker.
4. Observing progress.
5. Updating Compose state.

---

# Part 5 – Understand the Data Flow

When the button is selected the following process occurs:

```text
Dispatch Cat Agent Button
          │
          ▼
checkNotificationPermission()
          │
          ▼
startTracking()
          │
          ▼
WorkManager
          │
          ▼
CatDispatchWorker
          │
          ▼
setProgress(...)
          │
          ▼
WorkInfo.progress
          │
          ▼
observeWork()
          │
          ▼
Compose UI Updated
```

Use this diagram when debugging the application.

---

# Part 6 – Run and Test

Run the application.

Expected behaviour:

## Initial State

```text
Press Dispatch Cat Agent
```

## Runtime State

```text
Agent: CatAgent007
Remaining: 10 seconds
```

The value should decrease once every second.

## Completion State

```text
Cat Agent arrived at destination!
```

---

# Part 7 – Verify Notification Behaviour

When tracking begins:

- A notification appears.
- The notification remains visible.
- The countdown updates every second.
- Selecting the notification opens MainActivity.

Example:

```text
Cat Agent Tracking
Agent CatAgent007 arriving in 8 seconds
```

---

# Common Errors and Fixes

| Error | Cause | Solution |
|---------|---------|---------|
| Unresolved reference createNotificationChannel | Method missing from worker | Copy the complete worker implementation |
| Unresolved reference getWorkInfoByIdFlow | WorkManager KTX missing | Add work-runtime-ktx dependency |
| Notification not shown | Permission not granted | Approve POST_NOTIFICATIONS |
| Foreground service error | Manifest entry missing | Add SystemForegroundService |
| CatAgentScreen unresolved | File missing or wrong package | Create file inside existing ui package |
| ServiceInfo unresolved | Missing import | Add android.app.ServiceInfo |

---

# Testing Checklist

| Test | Pass / Needs Work | Evidence / Notes |
|------|------|------|
| Notification channel created | | |
| Runtime notification permission handled | | |
| Notification appears and remains visible | | |
| Countdown updates every second | | |
| Notification opens MainActivity | | |
| Progress displayed in Compose UI | | |
| Agent ID displayed correctly | | |
| Completion feedback displayed | | |

---

# Reflection Questions

1. Why is this implemented as a foreground worker rather than an ordinary Worker?
2. What is the purpose of ForegroundInfo?
3. Why should notification updates avoid repeatedly alerting the user?
4. What is the relationship between setProgress(Data) and WorkInfo.progress?
5. What happens if POST_NOTIFICATIONS permission is denied?

---

