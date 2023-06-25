---
title: "Enhancement Two: Algorithms & Data Structures"
date: 2023-06-25T00:15:00-04:00
---

---

&emsp;The artifact I have selected to improve to prove my ability in algorithms and data structure is the [Inventory application](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming) I created almost a year ago for my Mobile Architecture and Development class. Specifically, I rewrote the Login Activity in Kotlin instead of Java because Google considers it to be the best new way to develop Android apps. I wanted to better expand on the functionality of the activity. The original project simply disabled the login and account creation buttons when there was no input in the email and password text fields as input validation for a login or account creation, and it did not encrypt the login details when storing them. Important user information such as an email address and password should be encrypted. Additionally, the original project did not store IDs for the objects but used the String usernames and item names as the primary keys of each object.
  
&emsp;My [Kotlin Login application](https://github.com/trevor-leon/CS-499-Kotlin-Login) allows the buttons always be displayed, and it properly lets the user know if the input email username does not follow an email address pattern, and the input password follows secure password patterns (uppercase and lowercase letters, numbers, and symbols; for example). This clearly shows that I have *designed and evaluated computing solutions that solve a given problem using algorithmic principles and computer science practices and standards appropriate to its solution while managing the trade-offs.* I *developed a security mindset that anticipates adversarial exploits in software architecture and design to expose potential vulnerabilities, mitigate design flaws, and ensure privacy and enhanced security of data and resources* by creating a CryptoManager class that algorithmically develops an encryption key to store in an encrypted file on the device if the file hasn’t been created yet. Then, this key is used by the SQLCipher class with the Room database to encrypt and decrypt data flowing in and out of it. Regarding the data structure of the Login class, I gave each Login an ID as the primary key, but also ensured that each username/email was unique when storing them.
  
&emsp;Developing this part of the project helped deepen my understanding of algorithms and data structures in general and how important it is to plan them out. It made me think and understand more about what should be included in a data structure, and what can be excluded. Originally, I thought my idea of hiding the buttons when the username and password text fields had no input was unique and innovative, but it wasn’t necessary and included a lot more code. Simply letting the user know when they have input something invalid is much easier, more user-friendly, and more informative. It’s also useful to store an ID for the user because any program that extends this login screen will probably want to pass that user ID to the next screen to use for the application.

---

## Enhancement Code Examples

The original Login activity that I designed for my Inventory app for CS-360 did not validate that a proper email pattern was input into the email text field or that a strong password was input in the password text field before storing it, but I did note in some comments that I wanted that functionality.

### [LoginDatabase.java](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming/blob/master/app/src/main/java/com/example/inventory/LoginDatabase.java)

```
...

    /**
     *
     * @param username the username of the login to add to the database
     * @param password the password of the login to add to the database
     * @return the id of the inserted row or -1 if insert failed
     */
    public boolean addLogin(String username, String password) {
        SQLiteDatabase db = getWritableDatabase();
        ContentValues values = new ContentValues();
        // TODO: possibly check username for valid Email address (@gmail.com, @yahoo.com, etc.)
        values.put(LoginTable.COL_USERNAME, username);
        // TODO: possibly check password strength
        values.put(LoginTable.COL_PASSWORD, password);
        long id = db.insert(LoginTable.TABLE, null, values);
        db.close();
        return id != -1;
    }

...
```

In my Kotlin Login application, both usernames and passwords were checked for their respective patterns. Passwords must have an uppercase and lowercase letter, a number, and a symbol, and be at least ten characters long.

### [LoginViewModel.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/ui/login/LoginViewModel.kt)

```
...

/**
     * Validate the login by checking for an email pattern in the username, and then checking
     * the password for the password pattern
     */
    private fun validateLogin(uiState: LoginDetails = loginUiState.loginDetails): Boolean {
        return with(uiState) {
            // This represents a pattern for an email address
            val emailPattern = Patterns.EMAIL_ADDRESS
            // Check if the provided username matches a valid email pattern
            val matcher: Matcher = emailPattern.matcher(uiState.username.trim())

            /**
             * If the username matches the username pattern,
             * ensure the UiState's username is considered to be valid.
             */
            if(matcher.matches() && username.trim().isNotBlank()) {
                updateUiState(
                    LoginDetails(
                        username = username,
                        password = password),
                    isUsernameInvalid = false
                )
                validatePassword()
                true
            } else {
                /**
                 * Update the UI state setting the [LoginDetails]' username to "" and password to
                 * the current uiState password. Also, set isUsernameInvalid to true.
                 */
                updateUiState(
                    LoginDetails(
                        username = "",
                        password = uiState.password
                    ),
                    isUsernameInvalid = true
                )
                validatePassword()
                false
            }
        }
    }

    private fun validatePassword(uiState: LoginDetails = loginUiState.loginDetails): Boolean {
        return with(uiState) {
            val pattern: Pattern

            /**
             * Password checking code derived from the solution here:
             * https://stackoverflow.com/questions/23214434/regular-expression-in-android-for-password-field
             * Check if the password contains a number, lowercase and uppercase letters, a symbol,
             * and has at least ten characters
             */
            val passwordPattern =
                "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[!@#$%^&*()+=]).{10,}$"

            pattern = Pattern.compile(passwordPattern)
            val matcher: Matcher = pattern.matcher(uiState.password.trim())

            /**
             * If the password matches the password pattern, set isUsernameInvalid to the current
             * state, and ensure the UiState's password is considered to be valid.
             */
            if(matcher.matches() && password.trim().isNotBlank()) {
                updateUiState(
                    LoginDetails(
                        username = username,
                        password = password
                    ),
                    isUsernameInvalid = loginUiState.isUsernameInvalid,
                    isPasswordInvalid = false
                )
                true
            } else {
                /**
                 * Update the UI state setting the [LoginDetails]' password to "" and username to
                 * the current uiState username. Also, set isPasswordInvalid to true.
                 */
                updateUiState(
                    LoginDetails(
                        username = uiState.username,
                        password = ""
                    ),
                    isUsernameInvalid = loginUiState.isUsernameInvalid,
                    isPasswordInvalid = true
                )
                false
            }
        }
    }

}

...
```


Unfortunately, my Inventory application did not encrypt usernames and passwords.

My Kotlin Login app uses a CryptoManager class to generate a cryptographic key that is stored in an encrypted file and is used by the Room database when it is created.

### [LoginDatabase.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/data/login/LoginDatabase.kt)

```
/**
 * Specify the database with the Login class as the entity; without keeping a version backup history
 */
@Database(entities = [Login::class], version = 3, exportSchema = false)
abstract class LoginDatabase : RoomDatabase() {
    // Initialize the LoginDao()
    abstract fun loginDao(): LoginDao
    // Companion object allows access to database methods
    companion object {
        /**
         * Set up an instance of LoginDatabase; annotated Volatile to ensure all reads and writes
         * come from the main memory and is always up-to-date
         */
        @Volatile
        private var Instance: LoginDatabase? = null

        /**
         * Return the instance of the database; otherwise, build the database with the
         * synchronized block to ensure only one instance of the database can be created.
         */
        fun getDatabase(context: Context): LoginDatabase {
            // Initialize/write a generated encrypted key to the encrypted file
            writeEncryptedFile(context, CryptoManager.getKey())
            val passphrase = readEncryptedFile(context)
            return Instance ?: synchronized(this) {
                Room.databaseBuilder(
                    context = context,
                    klass = LoginDatabase::class.java,
                    name = "login_database"
                )
                    // Implement SQLCipher to encrypt the database
                    .openHelperFactory(SupportFactory(passphrase))
                    // Simply destroy and recreate to migrate later if needed.
                    .fallbackToDestructiveMigration()
                    .build()
                    .also { Instance = it }
            }
        }
    }
}
```

### [CryptoManager.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/data/login/CryptoManager.kt)

```
class CryptoManager {
    companion object {
        // Initialize the keystore and then load it
        private val keyStore = KeyStore.getInstance("AndroidKeyStore").apply {
            load(null)
        }
        /**
         * Get the secret key, or create a new key if none exists yet.
         */
        fun getKey(): SecretKey {
            val existingKey = keyStore.getEntry("secret", null) as? KeyStore.SecretKeyEntry
            return existingKey?.secretKey ?: createKey()
        }

        /**
         * Create a cryptographic key to store and use by the SQLCipher database
         */
        private fun createKey(): SecretKey {
            return KeyGenerator.getInstance(ALGORITHM). apply {
                init(
                    KeyGenParameterSpec.Builder(
                        "secret",
                        KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
                    )
                        .setBlockModes(BLOCK_MODE)
                        .setEncryptionPaddings(PADDING)
                        .setUserAuthenticationRequired(false)
                        .setRandomizedEncryptionRequired(true)
                        .build()
                )
            }.generateKey()
        }

        /**
         * Read an encrypted file; used to get the key from the encrypted file
         * Redesigned from: https://developer.android.com/topic/security/data
         */
        fun readEncryptedFile(context: Context): ByteArray {
            // Key generation parameter specification
            val keyGenParameterSpec = MasterKeys.AES256_GCM_SPEC
            val mainKeyAlias = MasterKeys.getOrCreate(keyGenParameterSpec)

            val fileToRead = "encrypted.txt"
            val encryptedFile = EncryptedFile.Builder(
                File(context.filesDir, fileToRead),
                context,
                mainKeyAlias,
                EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
            ).build()

            val inputStream = encryptedFile.openFileInput()
            val byteArrayOutputStream = ByteArrayOutputStream()
            var nextByte: Int = inputStream.read()
            while (nextByte != -1) {
                byteArrayOutputStream.write(nextByte)
                nextByte = inputStream.read()
            }

            return byteArrayOutputStream.toByteArray()

        }

        /**
         * Write to an encrypted file; used to store the key
         * Redesigned from: https://developer.android.com/topic/security/data
         * @param stringToWrite the string to write to the encrypted file
         */
        fun writeEncryptedFile(context: Context, stringToWrite: SecretKey) {
            // Create a file with this name or replace an entire existing file
            // that has the same name. Note that you cannot append to an existing file,
            // and the filename cannot contain path separators.
            val fileToWrite = "encrypted.txt"
            // If the file to write to does not exist, create it.
            if (!File(context.filesDir, fileToWrite).exists()) {
                val keyGenParameterSpec = MasterKeys.AES256_GCM_SPEC
                val mainKeyAlias = MasterKeys.getOrCreate(keyGenParameterSpec)
                val encryptedFile = EncryptedFile.Builder(
                    File(context.filesDir, fileToWrite),
                    context,
                    mainKeyAlias,
                    EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
                ).build()

                val fileContent = stringToWrite
                    .toString()
                    .toByteArray(StandardCharsets.UTF_8)
                encryptedFile.openFileOutput().apply {
                    write(fileContent)
                    flush()
                    close()
                }
            }

        }

        private const val ALGORITHM = KeyProperties.KEY_ALGORITHM_AES
        private const val BLOCK_MODE = KeyProperties.BLOCK_MODE_CBC
        private const val PADDING = KeyProperties.ENCRYPTION_PADDING_PKCS7

        }
}
```



