# Android Lifecycle Demo

## 🎯 Aim

In this activity, you will build a simple Android app that demonstrates the **Android Activity Lifecycle**.

The app will write messages to **Logcat** whenever a lifecycle method is called, allowing you to see how Android manages applications behind the scenes.

---

## ✅ Learning Outcomes

By completing this task, you should be able to:

- Explain the purpose of the Android Activity Lifecycle.
- Identify common lifecycle methods.
- Use Logcat to monitor app behaviour.
- Observe how Android responds when an app starts, pauses, resumes, and closes.

---

## 📋 What You Need

- Android Studio installed
- An Android Emulator (AVD) or physical Android device
- Basic knowledge of creating an Android project

---

# Part 1: Create the Project

Create a new Android Studio project with the following settings:

- **Template:** Empty Activity
- **Project Name:** Lifecycle Demo
- **Language:** Kotlin

Once the project has been created, open:

```text
MainActivity.kt
```

---

# Part 2: Add the Lifecycle Code

Replace the contents of `MainActivity.kt` with the code below.

```kotlin
package com.example.lifecycledemo

import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Log.d("Lifecycle Demo", "onCreate called")
    }

    override fun onStart() {
        super.onStart()

        Log.d("Lifecycle Demo", "onStart called")
    }

    override fun onResume() {
        super.onResume()

        Log.d("Lifecycle Demo", "onResume called")
    }

    override fun onPause() {
        super.onPause()

        Log.d("Lifecycle Demo", "onPause called")
    }

    override fun onStop() {
        super.onStop()

        Log.d("Lifecycle Demo", "onStop called")
    }

    override fun onDestroy() {
        super.onDestroy()

        Log.d("Lifecycle Demo", "onDestroy called")
    }
}
```

---

## 🔍 What Does This Code Do?

Each function is automatically called by Android when the app changes state.

| Method | When It Is Called |
|----------|------------------|
| `onCreate()` | App is created |
| `onStart()` | App becomes visible |
| `onResume()` | App is ready for user interaction |
| `onPause()` | App loses focus |
| `onStop()` | App is no longer visible |
| `onDestroy()` | App is being destroyed |

---

# Part 3: Run the App

1. Start your emulator.
2. Click **Run ▶**.
3. Open **Logcat** at the bottom of Android Studio.
4. In the search box, type:

```text
Lifecycle Demo
```

You should see:

```text
onCreate called
onStart called
onResume called
```

---

## 🤔 Why Did This Happen?

When an app starts, Android follows this sequence:

```text
onCreate()
    ↓
onStart()
    ↓
onResume()
```

Your app is now running and ready for the user to interact with it.

---

# Experiment 1: Press the Home Button

## Instructions

1. Run the app.
2. Press the Home button on the emulator.

## Expected Output

```text
onPause called
onStop called
```

### What Happened?

The app moved to the background.

The app still exists in memory, so Android does **not** call:

```text
onDestroy()
```

---

# Experiment 2: Return to the App

## Instructions

1. Open the App Drawer.
2. Launch Lifecycle Demo again.

## Expected Output

```text
onStart called
onResume called
```

### What Happened?

Android reused the existing Activity instead of creating a new one.

Notice that:

```text
onCreate called
```

does not appear again.

---

# Experiment 3: Close the App from Recent Apps

## Instructions

1. Open the Recent Apps screen.
2. Swipe Lifecycle Demo away.

## Expected Output

```text
onPause called
onStop called
```

You may also see:

```text
PROCESS ENDED
```

### What Happened?

Android terminated the process before calling:

```text
onDestroy()
```

This is why developers should never rely solely on `onDestroy()` to save important data.

---

# Experiment 4: Trigger onDestroy()

## Instructions

1. Run the app again.
2. Press the **Back** button.

## Expected Output

```text
onPause called
onStop called
onDestroy called
```

### What Happened?

The Activity was properly closed, so Android destroyed it.

---

# Experiment 5: Close the App Programmatically

Add the following line inside `onResume()`:

```kotlin
finish()
```

Example:

```kotlin
override fun onResume() {
    super.onResume()

    Log.d("Lifecycle Demo", "onResume called")

    finish()
}
```

Run the app again.

## Expected Output

```text
onCreate called
onStart called
onResume called
onPause called
onStop called
onDestroy called
```

### What Happened?

The `finish()` method tells Android to close the Activity.

---

# Activity Lifecycle Diagram

```text
        onCreate()
             │
             ▼
         onStart()
             │
             ▼
        onResume()
             │
             ▼
       User Uses App
             │
             ▼
         onPause()
             │
             ▼
          onStop()
             │
     ┌───────┴────────┐
     │                │
     ▼                ▼
 onRestart()      onDestroy()
     │
     ▼
 onStart()
     │
     ▼
 onResume()
```

---

# 🎯 Challenge

Modify the code so that each lifecycle callback writes a custom message:

Example:

```kotlin
Log.d("Lifecycle Demo", "Application Started")
```

instead of:

```kotlin
Log.d("Lifecycle Demo", "onStart called")
```

Run the app and observe how the output changes.

---

# 📝 Reflection Questions

1. Which lifecycle methods are called when the app starts?
2. What happens when you press the Home button?
3. Why is `onCreate()` not called when returning to an already-running app?
4. When is `onDestroy()` called?
5. Why is `onDestroy()` not always guaranteed to execute?

---

# 📸 Evidence to Submit

Upload screenshots showing:

- The application running successfully.
- Logcat displaying lifecycle messages.
- Output after pressing the Home button.
- Output after pressing the Back button.
- Your completed `MainActivity.kt`.

---

## Key Takeaway

Android applications do not run continuously. The Android operating system controls the lifecycle of each Activity and automatically calls lifecycle methods as the user interacts with the app. Understanding these callbacks is essential for building reliable Android applications.
