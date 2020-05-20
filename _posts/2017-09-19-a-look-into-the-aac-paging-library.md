---
title: A look into the Android Architecture Components Paging Library
date: 2017-09-19
tags: [Android, Architecute, Kotlin]
---

Imagine you have data set that contains 10 000 items that you need to display in your application. Loading all these items before rendering them on the screen may take time affecting both performance and user experience. Thanks to the **Paging Library** this problem can now be avoided.

### What does the Paging library do?

The paging library enables the application to gradually load information from a data source without straining the device or waiting for a long time for all the data to be loaded. Information is load in small portions (pages) at a time.

### What does it consist of?

The library consists of a couple of the following classes among others:

#### DataSource

This class is used to define a data source you need to pull paged data from. I has the following subclasses one of which you need to extend depending on how you need to access your data:

- **KeyedDataSource** is used if you need to use data from item `N` to fetch item `N+1`.
- **TiledDataSource** is used if you need to fetch pages of data from any location in your data store, for instance getting 50 items starting from location 20.

The Room persistence library can create the DataSource class automatically as shown in the example below.

##### PagedList

This is the class that loads from a DataSource and you can configure how much data is loaded at a time and how much should be prefetched:

```kotlin
val allUsers = userDao.getUsers()
            .create(0,PagedList.Config.Builder()
                    .setPageSize(PAGE_SIZE)
                    .setPrefetchDistance(PREFETCH_DISTANCE)
                    .setEnablePlaceholders(ENABLE_PLACEHOLDERS).build())
```

##### PagedListAdapter

This class implements the `RecyclerView.Adapter` and is used to present data from a PagedList . When a new page of data is loaded, it signals the `RecyclerView` that new data has arrived and the RecyclerView deals with the data accordingly. It uses a background thread to compute changes from one PagedList to the next using `DiffUtil`.

##### LivePagedListProvider

This class generates `LiveData<PagedList>` from the DataSource . Again, the Room persistence library can generate it for you:

```kotlin
    @Query("SELECT * FROM users ORDER BY name ASC")
    fun getUsers(): LivePagedListProvider<Int, User>
```

### Example

Let’s create a simple app that displays a list of users using the Paging library. Check out the project on [github](https://github.com/vince-nyanga/paging-library-example) if you want to see the whole project.

#### Add dependencies

Add the following dependencies to your app `build.gradle` file:

```gradle
implementation “com.android.support:appcompat-v7:26.1.0”
implementation “android.arch.persistence.room:runtime:1.0.0-alpha9–1”
implementation “android.arch.lifecycle:runtime:1.0.0-alpha9–1”
implementation “android.arch.lifecycle:extensions:1.0.0-alpha9–1”
implementation “android.arch.paging:runtime:$paging_version”
kapt “android.arch.persistence.room:compiler:1.0.0-alpha9–1”
```

#### Create the Entity

Let’s create our `User` entity:

```kotlin
@Entity(tableName = "users")
data class User(
        @PrimaryKey(autoGenerate = true)val id: Int,
        val name: String,
        val surname: String="Nyanga"){

    companion object {
        val DIFF_CALLBACK = object: DiffCallback<User>(){
            override fun areContentsTheSame(oldItem: User, newItem: User): Boolean = oldItem == newItem

            override fun areItemsTheSame(oldItem: User, newItem: User): Boolean =  oldItem == newItem

        }
    }
}
```

#### UserDao

Let’s a data access object for our user entity:

```kotlin
@Dao
interface UserDao {

    @Query("SELECT * FROM users ORDER BY name ASC")
    fun getUsers(): LivePagedListProvider<Int, User>

    @Insert
    fun insert(users: List<User>)

    @Insert
    fun insert(user: User)

}
```

As you can see our `getUsers()` function returns LivePagedListProvider. This class generates a `LiveData<PagedList>` as explained above.

#### Create the User adapter

Let’s create a `PagedListAdapter` that we will use to load our PagedList into the `RecyclerView`. Note the `DiffCallback`that will be used to compute changes.

```kotlin
class UserAdapter: PagedListAdapter<User, UserAdapter.UserViewHolder>(User.DIFF_CALLBACK){

    override fun onBindViewHolder(holder: UserViewHolder?, position: Int) {
        holder?.bindTo(getItem(position))
    }

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): UserViewHolder =
            UserViewHolder(LayoutInflater.from(parent?.context).inflate(R.layout.user_item,parent,
            false))


    class UserViewHolder(view: View): RecyclerView.ViewHolder(view){

        private val nameTextView = itemView.findViewById<TextView>(R.id.name)

        fun bindTo(user: User?){
            nameTextView.text = ("${user?.name} ${user?.surname}")
        }
    }

}
```

#### Create the ViewModel

```kotlin
class UserViewModel(app: Application): AndroidViewModel(app) {

    val userDao = UsersDb.get(app).userDao()
    val allUsers = userDao.getUsers()
            .create(0,PagedList.Config.Builder()
                    .setPageSize(PAGE_SIZE)
                    .setPrefetchDistance(PREFETCH_DISTANCE)
                    .setEnablePlaceholders(ENABLE_PLACEHOLDERS).build())

    companion object {
        private const val PAGE_SIZE = 15
        private const val PREFETCH_DISTANCE = 5
        private const val ENABLE_PLACEHOLDERS = true
    }
}
```

In our `MainActivity` we will subscribe to the user list:

```kotlin
val adapter = UserAdapter()
        val userList = findViewById<RecyclerView>(R.id.userList)
        userList.adapter = adapter
        viewModel.allUsers.observe(this, Observer{list ->
            adapter.setList(list)
        } )
```

#### Page Size

When creating a `PagedList` you need to set the page size. What size should you use? After playing around the page size in the sample app I found out that if the page size does not fill at least a screen worth of content, say 2, for a brief moment I would see the first two items followed by nulls. So the page size should be a value that fills at least screen worth of content on a large device to avoid the user seeing null items.

#### Placeholders

If you enable placeholders in your `PagedList` it will report the full size of the list with some items as nulls depending on your page size.

### Conclusion

In this post I took a look at the Paging Library, the classes that make up the library and a sample app that uses the library. Once again thanks for taking your time to read.
