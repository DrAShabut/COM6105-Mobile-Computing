# Lab Notes: Building the Celeb Book App with Jetpack Compose

## Learning Outcomes

By completing this lab, you will be able to:

- Create Kotlin data classes.
- Manage state using a ViewModel.
- Build Jetpack Compose screens.
- Implement navigation between screens.
- Apply modern Android architecture principles.

---

# Overview

In this lab, you will build **Celeb Book**, a simple multi-screen Android application that allows users to:

1. View a list of celebrities.
2. Edit celebrity details.
3. Save changes and return to the list.

## Architecture

```text
Data Layer
    ↓
ViewModel
    ↓
UI Screens
    ↓
Navigation
```

---

# Before Activity 1: Create the Project

## Step 1: Create a New Android Studio Project

1. Open Android Studio.
2. Select **New Project**.
3. Choose **Empty Activity**.
4. Click **Next**.
5. Configure:
   - Name: `CelebBook`
   - Package Name: `com.example.celebbook`
   - Language: `Kotlin`
   - Minimum SDK: `API 24` or higher
6. Click **Finish**.
7. Wait for Gradle Sync to complete.

✅ Run the starter application to confirm everything works.

---

# Setup

## Step 2: Add Required Dependencies

Open:

```text
app > build.gradle.kts
```

Add:

```kotlin
implementation("androidx.navigation:navigation-compose:2.9.0")
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.9.2")
```

Click **Sync Now**.

---

## Step 3: Create the Package Structure

Create the following structure:

```text
com.example.celebbook
│
├── MainActivity.kt
├── Contact.kt
├── ContactViewModel.kt
│
└── ui
    ├── ContactItem.kt
    ├── ContactListScreen.kt
    └── EditContactScreen.kt
```

---

# Activity 1: Create the Data Model

## Create Contact.kt

Right-click the `com.example.celebbook` package and select:

```text
New → Kotlin Class/File
```

Name the file:

```text
Contact
```

Add:

```kotlin
data class Contact(
    val id: Int,
    val name: String,
    val bio: String
)
```

### Checkpoint

✅ Build → Make Project

---

# Activity 2: Create the ViewModel

Create `ContactViewModel.kt`.

```kotlin
import androidx.compose.runtime.mutableStateListOf
import androidx.lifecycle.ViewModel

class ContactViewModel : ViewModel() {

    private val _contacts = mutableStateListOf(
        Contact(1, "Taylor Swift", "Grammy-winning singer-songwriter."),
        Contact(2, "Keanu Reeves", "Renowned actor known for action cinema.")
    )

    val contacts: List<Contact>
        get() = _contacts

    fun getContactById(id: Int) =
        _contacts.find { it.id == id }

    fun updateContact(updatedContact: Contact) {
        val index = _contacts.indexOfFirst {
            it.id == updatedContact.id
        }

        if (index != -1) {
            _contacts[index] = updatedContact
        }
    }
}
```

### Checkpoint

✅ Project builds successfully.

---

# Activity 3: Build the UI

## Step 1: Create ContactItem.kt

```kotlin
@Composable
fun ContactItem(
    contact: Contact,
    onClick: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .clickable { onClick() }
    ) {
        Text(
            text = contact.name,
            modifier = Modifier.padding(16.dp)
        )
    }
}
```

## Step 2: Create ContactListScreen.kt

```kotlin
@Composable
fun ContactListScreen(
    contacts: List<Contact>,
    onContactClick: (Int) -> Unit
) {
    LazyColumn {
        items(contacts) { contact ->
            ContactItem(
                contact = contact,
                onClick = {
                    onContactClick(contact.id)
                }
            )
        }
    }
}
```

## Step 3: Create EditContactScreen.kt

```kotlin
@Composable
fun EditContactScreen(
    contact: Contact,
    onSaveContact: (Contact) -> Unit
) {
    var name by remember { mutableStateOf(contact.name) }
    var bio by remember { mutableStateOf(contact.bio) }

    Column(modifier = Modifier.padding(16.dp)) {

        OutlinedTextField(
            value = name,
            onValueChange = { name = it },
            label = { Text("Name") }
        )

        OutlinedTextField(
            value = bio,
            onValueChange = { bio = it },
            label = { Text("Biography") }
        )

        Button(
            onClick = {
                onSaveContact(
                    contact.copy(
                        name = name,
                        bio = bio
                    )
                )
            }
        ) {
            Text("Save Changes")
        }
    }
}
```

### Checkpoint

✅ Contact list appears.

✅ User can edit details.

---

# Activity 4: Implement Navigation

## Add Imports

```kotlin
import androidx.navigation.compose.*
import androidx.navigation.NavType
import androidx.navigation.navArgument
import androidx.lifecycle.viewmodel.compose.viewModel
```

## Replace MainActivity Content

Use:

```kotlin
val navController = rememberNavController()
val viewModel: ContactViewModel = viewModel()
```

Create routes:

```text
contact_list
edit_contact/{contactId}
```

Navigate using:

```kotlin
navController.navigate("edit_contact/$contactId")
```

Save and return using:

```kotlin
viewModel.updateContact(updatedContact)
navController.popBackStack()
```

---

# Activity 5: Run the Application

## Start an Emulator

```text
Tools → Device Manager
```

Create a virtual device if required.

## Run the App

```text
Shift + F10
```

---

# Final Testing

- [ ] App launches successfully.
- [ ] Two celebrities are displayed.
- [ ] List is scrollable.
- [ ] Clicking a celebrity opens the edit screen.
- [ ] Name field can be edited.
- [ ] Biography field can be edited.
- [ ] Save returns to the list.
- [ ] Updated information appears.
- [ ] No crashes occur.

---

# Common Problems

## Unresolved Reference: Contact

```kotlin
import com.example.celebbook.Contact
```

## Navigation Classes Not Found

Verify:

```kotlin
implementation("androidx.navigation:navigation-compose:2.9.0")
```

## Blank Screen

Verify:

```kotlin
startDestination = "contact_list"
```

---

# Knowledge Check

1. Why are data classes useful?
2. What is the role of a ViewModel?
3. Why use LazyColumn?
4. What does `remember { mutableStateOf() }` do?
5. Why use `copy()` when updating a Contact?
6. How is `contactId` passed between screens?
7. What is the purpose of `popBackStack()`?
