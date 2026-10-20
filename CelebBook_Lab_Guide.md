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

### Architecture

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

# Setup

Add the following dependencies to `build.gradle.kts`:

```kotlin
implementation("androidx.navigation:navigation-compose:2.9.0")
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.9.2")
```

Recommended project structure:

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

Create `Contact.kt`
In Android Studio, right-click the com.example.celebbook package, select New → Kotlin Class/File, enter Contact, click Create, and add the following code:

```kotlin
data class Contact(
    val id: Int,
    val name: String,
    val bio: String
)
```

### Checkpoint

✅ Build → Make Project to check project builds successfully.

---

# Activity 2: Create the ViewModel

Create `ContactViewModel.kt`

```kotlin
import androidx.compose.runtime.mutableStateListOf
import androidx.lifecycle.ViewModel

class ContactViewModel : ViewModel() {

    private val _contacts = mutableStateListOf(
        Contact(1, "Taylor Swift",
            "Grammy-winning singer-songwriter."),
        Contact(2, "Keanu Reeves",
            "Renowned actor known for action cinema.")
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

---

# Activity 3: Build the UI

Create ContactItem, ContactListScreen and EditContactScreen as discussed in the lab session.

---

# Activity 4: Implement Navigation

Use Navigation Compose with:

```kotlin
val navController = rememberNavController()
val viewModel: ContactViewModel = viewModel()
```

Create routes:

```text
contact_list
edit_contact/{contactId}
```

Ensure saving updates the ViewModel and returns to the list screen using:

```kotlin
navController.popBackStack()
```

---

# Final Testing

- [ ] App launches successfully.
- [ ] Celebrity list appears.
- [ ] Clicking a contact opens the edit screen.
- [ ] User can edit details.
- [ ] Save updates the contact.
- [ ] Updated details appear in the list.

---

# Knowledge Check

1. Why are data classes useful?
2. What is the role of a ViewModel?
3. Why use LazyColumn?
4. What does remember { mutableStateOf() } do?
5. Why use copy() when updating a Contact?
6. How is contactId passed between screens?
7. What is the purpose of popBackStack()?
