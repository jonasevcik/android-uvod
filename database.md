# Database

### SQLite

In the old days, there was just a plain SQLite database. It wasn't exactly easy to use. One had to define multiple classes full of boilerplate code to fulfill basic CRUD operations. Furthermore, it was designed to be used with [`Cursors`](https://developer.android.com/reference/android/database/Cursor). `Cursors` aren't necessarily a bad concept, it just isn't natural to use them, since it doesn't return data as an object, but one has to extract its values to compose an object first. Nowadays, it is discouraged to use SQLite directly. The recommended way is to use its wrapper, or ORM if you will - Room.

### Room

Room is a data persistence library that builds on top of SQLite and provides an easy-to-use experience along with compile-time schema verification. We'll cover the basics, for more information, visit the [official guide](https://developer.android.com/training/data-storage/room).

#### Database

The database is a special class grouping data entities and DAOs. It is an entry point into the underlying database.

```kotlin
@Database(entities = [Joke::class], version = 1)
abstract class JokeDatabase : RoomDatabase() {
    abstract fun jokeDao(): JokeDao
}
```

{% hint style="info" %}
Database instance should be handled as a singleton. Analogically to http client - database is a heavy object which you don't need to recreate every time you run a query.
{% endhint %}

#### Data Entity

Data entity represents Object, which can be persisted in a database.

```kotlin
@Entity
data class Joke(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "icon_url") val iconUrl: String,
    @ColumnInfo(name = "value") val value: String,
)
```

#### Data Access Object

Instead of running queries on the database directly, you are highly recommended to create [`Dao`](https://developer.android.com/reference/kotlin/androidx/room/Dao) classes. Using Dao classes will allow you to abstract the database communication in a more logical layer which will be much easier to mock in tests (compared to running direct SQL queries). It also automatically does the conversion from `Cursor` to your application data classes so you don't need to deal with lower-level database APIs for most of your data access.

Room also verifies all of your queries in [`Dao`](https://developer.android.com/reference/kotlin/androidx/room/Dao) classes while the application is being compiled so that if there is a problem in one of the queries, you will be notified instantly.

```kotlin
@Dao
interface JokeDao {
    @Query("SELECT * FROM joke")
    suspend fun getAll(): List<Joke>

    @Query("SELECT * FROM joke WHERE id IN (:ids)")
    suspend fun loadAllByIds(ids: IntArray): List<Joke>

    @Insert
    suspend fun insertAll(vararg jokes: Joke)

    @Delete
    suspend fun delete(joke: Joke)
}
```

#### Observable Access 

Queries can be run as a one-shot selects to the database, but usually, you want to requery if the underlying data gets changed, and ideally, you don't want to communicate the changed state yourself, instead, you want to get notified about data changes by the database. And it is possible, by specifying the query, to be observable, for instance in a form of `LiveData`.

```kotlin
@Dao
interface JokeDao {
    @Query("SELECT * FROM joke")
    fun getAll(): LiveData<List<Joke>>

    @Query("SELECT * FROM joke WHERE id IN (:ids)")
    fun loadAllByIds(ids: IntArray): LiveData<List<Joke>>
    
    // ...
```

#### Persisting Composite Types

If you need to persist an object composed of other objects, you must specify how to do it. One possible way is to use [`TypeConverters`](https://developer.android.com/training/data-storage/room/referencing-data).&#x20;

```kotlin
class Converters {
  @TypeConverter
  fun fromTimestamp(value: Long?): Date? {
    return value?.let { Date(it) }
  }

  @TypeConverter
  fun dateToTimestamp(date: Date?): Long? {
    return date?.time?.toLong()
  }
}
```

```kotlin
@Database(entities = [User::class], version = 1)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
  abstract fun userDao(): UserDao
}
```

### Repository Pattern

Repository pattern implements separation of concerns by abstracting the data persistence logic in your applications.&#x20;

> [Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.](https://martinfowler.com/eaaCatalog/repository.html)

In other words, you access data through single interface (single source of truth), without the need to know, where it came from or how it appeared.

![](.gitbook/assets/repository.png)
