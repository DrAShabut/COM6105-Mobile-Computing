# Lab 3: Cancel Ongoing Work (Jetpack Compose)

## Prerequisite

Complete Lab 2: Foreground Worker and Live Notification.

In Lab 2 you implemented a foreground worker that:

- Displayed a live notification.
- Tracked a Cat Agent deployment.
- Published progress updates using `setProgress()`.
- Updated the Compose UI using `WorkInfo.progress`.

In this lab, you will extend the application so that users can cancel a deployment before the Cat Agent reaches its destination.

## Learning Outcomes

By the end of this lab you will be able to:

- Store a `WorkRequest` UUID.
- Cancel a running `WorkRequest`.
- Use `cancelWorkById()`.
- Handle the states `RUNNING`, `SUCCEEDED`, `CANCELLED`, and `FAILED`.
- Update a Jetpack Compose UI when work is cancelled.

## Scenario

A Cat Agent has been dispatched.

The user now requires the ability to abort the mission before the countdown reaches zero.

When a deployment is cancelled:

- The foreground worker stops.
- Progress updates stop.
- The notification disappears.
- The UI displays **Deployment cancelled!**

## Existing Application Structure

```text
com.example.catagenttracker
│
├── MainActivity.kt
│
├── ui
│   └── CatAgentScreen.kt
│
└── worker
    └── CatDispatchWorker.kt
```

No new files are required for this lab.

## Part 1 – Store the WorkRequest ID

### Step 1.1

Open **MainActivity.kt**.

Add:

```kotlin
import java.util.UUID
```

Inside the `MainActivity` class add:

```kotlin
private var trackingWorkId: UUID? = null
```

### Why Is This Needed?

Every WorkManager request has a unique identifier.

WorkManager requires this identifier when cancelling a specific piece of work.

## Part 2 – Save the WorkRequest ID

Locate:

```kotlin
private fun startTracking()
```

Replace the method with:

```kotlin
private fun startTracking() {
    val inputData =
        workDataOf(
            CatDispatchWorker.KEY_CAT_AGENT_ID to "CatAgent007"
        )

    val request =
        OneTimeWorkRequestBuilder<CatDispatchWorker>()
            .setInputData(inputData)
            .build()

    trackingWorkId = request.id

    WorkManager.getInstance(this)
        .enqueue(request)

    observeWork(request.id)
}
```

### Checkpoint

Verify that:

```kotlin
trackingWorkId = request.id
```

appears before:

```kotlin
WorkManager.getInstance(this)
    .enqueue(request)
```

The application should still compile and run.

## Part 3 – Create the Cancellation Function

Inside `MainActivity` add:

```kotlin
private fun cancelTracking() {
    trackingWorkId?.let {
        WorkManager.getInstance(this)
            .cancelWorkById(it)
    }
}
```

### How It Works

The safe-call operator:

```kotlin
trackingWorkId?.let
```

ensures that cancellation only occurs when a valid request ID exists.

## Part 4 – Update the Compose Screen

Open **CatAgentScreen.kt**.

Replace the existing composable with:

```kotlin
@Composable
fun CatAgentScreen(
    statusText: String,
    onDispatchClick: () -> Unit,
    onCancelClick: () -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Card(
            modifier = Modifier.fillMaxWidth()
        ) {
            Text(
                text = statusText,
                modifier = Modifier.padding(16.dp)
            )
        }

        Spacer(
            modifier = Modifier.height(16.dp)
        )

        Button(
            onClick = onDispatchClick
        ) {
            Text("Dispatch Cat Agent")
        }

        Spacer(
            modifier = Modifier.height(8.dp)
        )

        Button(
            onClick = onCancelClick
        ) {
            Text("Cancel Deployment")
        }
    }
}
```

### Additional Imports

```kotlin
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.height
```

## Part 5 – Connect the Cancel Button

Open **MainActivity.kt**.

Replace the existing `CatAgentScreen()` call with:

```kotlin
CatAgentScreen(
    statusText = statusText,
    onDispatchClick = {
        checkNotificationPermission()
    },
    onCancelClick = {
        cancelTracking()
    }
)
```

## Part 6 – Handle Cancellation State

Locate:

```kotlin
when (info.state)
```

Replace with:

```kotlin
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

    WorkInfo.State.CANCELLED -> {
        statusText =
            "Deployment cancelled!"
    }

    WorkInfo.State.FAILED -> {
        statusText =
            "Deployment failed!"
    }

    else -> {
        statusText = info.state.toString()
    }
}
```

## Part 7 – Run the Application

### Test 1 – Successful Deployment

1. Select **Dispatch Cat Agent**.
2. Allow the countdown to finish.

**Expected Outcome:**

`Cat Agent arrived at destination!`

### Test 2 – Cancel Deployment

1. Select **Dispatch Cat Agent**.
2. Wait 3–5 seconds.
3. Select **Cancel Deployment**.

**Expected Outcome:**

- Deployment cancelled!
- The foreground notification disappears.

### Test 3 – Cancel Without Active Work

1. Launch the application.
2. Immediately select **Cancel Deployment**.

**Expected Outcome:**

- No crash.
- No work cancelled.
- Application remains responsive.

## Understanding the Flow

```text
Dispatch Button
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
      │
      ▼
Cancel Deployment
      │
      ▼
cancelWorkById()
      │
      ▼
WorkInfo.State.CANCELLED
      │
      ▼
Deployment cancelled!
```

## Test Log

| Test | Expected Outcome | Observed Outcome |
|------|------------------|------------------|
| Start and allow countdown to finish | Success message appears | |
| Start and cancel before completion | Cancellation message appears | |
| Notification disappears after cancellation | Yes | |
| Press Cancel when no request ID exists | No crash | |

## Reflection Questions

1. Why does cancellation require the WorkRequest UUID?
2. Why does WorkManager documentation state that cancellation is not guaranteed?
3. Why should the UI distinguish `SUCCEEDED` from `CANCELLED`?
4. What role does `WorkInfo` play when monitoring long-running tasks?
5. What happens if a cancellation request is made after the work has already completed?

## Extension Activity – My Water Tracker

### Requirements

- Create an Empty Activity project named **My Water Tracker**.
- Add WorkManager dependencies.
- Add notification permissions.
- Register `SystemForegroundService`.
- Create `WaterConsumptionWorker`.
- Store water values using `AtomicInteger`.
- Create a foreground notification.
- Request notification permissions.
- Publish progress updates.
- Decrease the water balance by 0.144 ml every 5 seconds.
- Add a **Drank a Glass of Water** button.
- Add a **Cancel Tracking** button.

### Planning and Calculation Space

| Prompt | Your Response |
|----------|----------|
| How will you convert millilitres to microlitres? | |
| How will the worker publish the current balance? | |
| How will the activity update its displayed balance? | |
| How will the glass-of-water action update the shared value safely? | |
| How will cancellation be reflected in the UI? | |

### Test Log

| Test | Expected Behaviour | Observed Result |
|------|-------------------|----------------|
| Start tracking | Foreground notification and balance updates begin | |
| Wait for 5 seconds | Balance decreases by 0.144 ml | |
| Tap Drank a Glass of Water | Balance increases by 250 ml | |
| Cancel | Ongoing work is cancelled | |
| Reopen / relaunch app | Record what state is retained or reset | |

## Review and Assessment

Explain:

- The difference between deferrable background work and user-visible foreground work.
- The roles of WorkManager, Worker, WorkRequest, Data, and WorkInfo.
- How a notification can launch an activity.
- Android version-specific considerations for notification permissions.
- Android version-specific considerations for foreground service permissions.
- The difference between `cancelWorkById()` and `updateWork()`.
- Why the water level is stored in microlitres rather than whole millilitres.

## Success Criteria

To successfully complete this lab you must demonstrate:

- Correct use of `cancelWorkById()`.
- Storage of the WorkRequest UUID.
- Correct handling of `SUCCEEDED` and `CANCELLED`.
- Working cancellation behaviour.
- Foreground notification removal after cancellation.
- Correct Compose UI updates.

**Lab Complete.**
