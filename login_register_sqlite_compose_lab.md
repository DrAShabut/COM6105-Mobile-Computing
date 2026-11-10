# Android Lab: Build a Login and Registration App with Jetpack Compose and SQLite

## 1. Lab overview

**Duration:** 2 hours  
**Platform:** Android  
**Language:** Kotlin  
**User interface:** Jetpack Compose  
**Local persistence:** Android SQLite through `SQLiteOpenHelper`  
**Level:** Beginner to intermediate

In this practical lab, you will build a small Android application that allows a user to register an account, sign in with that account, and sign out. Account details are stored in a local SQLite database on the device, so a registered account remains available after the app is closed and reopened.

The interface is built with Jetpack Compose. Database work is performed away from the main UI thread so that database operations do not freeze the screen.

> **Important:** This is a teaching exercise for learning Android UI and local data persistence. A real production login system should normally use a trusted authentication service and a secure backend. A local database on a device is not a substitute for server-side authentication.

---

## 2. Learning outcomes

By the end of this lab, you should be able to:

1. Describe the role of Jetpack Compose in building an Android user interface.
2. Explain what a local SQLite database is and why an app may use one.
3. Create and manage a SQLite database using `SQLiteOpenHelper`.
4. Create a database table with a unique username constraint.
5. Insert a new user record and check whether a username is already registered.
6. Retrieve a user record and verify a password.
7. Use a random salt and a password-based key derivation function rather than storing plain-text passwords.
8. Connect Compose UI events to Kotlin functions.
9. Run database operations on a background dispatcher using Kotlin coroutines.
10. Test registration, login, validation, and persistence.

---

## 3. Requirements

- Android Studio with a working Android SDK.
- A Kotlin Android project with Jetpack Compose enabled.
- An Android emulator or physical Android device.
- Basic familiarity with Kotlin functions, classes, and variables.

The example uses the Android platform's built-in SQLite APIs. You do not need to add a separate SQLite database library.

## 4. What you are going to build

The application has two main screens or modes:

- **Register:** A new user enters a username and password. The app validates the input and saves the account details in SQLite.
- **Login:** A registered user enters their username and password. The app checks the stored password hash and displays a welcome message if the credentials match.

After a successful login, the app displays a welcome screen with a **Log out** button.

### Application flow

```text
                 Open application
                        |
                        v
                Login / Register UI
                   /         \
                  /           \
             Register         Login
                |                |
        Validate input     Validate input
                |                |
        Create salt and    Find saved account
        password hash            |
                |          Verify password hash
                |                |
         Save in SQLite     Credentials match?
                |             /       \
         Registration OK   Yes         No
                |            |           |
                +--------> Welcome     Show error
                             |
                           Log out
                             |
                        Login screen
```

---

## 5. Database design

Create a table named `users` with these columns:

| Column | Type | Purpose |
|---|---|---|
| `id` | INTEGER | Unique row identifier; generated automatically |
| `username` | TEXT | User's chosen username; must be unique |
| `salt` | BLOB | Random bytes used when deriving the password hash |
| `password_hash` | BLOB | Derived password value; the original password is not stored |

A simplified SQL definition is:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    salt BLOB NOT NULL,
    password_hash BLOB NOT NULL
);
```

### Why not save the password directly?

Saving a plain-text password means anyone who can read the database can immediately see the password. Instead, the example creates a random salt and derives a password hash using `PBKDF2WithHmacSHA256`. During login, the app derives a new value from the entered password and the saved salt, then compares the result with the saved hash.

A salt is not a secret. It makes identical passwords produce different stored values when different random salts are used. The example uses a constant iteration count for clarity; production systems should use current security guidance and a carefully reviewed authentication design.

---

## 6. Practical task A: Create the database helper

1. In Android Studio, open the **app** module's Kotlin source folder.
2. Create a package named `database` if you want to keep database code separate.
3. Create a Kotlin file named `UserDatabase.kt`.
4. Paste the code below.
5. Replace the package declaration with your own app package. For example, if your namespace is `com.example.myapplication`, use `package com.example.myapplication.database`.

```kotlin
package com.example.myapplication.database

import android.content.ContentValues
import android.content.Context
import android.database.sqlite.SQLiteConstraintException
import android.database.sqlite.SQLiteDatabase
import android.database.sqlite.SQLiteOpenHelper
import java.security.MessageDigest
import java.security.SecureRandom
import javax.crypto.SecretKeyFactory
import javax.crypto.spec.PBEKeySpec

class UserDatabase(context: Context) :
    SQLiteOpenHelper(context, DATABASE_NAME, null, DATABASE_VERSION) {

    companion object {
        private const val DATABASE_NAME = "user_accounts.db"
        private const val DATABASE_VERSION = 1

        private const val TABLE_USERS = "users"
        private const val COL_ID = "id"
        private const val COL_USERNAME = "username"
        private const val COL_SALT = "salt"
        private const val COL_PASSWORD_HASH = "password_hash"

        private const val SALT_LENGTH = 16
        private const val HASH_BITS = 256
        private const val HASH_ITERATIONS = 120_000
    }

    override fun onCreate(db: SQLiteDatabase) {
        val createTable = """
            CREATE TABLE $TABLE_USERS (
                $COL_ID INTEGER PRIMARY KEY AUTOINCREMENT,
                $COL_USERNAME TEXT NOT NULL UNIQUE,
                $COL_SALT BLOB NOT NULL,
                $COL_PASSWORD_HASH BLOB NOT NULL
            )
        """.trimIndent()

        db.execSQL(createTable)
    }

    override fun onUpgrade(
        db: SQLiteDatabase,
        oldVersion: Int,
        newVersion: Int
    ) {
        // No schema changes are needed in version 1.
        // Add a deliberate upgrade path here if the table structure changes.
    }

    /** Registers an account. Returns false if the username already exists. */
    fun registerUser(username: String, password: String): Boolean {
        val salt = generateSalt()
        val passwordHash = hashPassword(password, salt)

        val values = ContentValues().apply {
            put(COL_USERNAME, username)
            put(COL_SALT, salt)
            put(COL_PASSWORD_HASH, passwordHash)
        }

        return try {
            writableDatabase.insertOrThrow(TABLE_USERS, null, values)
            true
        } catch (exception: SQLiteConstraintException) {
            false
        }
    }

    /** Returns true only when the account exists and its password is correct. */
    fun loginUser(username: String, password: String): Boolean {
        val query = """
            SELECT $COL_SALT, $COL_PASSWORD_HASH
            FROM $TABLE_USERS
            WHERE $COL_USERNAME = ?
            LIMIT 1
        """.trimIndent()

        readableDatabase.rawQuery(query, arrayOf(username)).use { cursor ->
            if (!cursor.moveToFirst()) return false

            val salt = cursor.getBlob(cursor.getColumnIndexOrThrow(COL_SALT))
            val savedHash = cursor.getBlob(
                cursor.getColumnIndexOrThrow(COL_PASSWORD_HASH)
            )
            val enteredHash = hashPassword(password, salt)

            return MessageDigest.isEqual(savedHash, enteredHash)
        }
    }

    private fun generateSalt(): ByteArray {
        return ByteArray(SALT_LENGTH).also { bytes ->
            SecureRandom().nextBytes(bytes)
        }
    }

    private fun hashPassword(password: String, salt: ByteArray): ByteArray {
        val spec = PBEKeySpec(
            password.toCharArray(),
            salt,
            HASH_ITERATIONS,
            HASH_BITS
        )

        return try {
            SecretKeyFactory
                .getInstance("PBKDF2WithHmacSHA256")
                .generateSecret(spec)
                .encoded
        } finally {
            spec.clearPassword()
        }
    }
}
```

### Understanding the database helper

- `SQLiteOpenHelper` creates and manages the database file for the application.
- `onCreate()` runs when the database is first created. It creates the `users` table.
- `onUpgrade()` is where you write changes when you increase `DATABASE_VERSION`.
- `ContentValues` holds the values to insert into a row.
- `insertOrThrow()` attempts to insert the record. The unique constraint prevents two accounts from using the same username.
- `rawQuery()` runs a parameterised query. The `?` placeholder keeps the username value separate from the SQL statement.
- `use { ... }` closes the database cursor after the query is complete.
- `MessageDigest.isEqual()` compares the saved and newly derived password values.

**Important:** Do not build SQL by concatenating user-entered text into the query. Use placeholders and selection arguments as shown.

---

## 7. Practical task B: Build the Compose interface

1. Open `MainActivity.kt`.
2. Replace its contents with the code below.
3. Change the package declaration to match your project.
4. Check that your project has the Material 3 and Activity Compose dependencies supplied by the Compose project template.

```kotlin
package com.example.myapplication

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.PasswordVisualTransformation
import androidx.compose.ui.unit.dp
import com.example.myapplication.database.UserDatabase
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.withContext

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val userDatabase = UserDatabase(applicationContext)

        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    LoginRegisterScreen(userDatabase)
                }
            }
        }
    }
}

@Composable
fun LoginRegisterScreen(database: UserDatabase) {
    var isRegisterMode by remember { mutableStateOf(false) }
    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    var message by remember { mutableStateOf("") }
    var loggedInUser by remember { mutableStateOf<String?>(null) }
    var isWorking by remember { mutableStateOf(false) }

    val coroutineScope = rememberCoroutineScope()

    if (loggedInUser != null) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Text(
                text = "Welcome, $loggedInUser!",
                style = MaterialTheme.typography.headlineSmall
            )
            Spacer(modifier = Modifier.height(20.dp))
            Button(onClick = {
                loggedInUser = null
                username = ""
                password = ""
                message = "You have logged out."
            }) {
                Text("Log out")
            }
        }
    } else {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally,
            verticalArrangement = Arrangement.Center
        ) {
            Text(
                text = if (isRegisterMode) "Create account" else "Login",
                style = MaterialTheme.typography.headlineMedium
            )

            Spacer(modifier = Modifier.height(20.dp))

            OutlinedTextField(
                value = username,
                onValueChange = { username = it },
                label = { Text("Username") },
                singleLine = true,
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(modifier = Modifier.height(12.dp))

            OutlinedTextField(
                value = password,
                onValueChange = { password = it },
                label = { Text("Password") },
                singleLine = true,
                visualTransformation = PasswordVisualTransformation(),
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(modifier = Modifier.height(16.dp))

            Button(
                enabled = !isWorking,
                modifier = Modifier.fillMaxWidth(),
                onClick = {
                    val cleanUsername = username.trim()

                    if (cleanUsername.isBlank() || password.isBlank()) {
                        message = "Please enter both a username and password."
                    } else if (password.length < 6) {
                        message = "Password must contain at least 6 characters."
                    } else {
                        isWorking = true
                        message = ""

                        coroutineScope.launch {
                            val success = withContext(Dispatchers.IO) {
                                if (isRegisterMode) {
                                    database.registerUser(cleanUsername, password)
                                } else {
                                    database.loginUser(cleanUsername, password)
                                }
                            }

                            isWorking = false

                            if (success) {
                                if (isRegisterMode) {
                                    message = "Registration successful. You can now log in."
                                    isRegisterMode = false
                                    password = ""
                                } else {
                                    loggedInUser = cleanUsername
                                    message = ""
                                }
                            } else {
                                message = if (isRegisterMode) {
                                    "That username is already registered. Try another one."
                                } else {
                                    "Incorrect username or password."
                                }
                            }
                        }
                    }
                }
            ) {
                Text(
                    when {
                        isWorking -> "Please wait..."
                        isRegisterMode -> "Register"
                        else -> "Login"
                    }
                )
            }

            Spacer(modifier = Modifier.height(12.dp))

            Button(
                enabled = !isWorking,
                onClick = {
                    isRegisterMode = !isRegisterMode
                    password = ""
                    message = ""
                }
            ) {
                Text(
                    if (isRegisterMode) "Already registered? Login"
                    else "New user? Register"
                )
            }

            Spacer(modifier = Modifier.height(16.dp))

            if (message.isNotBlank()) {
                Text(text = message)
            }
        }
    }
}
```

### How the Compose screen works

- `mutableStateOf()` creates observable state. When a state value changes, Compose recomposes the parts of the interface that read it.
- `remember` keeps the state while this composable remains in the composition.
- `OutlinedTextField` displays an input field. `onValueChange` updates the corresponding state variable.
- `PasswordVisualTransformation()` hides the password characters on screen.
- `Button` runs its `onClick` action when tapped.
- `rememberCoroutineScope()` provides a coroutine scope tied to the composable's lifecycle.
- `withContext(Dispatchers.IO)` moves the database operation to an IO dispatcher rather than running it on the main UI thread.
- `isWorking` temporarily disables the buttons while a database operation is in progress.
- `loggedInUser` determines whether the login form or welcome screen is displayed.

The sample keeps the screen deliberately small so that the focus is on registration, login, and persistence. A larger application would normally place database operations in a repository and a ViewModel rather than calling the database helper directly from the screen.

---

## 8. Practical task C: Run and test the app

Build and run the application on an emulator or Android device.

### Test 1: Register a new user

1. Start the app.
2. Tap **New user? Register**.
3. Enter a username such as `student01`.
4. Enter a password with at least six characters.
5. Tap **Register**.
6. Confirm that the app displays a registration success message and returns to the login mode.

**Expected result:** The account is saved in the local SQLite database.

### Test 2: Log in with the correct details

1. Enter the username you registered.
2. Enter the same password.
3. Tap **Login**.

**Expected result:** The app displays a welcome message containing the username.

### Test 3: Reject an incorrect password

1. Log out.
2. Enter the registered username.
3. Enter an incorrect password.
4. Tap **Login**.

**Expected result:** The app displays an error message and does not log the user in.

### Test 4: Reject a duplicate username

1. Switch to registration mode.
2. Enter the same username used in Test 1.
3. Enter a password.
4. Tap **Register**.

**Expected result:** The app reports that the username is already registered. The unique database constraint prevents a duplicate account.

### Test 5: Check persistence

1. Register an account if you have not already done so.
2. Close the app completely.
3. Reopen the app.
4. Log in with the previously registered details.

**Expected result:** The account is still available. The data is stored in the app's local database rather than only in the screen's temporary state.

### Test 6: Check input validation

Try to register with an empty username, an empty password, and a password shorter than six characters.

**Expected result:** The app displays a validation message and does not attempt to save an incomplete account.

---

## 9. Troubleshooting

| Problem | Possible cause | What to check |
|---|---|---|
| `Unresolved reference: UserDatabase` | Package or import does not match | Check the package declaration in `UserDatabase.kt` and the import in `MainActivity.kt`. |
| `Unresolved reference: kotlinx` | Coroutines dependency is missing | Add the Android coroutines dependency and sync the project. |
| App crashes when opening the database | SQL statement or database code has an error | Check the table name, column names, SQL syntax, and Logcat error. |
| Registration always reports a duplicate | The username was already saved | Try a new username. For a fresh test database, uninstall and reinstall the app or clear its app data. |
| Login always fails | Username/password mismatch or hash settings changed | Use the exact username and password from registration. Keep the salt and hash parameters consistent. |
| UI appears frozen | Database work may be running on the main thread | Ensure database calls are inside `withContext(Dispatchers.IO)`. |
| `PBKDF2WithHmacSHA256` is unavailable | Device API or security provider does not support the algorithm | Test on a supported API level or use a vetted compatible cryptographic library. |

**Note about clearing app data:** Uninstalling the app or clearing its storage deletes this example's local database. Do not do this if you need to preserve the test accounts.

---

## 10. Extension activities

Complete at least two of the following activities.

### Extension A: Add an email address

Add an `email` column to the `users` table and update the registration form to collect it.

Tasks:
1. Update the SQL table definition.
2. Add an email input field to the Compose screen.
3. Validate that the email field is not empty.
4. Save the email during registration.
5. Retrieve and display the email after a successful login.

Remember that changing the table structure requires a database upgrade strategy. For a classroom experiment, you can uninstall the app to create a fresh database, but a real app must preserve existing user data through a planned upgrade.

### Extension B: Add a password confirmation field

During registration, ask the user to enter the password twice.

Tasks:
1. Add a second password state variable.
2. Add a second password field.
3. Check that both password entries match before saving the account.
4. Display a useful message if the values do not match.

### Extension C: Add a show/hide password option

Add a control that allows the user to reveal or hide the password while typing.

Tasks:
1. Create a Boolean state variable.
2. Change the field's visual transformation based on that state.
3. Add a clickable label or icon to toggle the state.

### Extension D: Add a profile screen

After login, show a simple profile screen with the username and a sign-out action. You may add further fields such as full name or course.

### Extension E: Add account deletion

Add a `deleteUser(username: String)` function to the database helper. Add a button on the profile screen to delete the current account after the user confirms the action.

Consider what should happen if the database delete operation fails or no matching account exists.

### Extension F: Improve the app structure

Refactor the application so that the Compose screen does not call `UserDatabase` directly.

Suggested structure:

```text
com.example.myapplication
|-- database
|   |-- UserDatabase.kt
|-- data
|   |-- UserRepository.kt
|-- ui
|   |-- LoginRegisterScreen.kt
|-- MainActivity.kt
```

Create a repository class to provide a clean interface for registration and login. For a more advanced version, introduce a ViewModel to hold screen state and coordinate operations.