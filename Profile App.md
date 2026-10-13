Jetpack Compose Lesson: Creating a Profile Page
This lesson guides students through building a profile card in Jetpack Compose. It introduces the core layout composables—Surface, Column, Box, and Row—along with images, buttons, modifiers, spacing, and previews.
Learning objectives
By the end of the exercise, students should be able to:
- Create and preview a reusable @Composable function.
- Use Surface and Card to create a profile panel.
- Arrange content with Column, Row, and Box.
- Display and clip an image with Image.
- Use Modifier to control size, padding, shape, alignment, and click behavior.
- Build a profile interface with Follow and Message buttons.
1. Create the project
1. Open Android Studio and select New Project.
2. Choose Empty Activity.
3. Name the project Profile.
4. Make sure the project uses Jetpack Compose and Material 3.
We will build a profile card for a fictional user called Jane Doe.
2. Create the Profile composable and preview
Start by creating a composable named Profile:
@Composable
fun Profile(modifier: Modifier = Modifier) {
    // Profile UI will go here
}


Add a preview underneath it:
@Preview(showBackground = true)
@Composable
fun ProfilePreview() {
    Profile(
        modifier = Modifier.padding(20.dp)
    )
}


The preview lets you inspect the UI in Android Studio without running the app. The optional modifier parameter also allows the caller to control how the profile is positioned.
3. Create the card's outer surface
Use Surface to provide a white background, rounded corners, and a shadow.
Surface(
    modifier = modifier
        .fillMaxWidth()
        .padding(16.dp),
    shape = RoundedCornerShape(16.dp),
    shadowElevation = 8.dp,
    color = Color.White
) {
    // Profile content goes here
}


What these parameters do:
- fillMaxWidth() makes the surface occupy the available width.
- padding(16.dp) adds space around the surface.
- RoundedCornerShape(16.dp) rounds its corners.
- shadowElevation = 8.dp creates a shadow that makes the surface look raised.
- Color.White sets the background color.
The surface is currently empty, so it will show only the white panel and its shadow.
4. Arrange the content with a Column
Inside Surface, add a Column:
Column(
    horizontalAlignment = Alignment.CenterHorizontally,
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
) {
    // Add profile components here
}


A Column arranges its children vertically, from top to bottom. Alignment.CenterHorizontally centers the children horizontally, while the padding creates space between the surface's edges and its content.
5. Add the profile image and Edit button
We want the profile picture to be circular, with an Edit button overlapping its bottom-right corner. A Box allows composables to be layered on top of one another.
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
            // TODO: Add profile picture editing
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


Before running this code, add an image named cat to the project's app/src/main/res/drawable folder. You can use your own image or the resource linked in the original exercise: Profile image resource.
Important concepts:
- Image displays the picture loaded by painterResource.
- contentDescription provides an accessibility description for users of assistive technology.
- CircleShape clips the picture into a circle.
- ContentScale.Crop fills the image area while cropping any excess. This is useful for profile pictures.
- Box lets the Edit button overlap the image.
- Alignment.BottomEnd positions the Edit button at the bottom-right of the box.
- Card(onClick = ...) makes the Edit card clickable, but the actual editing behavior still needs to be implemented.
Experiment: modifier order
Compare these two modifier chains:
Modifier
    .padding(16.dp)
    .size(100.dp)


Modifier
    .size(100.dp)
    .padding(16.dp)


Modifier order matters because each modifier wraps or constrains the next one. In the first example, padding is applied outside the size modifier; in the second, the size is set first and padding is applied inside it. This changes the space available for the image, depending on the parent's constraints.
6. Add the name, description, and action buttons
Below the Box, add two Text composables and a Row containing the Follow and Message buttons.
Spacer(modifier = Modifier.height(16.dp))

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

Row(
    horizontalArrangement = Arrangement.SpaceEvenly,
    modifier = Modifier.fillMaxWidth()
) {
    Button(
        onClick = {
            // TODO: Add follow action
        }
    ) {
        Text(text = "Follow")
    }

    Button(
        onClick = {
            // TODO: Add message action
        }
    ) {
        Text(text = "Message")
    }
}


Spacer creates a fixed amount of empty space. The Row places the buttons horizontally, and Arrangement.SpaceEvenly distributes the available horizontal space evenly around its children.
The button callbacks are placeholders: the buttons respond to clicks, but they do not yet follow the user or send a message.
7. Put everything together
Here is the complete MainActivity.kt file. This version includes the profile composable, preview, and app entry point.
Before running: add cat.png or another supported image file named cat to app/src/main/res/drawable. Change the package name to match your project.

package com.example.profile

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.BorderStroke
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.Card
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.SolidColor
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.profile.ui.theme.ProfileTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            ProfileTheme {
                Scaffold(
                    modifier = Modifier.fillMaxSize()
                ) { innerPadding ->
                    Profile(
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }
}

@Composable
fun Profile(modifier: Modifier = Modifier) {
    Surface(
        modifier = modifier
            .fillMaxWidth()
            .padding(16.dp),
        shape = RoundedCornerShape(16.dp),
        shadowElevation = 8.dp,
        color = Color.White
    ) {
        Column(
            horizontalAlignment = Alignment.CenterHorizontally,
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        ) {
            Box(
                modifier = Modifier.size(116.dp)
            ) {
                Image(
                    painter = painterResource(
                        id = R.drawable.cat
                    ),
                    contentDescription = "Profile picture",
                    contentScale = ContentScale.Crop,
                    modifier = Modifier
                        .padding(8.dp)
                        .size(100.dp)
                        .clip(CircleShape)
                )

                Card(
                    onClick = {
                        // TODO: Add image editing
                    },
                    modifier = Modifier.align(
                        Alignment.BottomEnd
                    ),
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

            Spacer(modifier = Modifier.height(16.dp))

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

            Row(
                horizontalArrangement = Arrangement.SpaceEvenly,
                modifier = Modifier.fillMaxWidth()
            ) {
                Button(
                    onClick = {
                        // TODO: Add follow action
                    }
                ) {
                    Text(text = "Follow")
                }

                Button(
                    onClick = {
                        // TODO: Add message action
                    }
                ) {
                    Text(text = "Message")
                }
            }
        }
    }
}

@Preview(showBackground = true)
@Composable
fun ProfilePreview() {
    ProfileTheme {
        Profile(
            modifier = Modifier.padding(20.dp)
        )
    }
}



If your generated theme has a different name or package, update the ProfileTheme import to match your project. Android Studio can also add missing imports automatically.
8. Understand the component hierarchy
The UI is structured like this:
Scaffold
Provides the screen content area
Profile — Surface
White background, rounded corners and shadow
Column
Box
Profile image + Edit card

Text — Name

Text — Description

Row — Follow and Message buttons





This hierarchy illustrates a key Compose principle: small composables can be combined to build larger interfaces. Each layout has a specific role—Column arranges content vertically, Row arranges it horizontally, and Box layers content.
9. Student exercises
Try these extensions after the profile card works.
Practice checklist
0 of 5

[ ] Customize the profile
Change the name, job description, and profile image.


[ ] Change the image shape
Try RoundedCornerShape(16.dp) instead of CircleShape.


[ ] Add a follower count
Display a new Text composable showing the number of followers.


[ ] Add interaction
Use state to change the Follow button label to Following when clicked.


[ ] Improve the design
Add a divider, adjust spacing, or use MaterialTheme typography styles.



Challenge: make Follow interactive
The original example uses a placeholder callback. To make it interactive, introduce state:
var isFollowing by remember {
    mutableStateOf(false)
}

Button(
    onClick = {
        isFollowing = !isFollowing
    }
) {
    Text(
        text = if (isFollowing) {
            "Following"
        } else {
            "Follow"
        }
    )
}


Remember to import remember, mutableStateOf, getValue, and setValue. The state variable belongs inside a composable function, such as Profile. Because it is observable Compose state, clicking the button changes the label and triggers the necessary recomposition.
Important distinction: this changes the button's local UI state only. A real social app would also need to store the follow relationship and synchronize it with the backend.
Key takeaways
- Surface creates a visual container; Card is useful for grouped content and clickable cards.
- Column, Row, and Box are core layout building blocks.
- Modifier controls layout, padding, size, shape, and alignment; modifier order matters.
- Image displays visual resources, while contentDescription supports accessibility.
- @Preview helps you iterate on UI without launching the app.
- Observable state allows interactive UI elements to update when their state changes.
