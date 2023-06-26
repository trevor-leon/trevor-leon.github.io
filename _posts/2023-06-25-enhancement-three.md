---
title: "Enhancement Three: Databases"
date: 2023-06-25T00:10:00-04:00
---

---

&emsp;The artifact I have selected to improve to demonstrate my ability in databases is the [Inventory application](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming) I created almost a year ago for CS-360: Mobile Architecture & Programming. Specifically, I rewrote the Login Activity in Kotlin instead of Java because Google considers it the [best new way to develop Android apps](https://developer.android.com/kotlin/first). I wanted to follow best practices and utilize a Room database, which acts as an abstraction of a SQLite database to reduce boilerplate code and avoid common oversights such as SQL injection. The original Inventory app used regular SQLite databases, did not store IDs as primary keys, and was not encrypted. It also created a HashMap of all the Logins from the Login database to verify if the username/password combination is correct rather than checking the database itself.

&emsp;For my [Kotlin Login activity](https://github.com/trevor-leon/CS-499-Kotlin-Login), I **demonstrated an ability to use well-founded and innovative techniques, skills, and tools in computing practices for the purpose of implementing computer solutions that deliver value and accomplish industry-specific goals** by implementing the recommended pipeline of components to utilize the database such as Room entities defining each row or object of the database; the data access objects (DAOs) that define operations such as insert, update, delete, and queries; and repository that utilizes the DAOs to manipulate the data in the database. Additionally, I added IDs to each user as a primary key in the database and ensured each email address is unique. Lastly, I added and utilized direct data access functions to determine if a login is in the database rather than getting a HashMap and verifying it with that. These factors showcase that I have **designed and evaluated computing solutions that solve a given problem using algorithmic principles and computer science practices and standards appropriate to its solution while managing the trade-offs involved in design choices.**

&emsp;When developing my Room database, I learned a lot about Room itself, database best practices, and the pipeline to access data within the database when developing Android apps with Kotlin. I also learned the importance of planning queries that are required for the development of the app. For example, I initially did not realize that I would need both a query to verify a login attempt and a query to determine if a username already exists in the database. I realized these were both required because of the different needs of both the login and account creation buttons. The account creation button only needs to determine if the input username is already in the database, while the login button needs both the input username and password.

---

## Enhancement Plan

### Kotlin Login Overview

<img width="976" alt="DatabaseDaoEntityRelationship" src="https://github.com/trevor-leon/trevor-leon.github.io/assets/72781990/bf7b8bb5-3de1-4c3d-8790-80fa9201a1bd">


---

## Enhancement Code Examples

The original Login activity that I designed for my Inventory app for CS-360: Mobile Architecture & Programming was made utilizing a recommended helper class for Java projects, SQLiteOpenHelper.

### [LoginDatabase.java](https://github.com/trevor-leon/CS-360_Mobile_Arch_and_Programming/blob/master/app/src/main/java/com/example/inventory/LoginDatabase.java)

```
public class LoginDatabase extends SQLiteOpenHelper {

    // Model

    private static final String DATABASE_NAME = "login.db";
    private static final int VERSION = 1;

    private final HashMap<String, String> logins;

    private static LoginDatabase loginDatabase;

    public static LoginDatabase getInstance(Context context) {
        if (loginDatabase == null) {
            loginDatabase = new LoginDatabase(context);
        }
        return loginDatabase;
    }

    public LoginDatabase(Context context) {
        super(context, DATABASE_NAME, null, VERSION);
        logins = new HashMap<>();
    }

    private static final class LoginTable {
        private static final String TABLE = "Login";
        private static final String COL_USERNAME = "Username";
        private static final String COL_PASSWORD = "Password";
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        // Database creation: create table Login (Username text PRIMARY KEY, Password text NOT NULL)
        sqLiteDatabase.execSQL("create table " + LoginTable.TABLE + " (" +
                LoginTable.COL_USERNAME + " text PRIMARY KEY, " +
                LoginTable.COL_PASSWORD + " text NOT NULL )");
    }

...
```

My Kotlin Login app uses an encrypted Room database as an abstraction of a SQLite database to securely store Logins using the pipeline of Room entities to define the rows of the database, data access objects (DAOs) to query the database, and repositories to use the DAOs.

The Login Entities are defined with an id that is autogenerated and a unique username.

### [Login.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/data/login/Login.kt)

```
/**
 * Entity data class to represent a row of the Login database
 * Set the id and username to be unique
 */
@Entity(indices = [Index(value = ["username"], unique = true)])
data class Login(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val username: String,
    val password: String
)
```

The LoginDao class specifies the queries that can be performed on the Login database. Room makes defining common queries such as insert, update, and delete simple.

### [LoginDao.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/data/login/LoginDao.kt)

```
/**
 * Define the Data Access Object (DAO) for the Login DB
 */
@Dao
interface LoginDao {

    // Specify the OnConflictStrategy to ignore conflicting inserts into the Login DB
    @Insert(onConflict = OnConflictStrategy.IGNORE)
    suspend fun insert(login: Login)

    @Update
    suspend fun update(login: Login)

    @Delete
    suspend fun delete(login: Login)

    // Determine if a login exists in the database; used for validating logins
    @Query("SELECT EXISTS(SELECT * FROM Login WHERE username = :username AND password = :password)")
    suspend fun loginValid(username: String, password: String): Boolean

    // Determine if a username already exists in the database; used to confirm storage
    @Query("SELECT EXISTS(SELECT * FROM Login WHERE username = :username)")
    suspend fun usernameExists(username: String): Boolean
}
```

These classes are applied directly by the database in its creation.

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

The repository classes utilize the Daos and are used by the LoginViewModel for database access. The LoginViewModel bridges the gap between the user interface (UI) and the database.

### [LoginViewModel.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/ui/login/LoginViewModel.kt)

```
/**
 * ViewModel uses the UI State, repositories, and generally represents the functions of the app
 */
class LoginViewModel (private val loginsRepository: LoginsRepository) : ViewModel() {

    // Hold the current Login UI state
    var loginUiState by mutableStateOf(LoginUiState())
        private set

    /**
     * Update the private set loginUiState to the passed loginDetails
     */
    fun updateUiState(
        loginDetails: LoginDetails = loginUiState.loginDetails,
        isUsernameInvalid: Boolean = false,
        isPasswordInvalid: Boolean = false
    ) {
        loginUiState = LoginUiState(
            loginDetails = loginDetails,
            isUsernameInvalid = isUsernameInvalid,
            isPasswordInvalid = isPasswordInvalid
        )
    }

    /**
     * Get the login from the database according to the username and password
     * and verify that it matches the UI state
     */
    suspend fun findLogin(uiState: LoginDetails = loginUiState.loginDetails): Boolean {
        // First, validate the login as an invalid login should never be entered.
        // If the login exists, return true. Else, return false.
        return if (validateLogin()) {
            loginsRepository.loginExists(
                uiState.username,
                uiState.password
            )
        } else {
            false
        }
    }

    /**
     * Validate and save the current loginUiState details if the username and password
     */
    suspend fun saveLogin(uiState: LoginDetails = loginUiState.loginDetails): Boolean {
        // Validate the login before checking it
        return if (!findUsername()) {
            // If the username was not found in the database, store the login, and return true
            if(validateLogin()) {
                loginsRepository.insertLogin(uiState.toLogin())
                true
            } else {
                false
            }
        } else {
            false
        }
    }

...
```

### [OfflineLoginsRepository.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/data/login/OfflineLoginsRepository.kt)

```
/**
 * Offline repository representation of a [LoginsRepository] that uses the [LoginDao] to access logins
 */
class OfflineLoginsRepository(private val loginDao: LoginDao) : LoginsRepository {

    // Determine whether or not the username and password combination exists in the database.
    override suspend fun loginExists(username: String, password: String): Boolean =
        loginDao.loginValid(username, password)

    // Determine whether or not the username exists in the database.
    override suspend fun usernameExists(username: String): Boolean =
        loginDao.usernameExists(username)

    override suspend fun insertLogin(login: Login) = loginDao.insert(login)

    override suspend fun deleteLogin(login: Login) = loginDao.delete(login)

    override suspend fun updateLogin(login: Login) = loginDao.update(login)

}
```

### [LoginsRepository.kt](https://github.com/trevor-leon/CS-499-Kotlin-Login/blob/master/app/src/main/java/com/example/kotlinlogin/data/login/LoginsRepository.kt)

```
/**
 * Repository to read, insert, update, and delete [Login] from a given data source.
 */
interface LoginsRepository {
    // Retrieve a login from the given data source that matches the username and password
    suspend fun loginExists(username: String, password: String): Boolean

    // Determine if a username already exists within the database
    suspend fun usernameExists(username: String): Boolean

    // Insert a login into the data source
    suspend fun insertLogin(login: Login)

    // Delete a login in the data source
    suspend fun deleteLogin(login: Login)

    // Update a login in the data source
    suspend fun updateLogin(login: Login)

}
```

---
