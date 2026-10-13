If you need a file that students can save directly as ProfilePageLab.md, remove the rich formatting and use pure Markdown syntax only, like this:

# Jetpack Compose Lab Exercise: Creating a Profile Page

## Overview

In this practical activity, you will build a simple profile page using Jetpack Compose. The exercise introduces key Compose layout components, including `Surface`, `Column`, `Row`, and `Box`, as well as images, buttons, modifiers, and previews.

By completing this lab, you will gain hands-on experience in creating modern Android user interfaces using the declarative UI approach of Jetpack Compose.

---

## Learning Objectives

By the end of this lab, you will be able to:

- Create and preview reusable `@Composable` functions.
- Use `Surface` and `Card` to create visually structured UI components.
- Arrange content using `Column`, `Row`, and `Box`.
- Display and style images with `Image`.
- Apply `Modifier` to control layout, spacing, shape, and alignment.
- Create interactive buttons.
- Build a complete profile interface.

---

## Prerequisites

Before starting:

- Android Studio installed.
- Jetpack Compose enabled.
- Material 3 enabled.
- A profile image ready to use.

---

# Step 1: Create a New Project

1. Open Android Studio.
2. Select **New Project**.
3. Choose **Empty Activity**.
4. Name the project:

```text
Profile
```

5. Ensure Jetpack Compose is enabled.
6. Finish project creation.

---

# Step 2: Create the Profile Composable

Open `MainActivity.kt` and add:

```kotlin
@Composable
fun Profile(modifier: Modifier = Modifier) {
    // Profile UI will be added here
}
```

Add a preview:

```kotlin
@Preview(showBackground = true)
@Composable
fun ProfilePreview() {
    Profile(
        modifier = Modifier.padding(20.dp)
    )
}
```

### Why Use Preview?

- View the UI without running the application.
- Quickly identify layout issues.
- Speed up development.

---

# Step 3: Create the Outer Surface

Replace the contents of `Profile()` with:

```kotlin
Surface(
    modifier = modifier
        .fillMaxWidth()
        .padding(16.dp),
    shape = RoundedCornerShape(16.dp),
    shadowElevation = 8.dp,
    color = Color.White
) {
    // Content goes here
}
```

### What Does This Do?

- `fillMaxWidth()` uses the available width.
- `padding(16.dp)` adds spacing around the component.
- `RoundedCornerShape(16.dp)` creates rounded corners.
- `shadowElevation = 8.dp` adds depth with a shadow.
- `color = Color.White` sets the background colour.

---

# Step 4: Add a Column Layout

Inside the `Surface`, add:

```kotlin
Column(
    horizontalAlignment = Alignment.CenterHorizontally,
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
) {
    // Profile content here
}
```

### Understanding Column

A `Column` arranges child components vertically:

```text
Item 1
Item 2
Item 3
```

---

# Step 5: Add the Profile Image

Before continuing, add an image named:

```text
cat.png
```

to:

```text
app/src/main/res/drawable
```

Then add:

```kotlin
Box(
    modifier = Modifier.size(116.dp)
) {
    Image(
        painter = painterResource(id = R.drawable.cat),
        contentDescription = "Profile picture",
        contentScale = ContentScale.Crop,
        modifier = Modifier
            .padding(8.dp)
            .size(100.dp)
            .clip(CircleShape)
    )

    Card(
        onClick = { },
        modifier = Modifier.align(Alignment.BottomEnd),
        border = BorderStroke(
            1.dp,
            SolidColor(Color.Blue)
        )
    ) {
        Text(
            text = "Edit",
            modifier = Modifier.padding(
                horizontal = 4.dp,
                vertical = 2.dp
            ),
            fontSize = 12.sp
        )
    }
}
```

### Key Concepts

- `Box` layers elements on top of each other.
- `Image` displays drawable resources.
- `CircleShape` creates a circular image.
- `ContentScale.Crop` fills the available image area.
- `Alignment.BottomEnd` positions the Edit card at the bottom-right corner.

---

# Step 6: Explore Modifier Order

Compare:

```kotlin
Modifier
    .padding(16.dp)
    .size(100.dp)
```

with:

```kotlin
Modifier
    .size(100.dp)
    .padding(16.dp)
```

### Observation

Modifier order matters because each modifier wraps the next one. Changing the order changes layout behaviour and available space.

---

# Step 7: Add Profile Information

Add the following below the image:

```kotlin
Spacer(
    modifier = Modifier.height(16.dp)
)

Text(
    text = "Jane Doe",
    fontSize = 24.sp,
    fontWeight = FontWeight.Bold,
    textAlign = TextAlign.Center,
    modifier = Modifier.padding(top = 16.dp)
)

Text(
    text = "Mobile Developer | Tech Enthusiast",
    fontSize = 16.sp,
    color = Color.Gray,
    textAlign = TextAlign.Center,
    modifier = Modifier.padding(vertical = 16.dp)
)
```

---

# Step 8: Add Action Buttons

Add:

```kotlin
Row(
    horizontalArrangement = Arrangement.SpaceEvenly,
    modifier = Modifier.fillMaxWidth()
) {

    Button(
        onClick = { }
    ) {
        Text("Follow")
    }

    Button(
        onClick = { }
    ) {
        Text("Message")
    }
}
```

### Understanding Row

A `Row` arranges components horizontally:

```text
[ Follow ]   [ Message ]
```

`Arrangement.SpaceEvenly` distributes the available space evenly.

---

# Step 9: Run the Application

Run the application on:

- An Android emulator, or
- A physical Android device.

### Expected Output

- Circular profile image
- Edit label
- User name
- User description
- Follow button
- Message button

---

# Challenge Activity: Make Follow Interactive

Inside `Profile()`, add:

```kotlin
var isFollowing by remember {
    mutableStateOf(false)
}
```

Replace the Follow button with:

```kotlin
Button(
    onClick = {
        isFollowing = !isFollowing
    }
) {
    Text(
        text = if (isFollowing)
            "Following"
        else
            "Follow"
    )
}
```

Required imports:

```kotlin
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
```

---

# Component Hierarchy

```text
Scaffold
└── Profile (Surface)
    └── Column
        ├── Box
        │   ├── Image
        │   └── Edit Card
        ├── Text (Name)
        ├── Text (Description)
        └── Row
            ├── Follow Button
            └── Message Button
```

---

# Practical Exercises

## Exercise 1: Personalise the Profile

Change:

- Name
- Job title
- Profile image

## Exercise 2: Change the Image Shape

Replace:

```kotlin
CircleShape
```

with:

```kotlin
RoundedCornerShape(16.dp)
```

## Exercise 3: Add a Follower Count

Add:

```kotlin
Text(
    text = "1,250 Followers"
)
```

## Exercise 4: Add Additional Information

Add:

- Location
- Email
- Website

## Exercise 5: Improve the Design

Experiment with:

- Dividers
- Colours
- Typography styles
- Icons
- Spacing

---

# Reflection Questions

1. What is the difference between `Column`, `Row`, and `Box`?
2. Why is modifier order important?
3. What role does `Surface` play?
4. Why should images include a `contentDescription`?
5. What is recomposition in Jetpack Compose?

---

# Key Takeaways

- `Surface` creates visual containers.
- `Column`, `Row`, and `Box` are core layout components.
- `Modifier` controls layout and appearance.
- `Image` displays drawable resources.
- `@Preview` supports rapid UI development.
- State enables interactivity.
- Small composables can be combined to create complex interfaces.

---

# Extension Activity

Create a Business Dashboard app that includes:

- Revenue Card
- Expenses Card
- Profit Card
- Monthly Statistics
- Action Buttons

Use the same Compose concepts (`Surface`, `Column`, `Row`, `Box`, and `Modifier`) introduced in this lab.


You can save the above directly as ProfilePageLab.md and upload it to Moodle.
