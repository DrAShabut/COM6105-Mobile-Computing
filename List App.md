List Demo App
Of course, our app users won't be running our apps in Android Studio and will expect to see their shopping list neatly formatted in the Android UI on their device screen. Fortunately, Jetpack makes this very straightforward.

Delete the Greeting function definition and GreetingPreview. Inside setContent, the template already has a Scaffold(...) { innerPadding -> ... }; keep that Scaffold and replace only the Greeting(...) call inside it with the ShoppingList(...) call below (that's where innerPadding comes from).
In the onCreate function, add this code to declare and initialize our List in the usual way. Place this line of code before the call to the setContent function.
val shoppingList = listOf("Bread", "Butter", "Milk")
Next, in the setContent call, replace the Greeting composable call with a call to the new ShoppingList composable. Here is the code.
ShoppingList(
    items = shoppingList,
    modifier = Modifier.padding(innerPadding)
)
There will be an error in the code until we create the ShoppingList function. Finally, add a new composable called ShoppingList.
@Composable
fun ShoppingList(items: List<String>, modifier: Modifier = Modifier) {
    Column(modifier = modifier.padding(16.dp)) {
        Text(
            text = "Shopping List",
            style = MaterialTheme.typography.headlineLarge
        )
     
        Spacer(modifier = Modifier.height(8.dp))
     
        for (item in items) {
            Text(text = "• $item", style =
               MaterialTheme.typography.bodyLarge)
        }
    }
}
Run the app
We have seen all the code in this composable before, but here is a quick refresher on what each part does.

The @Composable annotation marks a function for the attention of the Compose compiler
Column instructs Jetpack to stack elements vertically
Text displays the text Shopping List
Just for presentation purposes, I added a style to the shopping list heading with headlineLarge and passed it in as a named parameter to the Text composable
Spacer adds a space in the column for neatness
The code for (item in items) adds a call to Text for each item in the list, which will be stacked neatly by the call to Column
Again, for presentation purposes, I added the style bodyLarge to each Text composable
The UI looks like the following image.

Figure 10.1 - Simple List in a Column
Figure 10.1 - Simple List in a Column

We have seen two simple ways to display the contents of a List. We can log it to logcat with a single line of code, or we can display its contents in the Android UI using a composable function and a for loop that iterates over each item. We have seen something similar, but this is the first time we have fully understood the list-based code and also the Jetpack code.

Let's push our knowledge a bit further with another demo app that lets the user enter text and add it to a list.

TextField-to-list app
As we know more about composables and we are very well versed in lists, I thought an extra demo using a list, remember, and a TextField would fit nicely. We saw a TextField right back in Chapter 2 with the Enter Your Name app, but now we can keep entering values, store them nearly infinitely in a list, and display them easily in a LazyColumn. Most of the code in this app should look familiar, but the dynamic list effect is new. Since the new code is fairly limited, we will explain it as we proceed.

Create a new Empty Activity project called Text to List. Delete the Greeting and the GreetingPreview functions.
Add/replace the required import statements
package com.example.texttolist

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.Button
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.example.texttolist.ui.theme.TextToListTheme
Code the first composable function called ListOfEntries, which has a list of Strings as a parameter and loops through that list, adding each String to a Text. Notice that each Text has vertical padding applied to make the list look tidy. Furthermore, you might notice something a bit different: we are passing a parameter (.verticalScroll(rememberScrollState())) to a regular Column layout instead of using LazyColumn. LazyColumn is designed for efficiency with long lists. A regular Column layout is more flexible (we just loop through a list and output some Text widgets). This is fine for short lists, and now you have seen both options.
@Composable
fun ListOfEntries(entries: List<String>) {
    // Display the list with scrolling
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .verticalScroll(rememberScrollState()) // Scrollable list
            .padding(16.dp) // Padding for the list content
    ) {
        entries.forEach {
            Text(text = it, modifier = Modifier.padding(
                vertical = 4.dp))
        }
    }
}
Now, code the second composable function we discussed, called TextInputAndButton
@Composable
fun TextInputAndButton(
    text: String,
    onTextChange: (String) -> Unit,
    onAddClick: () -> Unit
) {
    // Row layout for input and button at the bottom of the screen
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        // TextField for input (bottom left)
        TextField(
            value = text,
            onValueChange = onTextChange,
            label = { Text("Enter text") },
            modifier = Modifier
                .fillMaxWidth(0.8f) // Take most of the width
                .padding(end = 16.dp)
        )

        // Button to add the text to the list (bottom right)
        Button(
            onClick = onAddClick,
            modifier = Modifier.align(Alignment.CenterVertically)
        ) {
            Text("Add")
        }
    }
}
In the preceding code, the second composable function called TextInputAndButton, notice in the signature the onTextChange and onAddClick text—this is new. The text String parameter is what we have seen many times before; it represents what is currently typed in the input. A function called onTextChange, which runs whenever the user types, and another called onAddClick, which runs when the button is pressed, are also passed as parameters.

It isn't as complicated as it seems. We are passing a function to a function—that's it. We will dig deeper into passing functions as parameters when we discuss lambdas in Chapter 13. For now, just understand that as those functions are present as parameters, they can be used from the scope of this function.

Inside the function, a Row layout is used to place elements side by side horizontally, stretching across the full width of the screen with some padding around it and spacing between its contents.

Within this row, we have our TextField widget on the left and a Button widget on the right. The text field displays the current text and calls the provided (as a parameter) onTextChange function whenever the user updates the input, allowing the code we will see in the next section to control and store the text. It is set to take up most of the available width, so it appears larger than the button. The button is vertically centered and displays the label "Add"; when clicked, it triggers the onAddClick function (also passed in as a parameter).

Identify the opening curly brace of the TextToListTheme function's opening curly brace as shown below
setContent {
    TextToListTheme {
Identify all the content within the TextToListTheme function, as shown next, and delete it
Scaffold(
    modifier = Modifier.fillMaxSize()
    ) { innerPadding ->
        Greeting(
            name = "Android",
            modifier = Modifier.padding(innerPadding)
    )
}
Replace the deleted code inside the opening and closing curly braces of the TextToListTheme function call with the following code
val list = remember { mutableStateOf(listOf<String>()) }
val text = remember { mutableStateOf("") }

Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
    Column(modifier = Modifier
    .padding(innerPadding)
    .fillMaxSize()) {

    // Display List of Entries
    ListOfEntries(list.value)

    // Spacer to add some space
    Spacer(modifier = Modifier.weight(1f))

    // Bottom section: Text Input and Button
    TextInputAndButton(
        text = text.value,
        onTextChange = { text.value = it },

        onAddClick = {
            if (text.value.isNotEmpty())
            {
                // Add text to the list
                list.value = list.value + text.value
                text.value = "" // Clear the text input
            }
        }
       )
    }
}
In the preceding code, we store the data and define what should happen when the user interacts with it. At the top, two variables are created using remember and mutableStateOf. The list contains Strings representing all the entries the user has added, while text holds the current value of the input field. The use of remember ensures that these values are preserved across recompositions, meaning when the user types or presses the Add button, Compose automatically updates the UI.

The layout is structured as we have seen before, using a Scaffold, a Column, and at the top of the column, ListOfEntries(list.value) displays the current list of items. This is followed by a Spacer to create a neat vertical space between entries.

We use .value here; by also works but needs the getValue/setValue imports.

At the bottom, the previously coded TextInputAndButton composable is called. This is where the connection to the earlier code becomes clear, and we see how the unusual parameters of TextInputAndButton are passed in. The text parameter is passed as text.value, so the text field shows whatever is currently stored in state. The onTextChange function updates the state whenever the user types, keeping the UI and data in sync. The onAddClick function defines what happens when the button is pressed: it checks that the text is not empty, adds the current text to the list, and then clears the input field.

In other words, the earlier component handled how things look, while this code handles what happens.

Run the app
When we run the app, we can see a field labeled Enter text. This is our TextField; type whatever you like, then press the Add button to add a String to the list.

Figure 10.2 - The Text to List app when first run
Figure 10.2 - The Text to List app when first run

The preceding image shows our empty TextField.

Figure 10.3 - The Text to List app with a line of text added
Figure 10.3 - The Text to List app with a line of text added

The preceding image shows the entered text at the top of the UI, but for space, it does not show the text field and button at the bottom.

Enter as much text as you like, as in this next image.

Figure 10.4 - The Text to List app with lots of text added
Figure 10.4 - The Text to List app with lots of text added

The preceding image shows our list with 4 entries.

We have spent long enough looking at lists and how to display them. Let's recap and then move on.
