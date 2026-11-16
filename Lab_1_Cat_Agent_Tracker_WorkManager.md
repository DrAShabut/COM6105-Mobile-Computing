# Lab: Build a Cat Agent Tracker with WorkManager

## Lab overview

In this lab, you will build an Android app that prepares a fictional cat
agent for a mission. WorkManager will run four background tasks in
sequence:

1.  Stretching
2.  Grooming
3.  Litter-box inspection
4.  Suiting up

Each task simulates three seconds of work. When a task succeeds, the app
displays a Toast message.

By the end of the lab, you should be able to create custom workers, read
input data, return output data, apply constraints, chain work requests,
and observe work status.

## Requirements

-   Android Studio and a Kotlin Android project
-   An Android emulator or physical Android device
-   An internet connection on the device

**Note:** The app requires a connected network because each request uses
`NetworkType.CONNECTED`. The workers do not actually access the
internet; the delay simply simulates background work.

## Part 1 --- Create the project

1.  In Android Studio, select **New Project**.
2.  Choose **Empty Views Activity** .
3.  Name the project `CatAgentTracker`.
4.  Set the package name to `com.example.catagenttracker`.
5.  Select Kotlin and finish creating the project.
6.  Wait for the initial Gradle sync to finish.

If you choose a different package name, update all package declarations
and imports accordingly.

## Part 2 --- Add dependencies

Open the app-level `app/build.gradle.kts` file and add these lines
inside `dependencies { }`:

``` kotlin
implementation("androidx.work:work-runtime-ktx:2.10.5")
implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.9.4")
```

Click **Sync Now** and wait for Gradle sync to complete. Make sure
`google()` and `mavenCentral()` are configured in the project's
dependency repositories.

## Part 3 --- Create the worker package

In the Project panel, navigate to
`app > java > com.example.catagenttracker`.

1.  Right-click `com.example.catagenttracker`.
2.  Select **New \> Package**.
3.  Name the package `worker`.

Create these five Kotlin files inside `worker`:

-   `CatAgentWorker.kt`
-   `CatStretchingWorker.kt`
-   `CatFurGroomingWorker.kt`
-   `CatLitterBoxSittingWorker.kt`
-   `CatSuitUpWorker.kt`

**Important:** Put each class in its own file. Do not paste all five
classes into one file.

## Part 4 --- Create the shared worker

Open `CatAgentWorker.kt` and enter:

``` kotlin
package com.example.catagenttracker.worker

import android.content.Context
import androidx.work.Worker
import androidx.work.WorkerParameters
import androidx.work.workDataOf

abstract class CatAgentWorker(
    context: Context,
    workerParameters: WorkerParameters,
    private val taskName: String
) : Worker(context, workerParameters) {

    override fun doWork(): Result {

        // 1. Read the input value
        val catAgentId = inputData.getString(KEY_CAT_AGENT_ID)
            ?: return Result.failure()

        return try {

            // 2. Perform the background operation
            Thread.sleep(3000L)

            // 3. Build output data
            val outputData = workDataOf(
                KEY_CAT_AGENT_ID to catAgentId,
                KEY_COMPLETED_TASK to taskName
            )

            // 4. Return success and pass the output onward
            Result.success(outputData)

        } catch (exception: InterruptedException) {

            // The work was interrupted
            Thread.currentThread().interrupt()
            Result.failure()
        }
    }

    companion object {
        const val KEY_CAT_AGENT_ID = "catAgentId"
        const val KEY_COMPLETED_TASK = "completedTask"
    }
}
```

### Understand this class

-   `CatAgentWorker` is an abstract base class shared by the four task
    workers.
-   `inputData` contains the values supplied to a work request.
-   `getString(KEY_CAT_AGENT_ID)` reads the cat agent identifier.
-   If the identifier is missing, the worker returns `Result.failure()`.
-   `Thread.sleep(3000L)` simulates a three-second operation.
-   `workDataOf(...)` creates the output data.
-   `Result.success(outputData)` marks the work as successful and
    supplies output data to the next request in the chain.

The companion object stores the data keys so that the same key strings
are used consistently.

## Part 5 --- Create the four task workers

Each class extends `CatAgentWorker` and passes a task name to the base
class.

### 5.1 CatStretchingWorker.kt

``` kotlin
package com.example.catagenttracker.worker

import android.content.Context
import androidx.work.WorkerParameters

class CatStretchingWorker(
    context: Context,
    params: WorkerParameters
) : CatAgentWorker(context, params, "Stretching")
```

### 5.2 CatFurGroomingWorker.kt

``` kotlin
package com.example.catagenttracker.worker

import android.content.Context
import androidx.work.WorkerParameters

class CatFurGroomingWorker(
    context: Context,
    params: WorkerParameters
) : CatAgentWorker(context, params, "Grooming")
```

### 5.3 CatLitterBoxSittingWorker.kt

``` kotlin
package com.example.catagenttracker.worker

import android.content.Context
import androidx.work.WorkerParameters

class CatLitterBoxSittingWorker(
    context: Context,
    params: WorkerParameters
) : CatAgentWorker(context, params, "Litter box")
```

### 5.4 CatSuitUpWorker.kt

``` kotlin
package com.example.catagenttracker.worker

import android.content.Context
import androidx.work.WorkerParameters

class CatSuitUpWorker(
    context: Context,
    params: WorkerParameters
) : CatAgentWorker(context, params, "Suiting up")
```

### Checkpoint

Your package should now look like this:

``` text
com.example.catagenttracker
├── MainActivity.kt
└── worker
    ├── CatAgentWorker.kt
    ├── CatStretchingWorker.kt
    ├── CatFurGroomingWorker.kt
    ├── CatLitterBoxSittingWorker.kt
    └── CatSuitUpWorker.kt
```

Build the project now and resolve any package or import errors before
continuing.

## Part 6 --- Configure MainActivity

Open `MainActivity.kt` and replace its contents with:

``` kotlin
package com.example.catagenttracker

import android.os.Bundle
import android.widget.Toast
import androidx.activity.ComponentActivity
import androidx.lifecycle.lifecycleScope
import androidx.work.Constraints
import androidx.work.NetworkType
import androidx.work.OneTimeWorkRequestBuilder
import androidx.work.WorkManager
import androidx.work.WorkInfo
import androidx.work.getWorkInfoByIdFlow
import androidx.work.workDataOf
import com.example.catagenttracker.worker.CatAgentWorker
import com.example.catagenttracker.worker.CatFurGroomingWorker
import com.example.catagenttracker.worker.CatLitterBoxSittingWorker
import com.example.catagenttracker.worker.CatStretchingWorker
import com.example.catagenttracker.worker.CatSuitUpWorker
import kotlinx.coroutines.launch

class MainActivity : ComponentActivity() {

    private lateinit var workManager: WorkManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        workManager = WorkManager.getInstance(this)

        // Require an internet connection
        val networkConstraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()

        // Cat agent identifier
        val catAgentId = "CatAgent1"

        // 1. Stretching request
        val stretchingRequest =
            OneTimeWorkRequestBuilder<CatStretchingWorker>()
                .setConstraints(networkConstraints)
                .setInputData(
                    workDataOf(
                        CatAgentWorker.KEY_CAT_AGENT_ID to catAgentId
                    )
                )
                .build()

        // 2. Grooming request
        val groomingRequest =
            OneTimeWorkRequestBuilder<CatFurGroomingWorker>()
                .setConstraints(networkConstraints)
                .build()

        // 3. Litter box request
        val litterBoxRequest =
            OneTimeWorkRequestBuilder<CatLitterBoxSittingWorker>()
                .setConstraints(networkConstraints)
                .build()

        // 4. Suit up request
        val suitUpRequest =
            OneTimeWorkRequestBuilder<CatSuitUpWorker>()
                .setConstraints(networkConstraints)
                .build()

        // Chain the four requests in sequence
        workManager.beginWith(stretchingRequest)
            .then(groomingRequest)
            .then(litterBoxRequest)
            .then(suitUpRequest)
            .enqueue()

        // Observe each worker
        observeWork(stretchingRequest.id, "Agent finished stretching")
        observeWork(groomingRequest.id, "Agent finished grooming")
        observeWork(litterBoxRequest.id, "Agent finished litter box")
        observeWork(suitUpRequest.id, "Agent is suited up. Ready to go!")
    }

    private fun observeWork(
        requestId: java.util.UUID,
        message: String
    ) {
        lifecycleScope.launch {
            workManager.getWorkInfoByIdFlow(requestId)
                .collect { info ->
                    if (info?.state == WorkInfo.State.SUCCEEDED) {
                        Toast.makeText(
                            this@MainActivity,
                            message,
                            Toast.LENGTH_SHORT
                        ).show()
                    }
                }
        }
    }
}
```

**Important import:** `getWorkInfoByIdFlow` is an extension function.
Make sure this import is present:

``` kotlin
import androidx.work.getWorkInfoByIdFlow
```

If Android Studio cannot resolve it, check the `work-runtime-ktx`
dependency and run Gradle sync again.

### Understand MainActivity

1.  `WorkManager.getInstance(this)` obtains the WorkManager instance.
2.  `Constraints.Builder()` creates a network constraint.
3.  Four one-time work requests are created.
4.  The first request receives `CatAgent1` as input.
5.  `beginWith()` and `then()` connect the requests into a sequential
    chain.
6.  `enqueue()` submits the chain to WorkManager.
7.  `observeWork()` observes each request and displays a Toast when it
    succeeds.

The later requests do not set their own input data. WorkManager passes
the successful output from one request to the next request in the chain.
The cat agent identifier is therefore carried forward.

## Part 7 --- Build and run the app

1.  Save all Kotlin files.
2.  Select **Build \> Make Project**.
3.  Fix any compilation errors shown in the Build window.
4.  Start an emulator or connect an Android device.
5.  Confirm that the device has an internet connection.
6.  Click **Run** in Android Studio.
7.  Wait for the app to start and the work chain to execute.

You should see these Toast messages, in order:

1.  `Agent finished stretching`
2.  `Agent finished grooming`
3.  `Agent finished litter box`
4.  `Agent is suited up. Ready to go!`

Each worker simulates approximately three seconds of work. The four
tasks therefore take at least about twelve seconds in total, plus
WorkManager scheduling and execution overhead.

**Running the app again:** This version enqueues a new chain whenever
`MainActivity.onCreate()` runs. Relaunching the app or recreating the
activity can enqueue another chain.

## Part 8 --- Understand the sequence

``` text
CatStretchingWorker
        |
        v
CatFurGroomingWorker
        |
        v
CatLitterBoxSittingWorker
        |
        v
CatSuitUpWorker
```

The next worker starts after the preceding worker completes
successfully. If a worker returns `Result.failure()`, dependent work
will not continue as a successful chain. If it returns `Result.retry()`,
WorkManager schedules another attempt according to the configured
backoff policy.

## Part 9 --- Understand the Result options

The supplied worker returns `Result.success(outputData)` when the task
completes.

  -----------------------------------------------------------------------
  Return value                        Meaning
  ----------------------------------- -----------------------------------
  `Result.success()`                  The work completed successfully,
                                      without output data.

  `Result.success(outputData)`        The work completed successfully and
                                      provides output data.

  `Result.failure()`                  The work failed permanently;
                                      dependent work will not proceed as
                                      successful work.

  `Result.retry()`                    The work should be tried again
                                      later, subject to its backoff
                                      configuration.
  -----------------------------------------------------------------------

A temporary problem might justify `Result.retry()`. A permanent problem,
such as missing required input, should normally be treated as a failure.

## Part 10 --- Optional: configure a backoff policy

Add these imports where you configure the work request:

``` kotlin
import androidx.work.BackoffPolicy
import java.util.concurrent.TimeUnit
```

Add the following when building a request:

``` kotlin
.setBackoffCriteria(
    BackoffPolicy.LINEAR,
    10,
    TimeUnit.SECONDS
)
```

For example, the stretching request can be configured as follows:

``` kotlin
val stretchingRequest =
    OneTimeWorkRequestBuilder<CatStretchingWorker>()
        .setConstraints(networkConstraints)
        .setInputData(
            workDataOf(
                CatAgentWorker.KEY_CAT_AGENT_ID to catAgentId
            )
        )
        .setBackoffCriteria(
            BackoffPolicy.LINEAR,
            10,
            TimeUnit.SECONDS
        )
        .build()
```

This sets a linear backoff beginning at ten seconds for retries of this
request. The worker must return `Result.retry()` for WorkManager to
schedule a retry. Setting a backoff policy alone does not trigger a
retry.

## Your activities

After the app runs successfully, complete these tasks:

1.  Change the cat agent identifier from `CatAgent1` to `CatAgent007`.
    Run the app and confirm that the chain still completes.
2.  Change the simulated work duration in `CatAgentWorker` from `3000L` to `5000L`. Observe
    how the total execution time changes.
3.  Change one task name, such as `"Grooming"` to `"Fur maintenance"`.
    Confirm that the task still runs.
4.  Temporarily remove the `catAgentId` input from the stretching
    request by locating the .setInputData(...) section and remove it. 
    Observe what happens when the first worker cannot find its
    required input.
5.  Explain why the grooming worker does not start at the same time as
    the stretching worker.
6.  Explain how the cat agent identifier reaches the later workers even
    though only the first request explicitly sets input data.
7.  Explain the difference between `Result.failure()` and
    `Result.retry()`.

## Troubleshooting

  -------------------------------------------------------------------------------------------
  Problem                                       What to check
  --------------------------------------------- ---------------------------------------------
  `Unresolved reference: work`                  Confirm the WorkManager dependency is in the
                                                app-level Gradle file and Gradle sync
                                                completed.

  `Unresolved reference: getWorkInfoByIdFlow`   Confirm `work-runtime-ktx` is included and
                                                add
                                                `import androidx.work.getWorkInfoByIdFlow`.

  `Unresolved reference: lifecycleScope`        Confirm the Lifecycle KTX dependency is
                                                included and Gradle has synced.

  Workers remain enqueued                       Check that the emulator/device has a network
                                                connection. The requests require
                                                `NetworkType.CONNECTED`.

  A worker fails immediately                    Check that the first request supplies
                                                `catAgentId` and that the key matches
                                                `KEY_CAT_AGENT_ID`.

  Toast messages do not appear                  Keep the app in the foreground and inspect
                                                the WorkManager state in Android Studio's
                                                Logcat.

  A class or package is unresolved              Check package declarations, imports, and that
                                                each class is in its own file.
  -------------------------------------------------------------------------------------------

## Final checklist

-   [ ] The app builds without errors.
-   [ ] The four task workers extend `CatAgentWorker`.
-   [ ] The first request supplies the cat agent identifier.
-   [ ] Each worker reads its input from `inputData`.
-   [ ] Each successful worker returns output using `workDataOf(...)`.
-   [ ] The work requests are chained in the correct order.
-   [ ] The network constraint is applied to all four requests.
-   [ ] The app displays the four completion Toast messages in order.
