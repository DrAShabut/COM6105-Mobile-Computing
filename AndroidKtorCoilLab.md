# Lab Tasks: Essential Android Libraries with Ktor, Kotlin Serialization, and Coil

## Learning Outcomes

By the end of this lab, you will be able to:

- Configure an Android project for internet-based applications.
- Use **Ktor** to retrieve data from a REST API.
- Use **Kotlin Serialization** to convert JSON into Kotlin objects.
- Use **Coil** to load and display remote images.
- Build a simple Jetpack Compose application that consumes and displays web data.

---

# Part 1: Environment Setup

## Task 1.1: Enable Internet Access

Add the following permission to `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Checkpoint

- ✅ Project builds successfully.
- ✅ No manifest errors are shown.

---

## Task 1.2: Configure Project Dependencies

Add Kotlin Serialization to the project-level Gradle file and configure Ktor, Kotlin Serialization, Coil, and Compose dependencies in the app-level Gradle file.

### Checkpoint

- ✅ Gradle sync completes successfully.
- ✅ No dependency errors are reported.

---

# Part 2: Reading Data from an API Using Ktor

## Task 2.1: Create a Network Repository

Create a class called `NetworkRepository` and initialise a Ktor `HttpClient`.

## Task 2.2: Fetch Raw JSON Data

Create a suspend function that:

- Uses `client.get()`
- Retrieves API data
- Converts the response using `bodyAsText()`
- Logs the result in Logcat

## Task 2.3: Test the API Call

Use:

```text
https://jsonplaceholder.typicode.com/posts/1
```

### Reflection Question

Why should network operations never run on the main UI thread?

---

# Part 3: Parsing JSON with Kotlin Serialization

## Task 3.1: Create Data Models

Create serializable Kotlin data classes matching the supplied JSON structure.

### Checkpoint

- ✅ Data classes compile successfully.
- ✅ All classes use `@Serializable`.

## Task 3.2: Configure Content Negotiation

Configure Ktor with JSON deserialization and `ignoreUnknownKeys = true`.

## Task 3.3: Automatically Deserialize JSON

Use:

```kotlin
val response: ImageResponse = client.get(endpointUrl).body()
```

Extract and display the image URL.

### Reflection Question

What advantages do strongly typed Kotlin data classes provide compared with manual JSON parsing?

---

# Part 4: Displaying Remote Images with Coil

## Task 4.1: Create an Image Composable

Create a composable named `ComposableRemoteImageDisplay()` that accepts an image URL.

## Task 4.2: Implement AsyncImage

Configure the image to:

- Fill available width
- Use a fixed height
- Display rounded corners
- Crop content appropriately

### Checkpoint

- ✅ Image appears on screen.
- ✅ No compilation errors occur.

## Task 4.3: Add Placeholder and Error Images

Configure placeholder and error images.

### Reflection Question

Why are placeholder images important for user experience?

---

# Part 5: Integration Challenge

## Task 5.1: Connect All Three Libraries

Build the following workflow:

```text
Ktor
 ↓
JSON Response
 ↓
Kotlin Serialization
 ↓
Image URL
 ↓
Coil AsyncImage
```

### Checkpoint

- ✅ API data retrieved successfully.
- ✅ JSON deserialized successfully.
- ✅ Image displayed correctly.
- ✅ No crashes occur.

---

# Part 6: Mini Project – Weather App

## Task 6.1: Create Weather Models

Create:

- `WeatherApiResponse`
- `ConditionData`

using `@Serializable`.

## Task 6.2: Retrieve Weather Data

Use Ktor to retrieve:

- Location name
- Temperature
- Weather condition
- Weather icon URL

## Task 6.3: Build the User Interface

Display:

- Location name
- Temperature
- Weather description
- Weather icon

using Compose.

## Task 6.4: Implement a Loading State

Display:

```kotlin
CircularProgressIndicator()
```

while data is loading.

### Checkpoint

- ✅ Loading indicator works.
- ✅ Weather information displays correctly.
- ✅ Weather icon loads successfully.

---

# Extension Activities

### Activity A: Error Handling

Handle:

- No internet connection
- Invalid API endpoints
- Server errors

### Activity B: Refresh Weather Data

Add a button to refresh data.

### Activity C: Search by City

Allow users to enter a city name and retrieve weather information dynamically.

---

# Lab Deliverables

- [ ] Added Internet permission.
- [ ] Configured Ktor.
- [ ] Configured Kotlin Serialization.
- [ ] Configured Coil.
- [ ] Retrieved API data.
- [ ] Parsed JSON.
- [ ] Displayed remote images.
- [ ] Built the Weather App.
- [ ] Tested the app successfully.

---

# Post-Lab Reflection

1. What role does Ktor play in Android applications?
2. How does Kotlin Serialization improve the reliability of API integration?
3. What advantages does Coil provide compared with manual image loading?
4. Describe the journey of data from the API request to the Compose UI.
5. Which part of today's lab did you find most challenging and why?
