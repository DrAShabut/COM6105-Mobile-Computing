Lab Tasks: Essential Android Libraries with Ktor, Kotlin Serialization, and Coil
Learning Outcomes
By the end of this lab, you will be able to:

Configure an Android project for internet-based applications.
Use Ktor to retrieve data from a REST API.
Use Kotlin Serialization to convert JSON into Kotlin objects.
Use Coil to load and display remote images.
Build a simple Jetpack Compose application that consumes and displays web data.
Part 1: Environment Setup
Task 1.1: Enable Internet Access
Instructions
Open AndroidManifest.xml.
Add the Internet permission above the <application> tag.
<uses-permission android:name="android.permission.INTERNET" />
Checkpoint
✅ Project builds successfully.
✅ No manifest errors are shown.
Task 1.2: Configure Project Dependencies
Step 1: Configure the Project-Level Gradle File
Open build.gradle.kts and add the Kotlin Serialization plugin:

plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    kotlin("plugin.serialization") version "2.0.0" apply false
}
Step 2: Configure the App-Level Gradle File
Add the following dependencies to app/build.gradle.kts:

dependencies {

    implementation("io.ktor:ktor-client-android:2.3.11")
    implementation("io.ktor:ktor-client-content-negotiation:2.3.11")
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.11")

    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.3")

    implementation("io.coil-kt:coil-compose:2.6.0")

    implementation(platform("androidx.compose:compose-bom:2024.05.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
}
Checkpoint
✅ Gradle sync completes successfully.
✅ No dependency errors are reported.
Part 2: Reading Data from an API Using Ktor
Task 2.1: Create a Network Repository
Instructions
Create a class called NetworkRepository.

class NetworkRepository {
    private val client = HttpClient(Android)
}
Task 2.2: Fetch Raw JSON Data
Instructions
Add a suspend function called:

suspend fun fetchRawApiPayload(endpointUrl: String): String
The function should:

Use client.get()
Retrieve the API response
Convert the response into text using bodyAsText()
Display the result in Logcat
Task 2.3: Test the API Call
API Endpoint
Use the following API:

https://jsonplaceholder.typicode.com/posts/1
Instructions
Call the function from a coroutine.
Run the application.
Open Logcat.
Search for:
KtorNetworkTest
Expected Result
You should see:

HTTP Status Code 200
JSON text displayed in Logcat
Reflection Question
Why should network operations never run on the main UI thread?

Part 3: Parsing JSON with Kotlin Serialization
Task 3.1: Create Data Models
Given the JSON response below:

{
  "status": "success",
  "data": {
    "id": 101,
    "image_url": "https://example.com/image.png"
  }
}
Create:

@Serializable
data class ImageResponse(...)
and

@Serializable
data class ImageData(...)
Checkpoint
✅ Data classes compile successfully.
✅ Both classes use the @Serializable annotation.
Task 3.2: Configure Ktor Content Negotiation
Instructions
Configure the HTTP client with JSON deserialization:

install(ContentNegotiation) {
    json(
        Json {
            ignoreUnknownKeys = true
        }
    )
}
Why?
This allows Ktor to automatically convert JSON responses into Kotlin objects.

Task 3.3: Automatically Deserialize JSON
Instructions
Replace manual JSON handling with automatic deserialization:

val response: ImageResponse =
    client.get(endpointUrl).body()
Extract the image URL and print it to Logcat.

Expected Output
SerializationTest:
https://example.com/image.png
Reflection Question
What advantages do strongly typed Kotlin data classes provide compared with manual JSON parsing?

Part 4: Displaying Remote Images with Coil
Task 4.1: Create an Image Composable
Instructions
Create a composable named:

ComposableRemoteImageDisplay()
that accepts:

imageUrl: String
Task 4.2: Implement AsyncImage
Requirements
Configure the image to:

Fill the width of the screen.
Have a fixed height.
Use rounded corners.
Crop the image appropriately.
Use:

AsyncImage()
Checkpoint
✅ Image space is visible on the screen.
✅ No compilation errors occur.
Task 4.3: Add Placeholder and Error Images
Instructions
Configure:

placeholder =
error =
using Android drawable resources.

Test Case
Use an invalid URL:

https://invalid-url.com/image.jpg
Expected Result
The error image should appear automatically when image loading fails.

Reflection Question
Why are placeholder images important for user experience?

Part 5: Integration Challenge
Task 5.1: Connect All Three Libraries
Create the following application workflow:

Ktor
 ↓
JSON Response
 ↓
Kotlin Serialization
 ↓
Image URL
 ↓
Coil AsyncImage
Instructions
Retrieve JSON data from an API.
Deserialize the response into Kotlin objects.
Extract an image URL.
Display the image in the user interface.
Checkpoint
✅ API data is successfully retrieved.
✅ JSON is successfully deserialized.
✅ Image is displayed correctly.
✅ Application runs without crashes.
Part 6: Mini Project – Weather App
Task 6.1: Create Weather Models
Instructions
Create the following serializable classes:

WeatherApiResponse
ConditionData
Use the @Serializable annotation.

Checkpoint
✅ Weather models are created successfully.
Task 6.2: Retrieve Weather Data
Instructions
Use Ktor to retrieve:

Location name
Temperature
Weather condition
Weather icon URL
from an online weather API.

Checkpoint
✅ Weather data is downloaded successfully.
Task 6.3: Build the User Interface
Instructions
Display the following information:

Location name
Temperature
Weather description
Weather icon
Use:

Card
Text
AsyncImage
within your Compose interface.

Checkpoint
✅ All weather information is displayed correctly.
Task 6.4: Implement a Loading State
Instructions
Display a loading spinner while data is being retrieved:

CircularProgressIndicator()
Checkpoint
✅ Loading indicator appears while fetching data.
✅ Spinner disappears when data has loaded.
✅ Weather information becomes visible.
Extension Activities (Optional)
Activity A: Error Handling
Add appropriate error handling using:

try {
    ...
} catch (e: Exception) {
    ...
}
Handle:

No internet connection
Invalid API endpoints
Server errors
Activity B: Refresh Weather Data
Add a button that allows users to refresh the weather information.

Challenge
Update the displayed weather without restarting the application.

Activity C: Search by City
Allow users to:

Enter a city name.
Submit the city.
Retrieve weather information dynamically.
Example Cities
Leeds
London
Manchester
Birmingham
Lab Deliverables
Before leaving the lab, ensure you have completed the following:

Added the Internet permission to AndroidManifest.xml.
Configured all required Ktor dependencies.
Configured Kotlin Serialization.
Configured Coil.
Retrieved data from a REST API.
Parsed JSON into Kotlin objects.
Displayed a remote image using AsyncImage.
Developed a basic Weather App.
Tested the application successfully.
Answered all reflection questions.
Post-Lab Reflection
Answer the following questions:

What role does Ktor play in Android applications?
How does Kotlin Serialization improve the reliability of API integration?
What advantages does Coil provide compared with manually downloading and displaying images?
Describe the journey of data from the API request to the Compose UI.
Which part of today's lab did you find most challenging and why?
Success Criteria
You have successfully completed this lab if you can:

Retrieve data from an online API using Ktor.
Convert JSON responses into Kotlin objects using Kotlin Serialization.
Display remote images using Coil.
Integrate networking, data processing, and UI rendering into a working Jetpack Compose application.
Build a functional Weather App using modern Android development practices.
