Jetpack Compose Lab Exercise: Creating a Profile Page
Overview

In this practical activity, you will build a simple profile page using Jetpack Compose. The exercise introduces key Compose layout components, including Surface, Column, Row, and Box, as well as images, buttons, modifiers, and previews.

By completing this lab, you will gain hands-on experience in creating modern Android user interfaces using the declarative UI approach of Jetpack Compose.

Learning Objectives

By the end of this lab, you will be able to:

Create and preview reusable @Composable functions.
Use Surface and Card to create visually structured UI components.
Arrange content using Column, Row, and Box.
Display and style images with Image.
Apply Modifier to control layout, spacing, shape, and alignment.
Create interactive buttons.
Build a complete profile interface.
Prerequisites

Before starting:

Install Android Studio (latest stable version).
Ensure your project is configured to use:
Jetpack Compose
Material 3
Have an image ready for the profile picture.
Step 1: Create a New Project
Open Android Studio.
Select New Project.
Choose Empty Activity.
Name the project:
Profile

Ensure Jetpack Compose is enabled.
Finish project creation.
Step 2: Create the Profile Composable

Open MainActivity.kt and create a reusable composable:

@Composable
fun Profile(modifier: Modifier = Modifier) {
    // Profile UI will be added here
}


Add a preview function:

@Preview(showBackground = true)
@Composable
fun ProfilePreview() {
    Profile(
        modifier = Modifier.padding(20.dp)
    )
}

Why use Preview?

The @Preview annotation allows you to:

View your UI without running the application.
Detect layout issues quickly.
Speed up development.
Step 3: Create the Outer Surface

Replace the contents of Profile() with:

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

Understanding the Code
Component	PurposefillMaxWidth()	Uses the available width
padding(16.dp)	Adds spacing around the surface
RoundedCornerShape(16.dp)	Creates rounded corners
shadowElevation = 8.dp	Adds a shadow effect
Color.White	Sets the background colour
Step 4: Add a Column Layout

Inside the Surface, add:

Column(
    horizontalAlignment = Alignment.CenterHorizontally,
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
) {
    // Profile content here
}

What is a Column?

A Column arranges items:

Vertically
From top to bottom

Example:

Item 1
Item 2
Item 3

Step 5: Add a Profile Image

Before continuing:

✅ Add an image called:

cat.png


to:

app/src/main/res/drawable


Now add the following code inside the Column:

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
        onClick = {
            // TODO
        },
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

Important Concepts
Box

Box allows components to be layered on top of one another.

+----------------+
|    Image       |
|            Edit|
+----------------+

CircleShape

Creates a circular image:

.clip(CircleShape)

ContentScale.Crop

Ensures that the image fills the available space while maintaining its proportions.

Step 6: Understanding Modifier Order

Compare:

Modifier
    .padding(16.dp)
    .size(100.dp)


with:

Modifier
    .size(100.dp)
    .padding(16.dp)

Key Point

Modifier order matters.

Modifiers are applied sequentially and can change:

Size
Layout
Available drawing space

Experiment with both versions and observe the differences.

Step 7: Add Profile Information

Below the Box, add:

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

Result

The page should now display:

Profile picture
Name
Professional title
Step 8: Add Action Buttons

Add the following underneath the profile description:

Row(
    horizontalArrangement = Arrangement.SpaceEvenly,
    modifier = Modifier.fillMaxWidth()
) {

    Button(
        onClick = {
            // TODO
        }
    ) {
        Text("Follow")
    }

    Button(
        onClick = {
            // TODO
        }
    ) {
        Text("Message")
    }
}

Understanding Row

A Row places items horizontally.

[ Follow ]     [ Message ]


Arrangement.SpaceEvenly distributes the available space evenly.

Step 9: Run the Application

Run the application on:

An Android emulator, or
A physical Android device.

You should see:

✅ Circular profile picture

✅ Edit badge

✅ User name

✅ Description

✅ Follow button

✅ Message button

Challenge Activity: Make Follow Interactive

Add state to your composable:

var isFollowing by remember {
    mutableStateOf(false)
}


Replace the Follow button with:

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

Required Imports
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue

Expected Behaviour
Click	Button TextInitial State	Follow
First Click	Following
Second Click	Follow
Component Hierarchy

The application structure is:

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

Practical Exercises

Complete the following tasks:

Exercise 1: Personalise the Profile

Change:

Name
Job title
Profile image
Exercise 2: Change the Image Shape

Replace:

CircleShape


with:

RoundedCornerShape(16.dp)


Observe the difference.

Exercise 3: Add a Follower Count

Add a new Text component:

Text(
    text = "1,250 Followers"
)


Position it appropriately within the column.

Exercise 4: Add More Information

Add:

Location
Email
Personal website

using additional Text composables.

Exercise 5: Improve the Design

Try:

Adding a divider.
Changing colours.
Using MaterialTheme.typography.
Increasing spacing.
Using icons.
Reflection Questions
What is the difference between Column, Row, and Box?
Why is Modifier order important?
What role does Surface play?
Why should images have a contentDescription?
What is recomposition in Jetpack Compose?
Key Takeaways
Surface creates visual containers with elevation and shape.
Column, Row, and Box are the fundamental layout components in Compose.
Modifier controls size, spacing, alignment, and appearance.
Image displays drawable resources.
@Preview accelerates UI development.
State allows Compose UIs to become interactive.
Small composables can be combined to create complex interfaces.
Extension Activity

Using the same techniques, create a Business Dashboard App containing:

Revenue Card
Expenses Card
Profit Card
Monthly Statistics
Action Buttons

Apply the same Compose building blocks (Surface, Column, Row, and Box) used in this profile page exercise.
