---
title: "Enhancement One: Software Design & Engineering"
date: 2023-06-25T00:20:00-04:00
---

---

&emsp;The artifact I have selected to work on improving to exhibit my abilities in software design and engineering is the [Inventory application](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming) I created almost a year ago for my Mobile Architecture and Development class. Specifically, I rewrote the Login Activity in Kotlin instead of Java because it is considered by Google to be the best new way to develop Android apps. The Login screen is responsible for getting a user’s input email and password. Then, the user can select to either login or create an account to store in the Login database.

&emsp;Firstly, my [Kotlin Login application](https://github.com/trevor-leon/CS-499-Kotlin-Login) is a good exhibition of my diverse programming language knowledge, as well as my ability to *employ strategies for building collaborative environments that enable diverse audiences to support organizational decision-making in the field of computer science* by implementing separation of concerns within the code, dependency injection for the app, and adding useful comments as I wrote my code. Additionally, I showcased that I *designed and evaluated computing solutions that solve a given problem using algorithmic principles and computer science practices and standards appropriate to its solution while managing the trade-offs involved in the process* by rewriting the screen in Kotlin. The artifact is being improved in general because Kotlin is widely considered to be superior to Java. After all, it provides additional security such as nullability and type safety that Java doesn’t necessarily come with. This *demonstrates that I have developed a security mindset that anticipates adversarial exploits in software architecture and designs to expose potential vulnerabilities, mitigate design flaws, and ensure privacy and enhanced security of data and resources.* Lastly, my code review and my excellent writing abilities *demonstrate that I designed, developed, and delivered professional-quality oral, written, and visual communications that are coherent, technically sound, and appropriately adapted to specific audiences and contexts.*

&emsp;Writing screens for an Android application following best practices such as separation of concerns and dependency injection was certainly more complex than my original simpler Java project. I didn’t know much about Kotlin when I started the project, but I know a lot more now after I reviewed the Android Developer guides. It was a big learning curve, but I successfully redesigned the Login screen of the app using best practices.

---

## Enhancement Code Examples

The original Login activity that I designed for my Inventory app for CS-360 was written in Java and XML, and does not utilize dependency injection.

### [LoginActivity.java](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming/blob/master/app/src/main/java/com/example/inventory/LoginActivity.java)

```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        // Enable a background thread on a later version
        // Get a one-of of the LoginDatabase, and then initialize logins.
        loginDB = LoginDatabase.getInstance(getApplicationContext());

        loginButton = findViewById(R.id.loginButton);
        createAccountButton = findViewById(R.id.createAccountButton);
        forgotPassword = findViewById(R.id.forgotPassword);

        loginButton.setEnabled(false);
        createAccountButton.setEnabled(false);
        forgotPassword.setEnabled(false);

        usernameText = findViewById(R.id.editTextTextUsername);
        passwordText = findViewById(R.id.editTextTextPassword);

        loginButton.setOnClickListener(view -> {
            /*
              If the user tries to login and the username and password are in the database,
              then they should be redirected into the inventory activity. If they are not in
              the database, they should instead be shown a toast explaining what went wrong.
              Username/password verification: If the logins HashMap's get function returns
              the password for the specified, and that password matches the provided password; open
              the inventory activity. Else, show "Invalid Email or password." toast.
             */
            boolean passwordVerified = Objects.equals(loginDB.getLogins().get(usernameText.getText().toString()),
                    passwordText.getText().toString());
            if (passwordVerified) {
                // Open the onHand inventory screen
                Intent intent = new Intent(getApplicationContext(), InventoryActivity.class);
                startActivity(intent);
            } else {
                // Else, show a toast explaining "Invalid Email or password"
                Toast.makeText(getApplicationContext(),
                        R.string.invalid_username_password_toast, Toast.LENGTH_LONG).show();
                usernameText.setText("");
                passwordText.setText("");
            }
        });
...
```

### [ActivityLogin.xml](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming/blob/master/app/src/main/res/layout/activity_login.xml)

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/black"
    tools:context=".LoginActivity">

    <ImageView
        android:id="@+id/logo"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:contentDescription="@string/logo"
        android:src="@drawable/ic_baseline_front_hand_24"
        app:layout_constraintBottom_toTopOf="@+id/editTextTextUsername"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
...
```

It also used only one color palette rather than letting the user choose to operate in dark or light mode on their device.

### [themes.xml](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming/blob/master/app/src/main/res/values/themes.xml)

```
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme. -->
    <style name="Theme.Inventory" parent="Theme.MaterialComponents.DayNight.DarkActionBar">
        <!-- Customize your theme here. -->
        <!-- Primary brand color. -->
        <item name="colorPrimary">@color/brandColor</item>
        <item name="colorPrimaryVariant">@color/black</item>
        <item name="colorOnPrimary">@color/white</item>
        <!-- Secondary brand color. -->
        <item name="colorSecondary">@color/gray</item>
        <item name="colorSecondaryVariant">@color/gray</item>
        <item name="colorOnSecondary">@color/white</item>
        <!-- Status bar color. -->
        <item name="android:statusBarColor">?attr/colorPrimaryVariant</item>
    </style>

</resources>
```

---

My new recreated Login application is developed with Kotlin applying dependency injection, separation of concerns, and applies a theme to the whole app that changes depending on the user's dark mode preferences.

### [MainActivity.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/MainActivity.kt)

```
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            KotlinLoginTheme {
                // A surface container using the 'background' color from the theme
                LoginApp()
            }
        }
    }
}
```

### [Theme.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/ui/theme/Theme.kt)

```
...
/**
 * Set the theme for the app according to the user's dark mode preferences
 */
@Composable
fun KotlinLoginTheme(
    useDarkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val systemUiController = rememberSystemUiController()
    val colors = if (!useDarkTheme) {
        systemUiController.setStatusBarColor(color = Color.White)
        LightColors
    } else {
        systemUiController.setStatusBarColor(color = Color.Black)
        DarkColors
    }

    MaterialTheme(
        colorScheme = colors,
        typography = Typography,
        content = content
    )
}
```

The LoginApp() function sets up the LoginNavHost, which define the screens of the app.

### [KotlinInventoryApp.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/KotlinInventoryApp.kt)

```
/**
 * Top-level Composable to manage the screens of the application called from MainActivity
 */
@Composable
fun LoginApp(navController: NavHostController = rememberNavController()) {
    LoginNavHost(navController = navController)
}
...
```

### [InventoryNavGraph.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/ui/navigation/InventoryNavGraph.kt)

```
/**
 * Provide the navigation and composables for the app
 */
@Composable
fun LoginNavHost(
    navController: NavHostController,
    modifier: Modifier = Modifier
) {
    /**
     * Set up the NavHost; specifying the start destination as Login
     */
    NavHost(
        navController = navController,
        startDestination = LoginDestination.route,
        modifier = modifier
    ) {
        composable(route = LoginDestination.route) {
            LoginScreen(
                // Implement navigation function here
                // Set the navigation destination to the successful login screen
                navigateToNextScreen = { navController.navigate(SuccessDestination.route) }
            )
        }
        // Implement other composables
        composable(route = SuccessDestination.route) {
            SuccessfulLoginScreen(
                onNavigateUp = { navController.navigateUp() }
            )
        }
    }
}
```
