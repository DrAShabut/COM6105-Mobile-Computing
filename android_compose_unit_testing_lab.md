# Android Development and Testing Lab
## Build a Kotlin Android app with Jetpack Compose, then test it step by step

**Outcome:** A working Login demo app with local unit tests and Jetpack Compose UI tests  

---

## Learning outcomes

By the end of this practical, you should be able to:

1. Create a new Android project using Jetpack Compose.
2. Recognise the purpose of `main`, `test`, and `androidTest` source sets.
3. Organise a small app using a simple MVVM/Clean Architecture-inspired structure.
4. Write a repository interface, implementation, use case, ViewModel, and Compose screen.
5. Write local unit tests with JUnit and MockK.
6. Test coroutine-based ViewModel code.
7. Write Compose UI tests for visible elements and user interactions.
8. Run tests in Android Studio and from the Gradle command line.

## What you will build

You will create a small Login demo. It accepts one demonstration credential pair:

- Email: `student@example.com`
- Password: `password123`

A matching pair displays a success message. Any other non-empty pair displays an error message.

> **Important:** This is a teaching example, not real authentication. It checks credentials locally in the app. Never store real passwords in an Android app or use this example as a production authentication system.

---

# Part 1 — Prepare your development environment

## Step 1. Create a new Compose project

1. From the Android Studio welcome screen, select **New Project**.
2. Select the **Phone and Tablet** category.
3. Choose **Empty Activity**. Make sure it is the Jetpack Compose template, not **Empty Views Activity**.
4. Select **Next**.
5. Enter the following details:

   | Setting | Value |
   |---|---|
   | Name | `LoginTestingApp` |
   | Package name | `com.example.logintestingapp` |
   | Language | Kotlin |
   | Minimum SDK | API 23 or a higher level supported by your course |

6. Keep the default build configuration unless your course tutor specifies otherwise.
7. Select **Finish**.
8. Wait for Gradle sync to complete. Do not continue while Android Studio is still syncing.

**Checkpoint:** The project opens without a Gradle sync error.

## Step 3. Run the starter app

1. Select your emulator from the device selector in the Android Studio toolbar.
2. Select **Run ▶**.
3. Wait for the app to install and open.
4. Confirm that the starter screen appears.

If the project does not build, read the first error in the **Build** window. Fix that error before moving on.

---

# Part 2 — Understand the project and create packages

## Step 4. Understand the source sets

In the Android project, expand `app/src`. You should see:

```text
app/
└── kotlin+java
    ├── main/          # Application source code
    ├── test/          # Local JVM unit tests
    └── androidTest/   # Tests that run on an Android device/emulator
```

Use these locations as follows:

- `main`: the app that you are building.
- `test`: fast local tests for Kotlin classes that do not need an Android device.
- `androidTest`: tests that need Android, including the Compose UI tests in this workshop.

## Step 5. Create the app packages

In Android Studio's Project panel, select the Android view or Project view and locate:

`app/src/main/java/com/example/logintestingapp`

Right-click the package, choose **New → Package**, and create:

- `data`
- `domain`
- `presentation`
- `ui`

Your intended structure is:

```text
com.example.logintestingapp/
├── data/
│   └── AuthRepositoryImpl.kt
├── domain/
│   ├── AuthRepository.kt
│   └── LoginUseCase.kt
├── presentation/
│   └── LoginViewModel.kt
├── ui/
│   └── LoginScreen.kt
└── MainActivity.kt
```

You will create the test files later under the matching package in `src/test` or `src/androidTest`.

---

# Part 3 — Add the required dependencies

## Step 6. Check the app Gradle file

Open:

`app/build.gradle.kts`

Do not delete the dependencies that Android Studio generated. The Compose template normally already includes the Activity, Compose UI, Material 3, and lifecycle Compose dependencies.

Inside the existing `dependencies { ... }` block, add:

```kotlin
// Local unit tests
testImplementation("junit:junit:4.13.2")
testImplementation("io.mockk:mockk:1.13.16")
testImplementation(
    "org.jetbrains.kotlinx:kotlinx-coroutines-test:1.10.2"
)

// Compose instrumented UI tests
androidTestImplementation(
    platform("androidx.compose:compose-bom:2025.02.00")
)
androidTestImplementation("androidx.compose.ui:ui-test-junit4")

// Test activity/manifest support for Compose UI tests
debugImplementation("androidx.compose.ui:ui-test-manifest")

implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")

```

Select **Sync Now** and wait for Gradle sync to finish.

**Checkpoint:** Gradle sync succeeds and no dependency is unresolved.

---

# Part 4 — Build the application

Create each file in the package shown. Check that the package declaration at the top of each file matches its location.

## Step 7. Create the repository interface

Create:

`domain/AuthRepository.kt`

Add:

```kotlin
package com.example.logintestingapp.domain

interface AuthRepository {
    suspend fun login(
        email: String,
        password: String
    ): Boolean
}
```

**Task 1:** Read the interface and explain why it describes *what* the repository does without specifying *how* it does it.

## Step 8. Create the repository implementation

Create:

`data/AuthRepositoryImpl.kt`

Add:

```kotlin
package com.example.logintestingapp.data

import com.example.logintestingapp.domain.AuthRepository

class AuthRepositoryImpl : AuthRepository {

    override suspend fun login(
        email: String,
        password: String
    ): Boolean {
        // Demonstration only: not real authentication.
        return email == "student@example.com" &&
            password == "password123"
    }
}
```

This class supplies the demonstration behaviour. In a real app, a repository might call a remote API or local database.

## Step 9. Create the use case

Create:

`domain/LoginUseCase.kt`

Add:

```kotlin
package com.example.logintestingapp.domain

class LoginUseCase(
    private val repository: AuthRepository
) {
    suspend operator fun invoke(
        email: String,
        password: String
    ): Boolean {
        if (email.isBlank() || password.isBlank()) {
            return false
        }

        return repository.login(email, password)
    }
}
```

The use case applies a simple business rule: both fields must contain text before the repository is called.

## Step 10. Create the ViewModel

Create:

`presentation/LoginViewModel.kt`

Add:

```kotlin
package com.example.logintestingapp.presentation

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.logintestingapp.domain.LoginUseCase
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch

class LoginViewModel(
    private val loginUseCase: LoginUseCase
) : ViewModel() {

    private val _loginState = MutableStateFlow<Boolean?>(null)
    val loginState = _loginState.asStateFlow()

    fun login(email: String, password: String) {
        viewModelScope.launch {
            _loginState.value = loginUseCase(email, password)
        }
    }
}
```

The state means:

- `null`: no login result has been produced yet.
- `true`: login succeeded.
- `false`: login failed.

This deliberately small ViewModel is sufficient for the first testing exercise. A production app should generally model loading, success, and error explicitly.

## Step 11. Create the Compose screen

Create:

`ui/LoginScreen.kt`

Add:

```kotlin
package com.example.logintestingapp.ui

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun LoginScreen(
    onLoginClick: (String, String) -> Unit,
    loginResult: Boolean? = null
) {
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(
            text = "Login",
            style = MaterialTheme.typography.headlineMedium
        )

        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            modifier = Modifier.fillMaxWidth()
        )

        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Password") },
            modifier = Modifier.fillMaxWidth()
        )

        Button(
            onClick = { onLoginClick(email, password) },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Login")
        }

        when (loginResult) {
            true -> Text("Login successful!")
            false -> Text("Invalid credentials")
            null -> Unit
        }
    }
}
```

Notice that the screen does not create or retrieve its own ViewModel. It receives a callback and a result as parameters. This makes it easier to test the UI independently.

## Step 12. Connect the screen in MainActivity

Open the generated `MainActivity.kt`. Replace its contents with:

```kotlin
package com.example.logintestingapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.runtime.collectAsState
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.logintestingapp.data.AuthRepositoryImpl
import com.example.logintestingapp.domain.LoginUseCase
import com.example.logintestingapp.presentation.LoginViewModel
import com.example.logintestingapp.ui.LoginScreen

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val loginUseCase = LoginUseCase(AuthRepositoryImpl())

        setContent {
            val loginViewModel: LoginViewModel = viewModel(
                factory = object : ViewModelProvider.Factory {
                    @Suppress("UNCHECKED_CAST")
                    override fun <T : ViewModel> create(
                        modelClass: Class<T>
                    ): T {
                        if (modelClass.isAssignableFrom(LoginViewModel::class.java)) {
                            return LoginViewModel(loginUseCase) as T
                        }
                        throw IllegalArgumentException(
                            "Unknown ViewModel class: ${modelClass.name}"
                        )
                    }
                }
            )

            val loginResult = loginViewModel.loginState.collectAsState()

            LoginScreen(
                onLoginClick = { email, password ->
                    loginViewModel.login(email, password)
                },
                loginResult = loginResult.value
            )
        }
    }
}
```

## Step 13. Run and manually check the app

1. Start your emulator.
2. Select the emulator in the Android Studio toolbar.
3. Select **Run ▶**.
4. Wait for the app to open.
5. Enter the demonstration email and password.
6. Select **Login**.
7. Confirm that the success message appears.
8. Try a different password and confirm that the error message appears.

| Email | Password | Expected message |
|---|---|---|
| `student@example.com` | `password123` | Login successful! |
| `student@example.com` | `incorrect` | Invalid credentials |
| Empty | `password123` | Invalid credentials |

**Checkpoint:** The app builds, launches, and responds to the demonstration credentials.

---

# Part 5 — Write local unit tests

Local unit tests go in:

`app/src/test`

They run on your computer's JVM and do not need an emulator.

## Step 14. Create the test package

1. In Android Studio, expand `app/src/test`.
2. Find the package `com.example.logintestingapp(test)`.
3. Right-click the package.
4. Select **New → Kotlin Class/File**.
5. Name the file `LoginUseCaseTest`.

## Step 15. Test the LoginUseCase

Add the following to `LoginUseCaseTest.kt`:

```kotlin
package com.example.logintestingapp

import com.example.logintestingapp.domain.AuthRepository
import com.example.logintestingapp.domain.LoginUseCase
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.test.runTest
import org.junit.Assert.assertEquals
import org.junit.Before
import org.junit.Test

class LoginUseCaseTest {

    private lateinit var repository: AuthRepository
    private lateinit var loginUseCase: LoginUseCase

    @Before
    fun setup() {
        repository = mockk()
        loginUseCase = LoginUseCase(repository)
    }

    @Test
    fun `valid credentials return true`() = runTest {
        coEvery {
            repository.login("student@example.com", "password123")
        } returns true

        val result = loginUseCase(
            "student@example.com",
            "password123"
        )

        assertEquals(true, result)
    }

    @Test
    fun `invalid credentials return false`() = runTest {
        coEvery {
            repository.login(any(), any())
        } returns false

        val result = loginUseCase(
            "student@example.com",
            "incorrect"
        )

        assertEquals(false, result)
    }

    @Test
    fun `empty email returns false`() = runTest {
        val result = loginUseCase("", "password123")

        assertEquals(false, result)
    }

    @Test
    fun `empty password returns false`() = runTest {
        val result = loginUseCase("student@example.com", "")

        assertEquals(false, result)
    }
}
```

### Understand the test

- `@Before` runs before each test and creates fresh test objects.
- `@Test` marks a function as a test.
- `mockk()` creates a mock repository.
- `coEvery { ... }` defines the result of a suspend function.
- `runTest` runs coroutine test code.
- `assertEquals` compares the actual result with the expected result.

The empty-input tests also check that the use case rejects blank input without needing a repository response.

## Step 16. Run the use case tests

1. Open `LoginUseCaseTest.kt`.
2. Right-click inside the class.
3. Select **Run 'LoginUseCaseTest'**.
4. Check the Run window.
5. Confirm that all four tests pass.

If a test fails, read the failure message and compare the expected and actual values.

---

# Part 6 — Test the ViewModel and coroutines

The ViewModel uses `viewModelScope`, which runs work on the Main dispatcher. A local JVM test does not have Android's normal Main dispatcher, so we replace it with a test dispatcher.

## Step 17. Create MainDispatcherRule

In `src/test/java/com/example/logintestingapp`, create:

`MainDispatcherRule.kt`

Add:

```kotlin
package com.example.logintestingapp

import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.StandardTestDispatcher
import kotlinx.coroutines.test.TestDispatcher
import kotlinx.coroutines.test.resetMain
import kotlinx.coroutines.test.setMain
import org.junit.rules.TestWatcher
import org.junit.runner.Description

@OptIn(ExperimentalCoroutinesApi::class)
class MainDispatcherRule(
    val dispatcher: TestDispatcher = StandardTestDispatcher()
) : TestWatcher() {

    override fun starting(description: Description) {
        Dispatchers.setMain(dispatcher)
    }

    override fun finished(description: Description) {
        Dispatchers.resetMain()
    }
}
```

This rule sets the Main dispatcher before each test and resets it afterwards.

## Step 18. Create LoginViewModelTest

Create:

`LoginViewModelTest.kt`

Add:

```kotlin
package com.example.logintestingapp

import com.example.logintestingapp.domain.LoginUseCase
import com.example.logintestingapp.presentation.LoginViewModel
import io.mockk.coEvery
import io.mockk.mockk
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.advanceUntilIdle
import kotlinx.coroutines.test.runTest
import org.junit.Assert.assertEquals
import org.junit.Before
import org.junit.Rule
import org.junit.Test

@OptIn(ExperimentalCoroutinesApi::class)
class LoginViewModelTest {

    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()

    private lateinit var loginUseCase: LoginUseCase
    private lateinit var viewModel: LoginViewModel

    @Before
    fun setup() {
        loginUseCase = mockk()
        viewModel = LoginViewModel(loginUseCase)
    }

    @Test
    fun `initial login state is null`() {
        assertEquals(null, viewModel.loginState.value)
    }

    @Test
    fun `successful login updates state`() = runTest {
        coEvery {
            loginUseCase("student@example.com", "password123")
        } returns true

        viewModel.login("student@example.com", "password123")

        advanceUntilIdle()

        assertEquals(true, viewModel.loginState.value)
    }

    @Test
    fun `failed login updates state`() = runTest {
        coEvery {
            loginUseCase(any(), any())
        } returns false

        viewModel.login("student@example.com", "incorrect")

        advanceUntilIdle()

        assertEquals(false, viewModel.loginState.value)
    }
}
```

### Understand the ViewModel test

- The use case is mocked, so the test focuses on the ViewModel rather than the business logic.
- `viewModel.login()` launches a coroutine.
- `advanceUntilIdle()` advances the test scheduler until scheduled work is complete.
- The assertion checks the state exposed by the ViewModel.

## Step 19. Run the ViewModel tests

1. Open `LoginViewModelTest.kt`.
2. Right-click the class.
3. Select **Run 'LoginViewModelTest'**.
4. Confirm that all three tests pass.

**Checkpoint:** You should now have seven passing local unit tests: four use case tests and three ViewModel tests.

---

# Part 7 — Write Jetpack Compose UI tests

Compose UI tests run on an Android emulator or physical device. They belong in:

`app/src/androidTest/java/`

## Step 20. Create the instrumented test package

1. Expand `app/src/androidTest`.
2. Find the package `com.example.logintestingapp(androidTest)`.
3. Right-click the package.
4. Select **New → Kotlin Class/File**.
5. Name the file `LoginScreenTest`.

## Step 21. Test the Compose screen

Add:

```kotlin
package com.example.logintestingapp

import androidx.compose.ui.test.assertIsDisplayed
import androidx.compose.ui.test.junit4.createComposeRule
import androidx.compose.ui.test.onNodeWithText
import androidx.compose.ui.test.performClick
import androidx.compose.ui.test.performTextInput
import com.example.logintestingapp.ui.LoginScreen
import org.junit.Assert.assertEquals
import org.junit.Rule
import org.junit.Test

class LoginScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun loginScreen_displaysAllElements() {
        composeTestRule.setContent {
            LoginScreen(
                onLoginClick = { _, _ -> }
            )
        }

        composeTestRule.onNodeWithText("Login")
            .assertIsDisplayed()

        composeTestRule.onNodeWithText("Email")
            .assertIsDisplayed()

        composeTestRule.onNodeWithText("Password")
            .assertIsDisplayed()
    }

    @Test
    fun loginButton_triggersCallback() {
        var actualEmail = ""
        var actualPassword = ""

        composeTestRule.setContent {
            LoginScreen(
                onLoginClick = { email, password ->
                    actualEmail = email
                    actualPassword = password
                }
            )
        }

        composeTestRule.onNodeWithText("Email")
            .performTextInput("student@example.com")

        composeTestRule.onNodeWithText("Password")
            .performTextInput("password123")

        composeTestRule.onNodeWithText("Login")
            .performClick()

        assertEquals("student@example.com", actualEmail)
        assertEquals("password123", actualPassword)
    }

    @Test
    fun successMessage_isDisplayed() {
        composeTestRule.setContent {
            LoginScreen(
                onLoginClick = { _, _ -> },
                loginResult = true
            )
        }

        composeTestRule.onNodeWithText("Login successful!")
            .assertIsDisplayed()
    }

    @Test
    fun errorMessage_isDisplayed() {
        composeTestRule.setContent {
            LoginScreen(
                onLoginClick = { _, _ -> },
                loginResult = false
            )
        }

        composeTestRule.onNodeWithText("Invalid credentials")
            .assertIsDisplayed()
    }
}
```

### Understand the Compose test

| Test API | Purpose |
|---|---|
| `createComposeRule()` | Creates a Compose UI test rule |
| `setContent {}` | Displays the composable under test |
| `onNodeWithText()` | Finds a UI element using its text |
| `assertIsDisplayed()` | Checks that an element is visible |
| `performTextInput()` | Enters text into a text field |
| `performClick()` | Simulates a click |

The button test verifies that the screen passes the entered values to its callback.

## Step 22. Run the Compose UI tests

1. Start your Android emulator.
2. Open `LoginScreenTest.kt`.
3. Right-click the class.
4. Select **Run 'LoginScreenTest'**.
5. Wait for the tests to execute on the emulator.
6. Confirm that all four tests pass.

**Checkpoint:** You should now have four passing Compose UI tests.

---

# Part 8 — Run all tests and review the results

## Step 23. Run local unit tests from the terminal

Open the Android Studio Terminal.

Run:

```bash
./gradlew testDebugUnitTest
```

This runs the local unit tests for the debug build.

On Windows, use:

```bat
gradlew.bat testDebugUnitTest
```

## Step 24. Run Compose instrumented tests

Make sure your emulator is running, then execute:

```bash
./gradlew connectedDebugAndroidTest
```

On Windows:

```bat
gradlew.bat connectedDebugAndroidTest
```

This runs the instrumented tests on a connected emulator or device.

## Step 25. Run both test tasks

You can run both with:

```bash
./gradlew testDebugUnitTest connectedDebugAndroidTest
```

The test summary should show:

| Test class | Number of tests | Expected |
|---|---:|---|
| `LoginUseCaseTest` | 4 | Pass |
| `LoginViewModelTest` | 3 | Pass |
| `LoginScreenTest` | 4 | Pass |
| **Total** | **11** | **Pass** |

If a test fails, do not simply remove it. Read the error, identify the cause, correct the code, and run the test again.

# Part 10 — Student exercises

Complete these additional tasks to demonstrate your understanding.

## Exercise 1 — Add another business rule

Change `LoginUseCase` so that an email without an `@` symbol is rejected.

**Your tasks:**

1. Update the use case.
2. Add a unit test for an invalid email.
3. Run the use case tests.
4. Confirm that the new test passes.

## Exercise 2 — Add a test for whitespace

Add a test that checks what happens when the email contains only spaces.

**Question:** Should the repository be called when the email is blank?

*Hint:* MockK provides `coVerify` to check whether a suspend function was called.

## Exercise 3 — Add another Compose UI test

Write a Compose UI test that checks that the error message is not displayed when `loginResult` is `null`.

*Hint:* Look at `assertDoesNotExist()`.

## Exercise 4 — Test the repository

Write local unit tests for `AuthRepositoryImpl`.

Test:

1. The demonstration email and password.
2. The correct email with an incorrect password.
3. An incorrect email with the correct password.

## Exercise 5 — Improve the application

Research how you could replace the nullable Boolean login state with a sealed UI state:

```kotlin
sealed interface LoginUiState {
    data object Idle : LoginUiState
    data object Loading : LoginUiState
    data object Success : LoginUiState
    data class Error(val message: String) : LoginUiState
}
```

**Challenge:** Update the ViewModel, Compose screen, and tests to support all four states.

---

## Recommended official resources

- [Android testing fundamentals](https://developer.android.com/training/testing)
- [Testing Jetpack Compose](https://developer.android.com/develop/ui/compose/testing)
- [Testing Kotlin coroutines](https://developer.android.com/kotlin/coroutines/test)
- [MockK documentation](https://mockk.io/)
- [JUnit 4 documentation](https://junit.org/junit4/)

---