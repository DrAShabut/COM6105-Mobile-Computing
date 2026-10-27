# Android Practical Workbook: Ktor, Kotlin Serialization, Coil, and Jetpack Compose

## Lab Overview

**Platform:** Android Studio, Kotlin, Jetpack Compose  
**Main libraries:** Ktor Client, Kotlin Serialization, Coil  

## Learning Outcomes

By the end of the practical, you should be able to:

- Create and configure a new Android Studio project using Kotlin and Jetpack Compose.
- Explain the roles of Ktor, Kotlin Serialization, Coil, coroutines, and Compose.
- Add internet permission and configure Gradle dependencies.
- Make HTTP requests without blocking the main UI thread.
- Convert JSON responses into strongly typed Kotlin data classes.
- Display remote images with loading and error placeholders.
- Separate UI code from networking and data-model code.
- Represent loading, success, and error states in the UI.
- Add search, refresh, retry, and basic input validation.
- Test successful requests and common failure scenarios.
- Extend the starter app into multiple small projects.

## What Each Library Does

| Technology | Purpose in this lab |
|---|---|
| Ktor Client | Sends HTTP requests and receives API responses. |
| Kotlin Serialization | Converts JSON into Kotlin objects. |
| Coil | Loads remote images asynchronously in Compose. |
| Jetpack Compose | Builds the app's user interface declaratively. |
| Kotlin coroutines | Support suspending network calls without blocking the UI. |
| ViewModel and StateFlow (extension) | Keep UI state and business logic separate from composables. |

---

# Part 1 — Create a New Android Project

## Task 1.1: Create the project

1. Open Android Studio.
2. Select **New Project**.
3. Choose **Empty Activity** (the Compose template, if the wizard offers multiple templates).
4. Enter a project name, such as `KtorCoilLab`.
5. Choose a package name, for example `com.example.ktorcoillab`.
6. Select **Kotlin** as the language.
7. Choose the minimum SDK required by target devices.
8. Finish creating the project and wait for Gradle sync to complete.
9. Run the starter app on an emulator or physical device.

### Checkpoint

- [ ] The project opens without Gradle errors.
- [ ] The starter app builds and launches.
- [ ] You can identify the app module and its `MainActivity.kt` file.

## Task 1.2: Understand the project structure

A typical project will include files similar to these:

```text
KtorCoilLab/
├── app/
│   ├── build.gradle.kts
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           └── java/com/example/ktorcoillab/
│               ├── MainActivity.kt
│               ├── data/
│               ├── model/
│               └── ui/
├── build.gradle.kts
├── gradle/
│   └── libs.versions.toml   (if using a version catalog)
└── settings.gradle.kts
```

The exact folders may differ depending on the Android Studio template. Create the `data`, `model`, and `ui` packages under your app's Kotlin package if they do not exist.

- `MainActivity.kt`: app entry point.
- `model/`: data classes representing API responses.
- `data/`: network client and repository code.
- `ui/`: composables and screen-level UI code.
- `AndroidManifest.xml`: app declarations and permissions.
- `app/build.gradle.kts`: app module plugins, SDK settings, and dependencies.

## Task 1.3: Enable internet access

Open `app/src/main/AndroidManifest.xml` and add the permission as a direct child of `<manifest>`, outside the `<application>` element:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

The permission is normally declared before the `<application>` element. You do not need to request this permission at runtime for ordinary internet access.

### Checkpoint

- [ ] The permission is in the correct location.
- [ ] The XML file has no errors.
- [ ] The app still builds.

## Task 1.4: Add the libraries

Open the app module's `build.gradle.kts`. Add the serialization compiler plugin in the appropriate `plugins` block, and add the Ktor, serialization JSON, and Coil dependencies in the `dependencies` block.

The following is an **illustrative dependency list**. Use mutually compatible versions and the current versions recommended by your course or official library documentation.

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.compose)
    // Add this one
    alias(libs.plugins.kotlin.serialization)
}

dependencies {
    // Ktor — keep all modules on the same version
    implementation("io.ktor:ktor-client-android:3.2.3")
    implementation("io.ktor:ktor-client-core:3.2.3")
    implementation("io.ktor:ktor-client-content-negotiation:3.2.3")
    implementation("io.ktor:ktor-serialization-kotlinx-json:3.2.3")

    // Kotlin Serialization — compatible with Kotlin 2.2
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.9.0")

    // Coil 3 — use a version before the Kotlin 2.4 upgrade
    implementation("io.coil-kt.coil3:coil-compose:3.4.0")
    implementation("io.coil-kt.coil3:coil-network-okhttp:3.4.0")
}
```
Open gradle/libs.versions.toml.

Under [plugins], add this entry if it isn't already there:

```kotlin
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

### Important setup notes

- The Kotlin Serialization Gradle plugin version should match the Kotlin version used by the project.
- Use the same Ktor version for the Ktor modules.
- Compose dependencies may already be configured by the project template.
- If Gradle reports a missing plugin alias, either define it in the version catalog or use the plugin declaration style already used by the project.
- Click **Sync Now** and resolve dependency or plugin errors before proceeding.

### Checkpoint

- [ ] Gradle sync completes.
- [ ] No unresolved dependency or plugin errors remain.
- [ ] The starter app builds.

---

# Part 2 — Make a Basic API Request with Ktor

For this section, use the sample endpoint:

`https://jsonplaceholder.typicode.com/posts/1`

It returns a JSON object containing fields such as `userId`, `id`, `title`, and `body`.

## Task 2.1: Create a network client

Create `data/NetworkClient.kt`:

Steps to create the data package
1. In Android Studio, open the Project panel and select the Android view.
2. Expand app → kotlin+java → com.example.ktorcoillab (the displayed label may vary).
3. Right-click com.example.ktorcoillab.
4. Select New → Package.
5. Enter com.example.ktorcoillab.data.
6. Right-click the new data package and select New → Kotlin Class/File.
7. Name the file NetworkClient.
Use the same process to create NetworkRepository.kt.

Add this code:


```kotlin
package com.example.ktorcoillab.data

import io.ktor.client.HttpClient
import io.ktor.client.engine.android.Android
import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
import io.ktor.serialization.kotlinx.json.json
import kotlinx.serialization.json.Json

object NetworkClient {
    val client = HttpClient(Android) {
        install(ContentNegotiation) {
            json(
                Json {
                    ignoreUnknownKeys = true
                }
            )
        }
    }
}
```

This creates one reusable client. The content negotiation configuration will also be used for typed JSON responses in Part 3.

## Task 2.2: Fetch raw response text

Create `data/NetworkRepository.kt`:

```kotlin
package com.example.ktorcoillab.data

import io.ktor.client.call.body
import io.ktor.client.request.get
import io.ktor.client.statement.bodyAsText

class NetworkRepository {
    private val client = NetworkClient.client

    suspend fun fetchPostAsText(): String {
        val response = client.get(
            "https://jsonplaceholder.typicode.com/posts/1"
        )
        return response.bodyAsText()
    }
}
```

## Task 2.3: Show the result in Compose

For a small learning exercise, you can temporarily call the repository from a composable using a coroutine scope. In a larger app, move this work into a ViewModel.

Replace the greeting compose with the following: 

```kotlin
@Composable
fun RawJsonScreen() {
    val repository = remember { NetworkRepository() }
    val scope = rememberCoroutineScope()
    var result by remember { mutableStateOf("Tap the button to load data.") }
    var isLoading by remember { mutableStateOf(false) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Button(
            onClick = {
                scope.launch {
                    isLoading = true
                    result = try {
                        repository.fetchPostAsText()
                    } catch (e: Exception) {
                        "Request failed: ${e.message ?: "Unknown error"}"
                    } finally {
                        isLoading = false
                    }
                }
            },
            enabled = !isLoading
        ) {
            Text("Fetch post")
        }

        if (isLoading) {
            CircularProgressIndicator()
        }

        Text(result)
    }
}
```

Call the function RawJsonScreen() from the setContent in the main activitiy.
Your code should look like this:

```kotlin
package com.example.ktorcoillab

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.example.ktorcoillab.ui.theme.KtorCoilLabTheme
import com.example.ktorcoillab.data.NetworkRepository
import kotlinx.coroutines.launch

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            setContent {
                MaterialTheme {
                    RawJsonScreen()
                }
            }
        }
    }

    @Composable
    fun RawJsonScreen() {
        val repository = remember { NetworkRepository() }
        val scope = rememberCoroutineScope()
        var result by remember { mutableStateOf("Tap the button to load data.") }
        var isLoading by remember { mutableStateOf(false) }

        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(16.dp)
        ) {
            Button(
                onClick = {
                    scope.launch {
                        isLoading = true
                        result = try {
                            repository.fetchPostAsText()
                        } catch (e: Exception) {
                            "Request failed: ${e.message ?: "Unknown error"}"
                        } finally {
                            isLoading = false
                        }
                    }
                },
                enabled = !isLoading
            ) {
                Text("Fetch post")
            }

            if (isLoading) {
                CircularProgressIndicator()
            }

            Text(result)
        }
    }
}
```

## Task 2.4: Test the API call

1. Run the app.
2. Tap **Fetch post**.
3. Confirm that JSON text appears.
4. Open **Logcat** if you also add a log statement for debugging.
5. Turn off the device's internet connection and repeat the request.
6. Record what the app displays when the request fails.

### Reflection question

Why should network operations not run directly on the main UI thread? Explain how a suspending function and coroutine help.

### Checkpoint

- [ ] The request returns JSON while online.
- [ ] The UI remains responsive during the request.
- [ ] A failure message appears when the request cannot complete.

---

# Part 3 — Parse JSON with Kotlin Serialization

## Task 3.1: Create a data model

Create `model/Post.kt`:

```kotlin
package com.example.ktorcoillab.model

import kotlinx.serialization.Serializable

@Serializable
data class Post(
    val userId: Int,
    val id: Int,
    val title: String,
    val body: String
)
```

The property names and types should match the JSON response. If the API uses different property names, use `@SerialName("api_field_name")` where appropriate.

## Task 3.2: Deserialize the response

Add a typed method to `NetworkRepository.kt`:

```kotlin
import com.example.ktorcoillab.model.Post
import io.ktor.client.call.body
import io.ktor.client.request.get

suspend fun fetchPost(): Post {
    return client.get(
        "https://jsonplaceholder.typicode.com/posts/1"
    ).body()
}
```

Use the `ContentNegotiation` configuration from Part 2. Ktor can then deserialize the JSON body into a `Post` object.

## Task 3.3: Display the typed data

Create a composable that displays the title and body:

```kotlin
@Composable
fun PostContent(post: Post) {
    Column(
        modifier = Modifier.padding(16.dp)
    ) {
        Text(
            text = post.title,
            style = MaterialTheme.typography.titleLarge
        )
        Spacer(modifier = Modifier.height(8.dp))
        Text(text = post.body)
    }
}
```

Connect `fetchPost()` to the screen using a coroutine and store the returned `Post` in Compose state, or implement it in a ViewModel as an extension task.

Call the PostContent() from the Raw

```kotlin
@Composable
    fun RawJsonScreen(modifier: Modifier = Modifier) {
        val repository = remember { NetworkRepository() }
        val scope = rememberCoroutineScope()

        var post by remember {
            mutableStateOf<Post?>(null)
        }

        var error by remember {
            mutableStateOf<String?>(null)
        }

        var isLoading by remember {
            mutableStateOf(false)
        }

        Column(
            modifier = modifier
                .fillMaxSize()
                .padding(16.dp)
        ) {
            Button(
                onClick = {
                    scope.launch {
                        isLoading = true
                        error = null

                        try {
                            post = repository.fetchPost()
                        } catch (e: Exception) {
                            error = e.message ?: "Unknown error"
                        } finally {
                            isLoading = false
                        }
                    }
                },
                enabled = !isLoading
            ) {
                Text("Fetch post")
            }

            Spacer(modifier = Modifier.height(16.dp))

            if (isLoading) {
                CircularProgressIndicator()
            }

            error?.let {
                Text(
                    text = "Request failed: $it",
                    color = MaterialTheme.colorScheme.error
                )
            }

            post?.let {
                PostContent(post = it)
            }
        }
    }
```
Your current PostContent() function displays only the title and body. If you want the post ID to appear too, add this line inside its Column, before the title:

```kotlin
Text(text = "Post ID: ${post.id}")
```

The post data is retrieved from the online test API, not hardcoded in your app.

### Reflection question

What advantages do strongly typed data classes provide compared with manually extracting values from raw JSON?

### Checkpoint

- [ ] The `Post` model compiles.
- [ ] The JSON response deserializes into a `Post`.
- [ ] The UI displays the title and body.
- [ ] You can explain the purpose of `@Serializable` and `ignoreUnknownKeys`.

---

# Part 4 — Display Remote Images with Coil

## Task 4.1: Choose a valid image URL

Use a publicly accessible image URL from an API or a test image. You can use this image: https://picsum.photos/id/237/400/300

## Task 4.2: Create an image composable

The following example uses the Coil 2 Compose API. 

```kotlin
@Composable
fun ComposableRemoteImageDisplay(
    imageUrl: String,
    contentDescription: String?
) {
    AsyncImage(
        model = imageUrl,
        contentDescription = contentDescription,
        modifier = Modifier
            .fillMaxWidth()
            .height(220.dp)
            .clip(RoundedCornerShape(16.dp)),
        contentScale = ContentScale.Crop
    )
}
```

Add the Coil 2 `AsyncImage` import and Compose layout, drawing, and shape imports.
Call the function from the setContent()
```kotlin
ComposableRemoteImageDisplay("https://picsum.photos/id/237/400/300","A dog")
```

## Task 4.3: Add loading and error feedback

Use local drawable resources for the placeholder and error images, or use the relevant Coil request API for your chosen version. For example, if you have `R.drawable.image_placeholder` and `R.drawable.image_error`, configure `placeholder` and `error` using the API supported by your Coil version.

Test the composable with:
- A valid image URL.
- An invalid URL.
- An empty string.
- A slow or unavailable network connection.

### Reflection question

How does a placeholder improve the perceived responsiveness of an app? Why should the UI also show an error state?

### Checkpoint

- [ ] The image loads and scales correctly.
- [ ] The image has rounded corners.
- [ ] The image uses crop scaling.
- [ ] Loading and failure cases have understandable visual feedback.

---

# Part 5 — Integrate Ktor, Serialization, and Coil

## Task 5.1: Follow the data flow

```text
API request
    ↓
Ktor HTTP client
    ↓
JSON response
    ↓
Kotlin Serialization
    ↓
Kotlin data object containing an image URL
    ↓
Compose screen
    ↓
Coil AsyncImage
```

## Task 5.2: Use an image API response model

Choose an image API endpoint that returns a JSON object with an image URL. The exact fields depend on the API you choose. Create a serializable data class matching that response, for example:

```kotlin
@Serializable
data class ImageResponse(
    val id: String,
    val author: String,
    val download_url: String
)
```

This is only an example model. Confirm the real endpoint's JSON field names before using it. If the API uses different names, update the model or use `@SerialName`.

## Task 5.3: Build the integrated screen

Create a screen that:

1. Requests the image metadata using Ktor.
2. Deserializes the response into the correct data class.
3. Displays the author or title.
4. Passes the returned image URL to `ComposableRemoteImageDisplay()`.
5. Shows a loading indicator while the request is in progress.
6. Shows an error message and a retry button if the request fails.

### Checkpoint

- [ ] The metadata request succeeds.
- [ ] The response is deserialized.
- [ ] The screen displays text from the API.
- [ ] Coil loads the URL returned by the API.
- [ ] Loading, success, and error states are handled.

---

# Part 6 — Build a Mini Project: Weather App

## Project goal

Build a screen where users can search for a city and view the current weather information returned by a weather API.

**Important:** Choose a weather provider and review its API documentation. Some providers require an API key, may impose rate limits, or use different response structures. Do not commit secret API keys to a public repository or hard-code a private key in a production app.

## Task 6.1: Plan the screen

The screen should include:

- A city search field.
- A Search button.
- The location or city name.
- Current temperature and unit.
- Weather condition description.
- Weather icon loaded using Coil.
- Loading indicator.
- Error message with a retry option.
- Optional last-updated time.

## Task 6.2: Create the data model

Inspect a sample response from your chosen provider and create `@Serializable` models for the fields you need. For example, the model may contain a location object, current temperature, condition text, and icon URL.

Do not copy a model blindly from another API. Model properties must match the provider's actual JSON response or use serialization annotations to map names.

## Task 6.3: Implement the repository

Create a `WeatherRepository` that:
1. Accepts a city name.
2. Builds the request using the provider's documented parameters.
3. Sends the request through Ktor.
4. Deserializes the response into a typed model.
5. Returns the model to the calling layer.
6. Reports failures in a way the UI can handle.

Avoid logging API keys or other secrets.

## Task 6.4: Implement screen states

Your UI should handle at least these states:

| State | Expected UI |
|---|---|
| Idle | Prompt the user to search for a city. |
| Loading | Show `CircularProgressIndicator`. |
| Success | Show city, temperature, condition, and icon. |
| Error | Show a useful message and retry option. |
| Empty input | Ask the user to enter a city name. |

For the initial lab, Compose state can be used to demonstrate the workflow. For a more maintainable implementation, use a ViewModel and `StateFlow`.

## Task 6.5: Test the weather app

- [ ] Search for a valid city.
- [ ] Search for a city with spaces in its name.
- [ ] Submit an empty search field.
- [ ] Test a city that the provider cannot find.
- [ ] Test with the internet disabled.
- [ ] Confirm that the weather icon loads.
- [ ] Confirm that loading feedback disappears after success or failure.
- [ ] Verify that a second search replaces the previous result.

