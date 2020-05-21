---
title: Android Architecture Components - Part 1
date: 2017-06-13
categories: [Android, Architecture]
---

At Google I/O 2017 Android Architecture Components were introduced. These components are building blocks that work together to implement an app architecture that is easy to maintain and avoids numerous pain points. They help developers:

- Manage activity and fragment lifecycles to avoid memory leaks, which are a real pain.
- Persist POJOs to a SQLite database without much of a hussle.

In this post I'm going to talk about a few of those components which are LiveData, Room and ViewModel. You can check out the rest of the components [here](https://developer.android.com/topic/libraries/architecture/index.html).

## LiveData

LiveData is an observable data holder class that is aware of the lifecycle. When the UI susbcribes to the changes in the data held by a LiveData class, it (LiveData) ensures that :

- The observer gets data updates if the lifecycle is in active state. This helps avoid trying to update the UI when it's not visible.
- The observer will be removed when the LifecycleOwner (Activity/Fragment) is destroyed to avoid memory leaks
- The observer will get the latest data when the Lifecycle owner is restarted.

LiveData observers need to be tied to a lifecycle owner like Activity or Fragment. With LiveData there won't be a need to implement all those cumbersome lifecycle methods like `onResume()`, `onPause()`, `onDestroy()` etc.

## ViewModel

ViewModel is a class that contains UI data for an Activity or a Fragment. It's purpose is to separate business logic from the UI. The cool thing about the ViewModel class is that it is retained for as long as the the scope of its Activity or Fragment is alive. It actually survives configuration changes so when the Activity is recreated due to configuration changes the data does not get lost. Wrapping UI data stored in a ViewModel with LiveData will give you an observable lifecycle-aware data that survives configuration changes. Cool!!!

## Room

The Room library contains components that provide an object-mapping layer that allows smooth access to the database. There are three major compnents in the Room library:

- **Entity**: An Entity represents the data for a single database row that is constructed using an annotated POJO. Each component persists in its own table.
- **DAO (Data Access Object)**: This defines methods that access the database using annotaions to bind SQL to each method.
- **Database**: This is a holder class that makes use of annotations to define the list of entities and the database version. It contains the list of DAOs and also acts as the main access point to the underlying database connection.

# Let's get started

In this series of posts we will create a simple task management app (I love task management) that makes use of the components discussed above. The app will have the following entities:

- **Task**: An summary of related stuff that the user needs to do.
- **TaskItem**: An individual item that belongs to a particular Task

Check out the complete code on [github](https://github.com/vince-nyanga/yo-tasks) in order for you to see the gradle dependencies that I used as well as other classes that I won't discuss about in this post. We will start by defining our entities:

### Task

The following code is for the Task entity:

```java
@Entity(tableName = "tasks")
public class Task {
    @PrimaryKey(autoGenerate = true)
    private int id;
    private String name;
    private Date dateCreated;


    public Task() {

    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getDateCreated() {
        return dateCreated;
    }

    public void setDateCreated(Date dateCreated) {
        this.dateCreated = dateCreated;
    }

}
```

As you can see we have an `@Entity` annotation that takes in the `tableName`. The table name parameter is the name of the table in the SQLite database. The `id` field is the primary key in our table.

### TaskItem

Here is the code for the TaskItem:

```java
@Entity(tableName = "items",
foreignKeys = {
        @ForeignKey(entity = Task.class,
        parentColumns = "id",
        childColumns = "taskId",
        onDelete = CASCADE)},
indices = {
        @Index(value = "taskId")
})
public class TaskItem {

    @PrimaryKey(autoGenerate = true)
    private int id;
    private int taskId;
    private String name;
    private boolean done;


    public TaskItem(){}

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getTaskId() {
        return taskId;
    }


    public void setTaskId(int taskId) {
        this.taskId = taskId;
    }


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }


    public boolean isDone() {
        return done;
    }

    public void setDone(boolean done) {
        this.done = done;
    }
}
```

We have defined the foreign key `taskId` as well as an index for the TaskItem entity. Other than that everything else is the same as the Task entity above.

Now let us create the DAOs for Task and TaskItem entities respectively:

### TaskDao

```java
@Dao
public interface TaskDao {

    @Query("SELECT * FROM tasks")
    LiveData<List<Task>> loadTasks();

    @Query("SELECT * FROM tasks WHERE id = :id")
    LiveData<Task> getTaskById(int id);

    @Insert
    void insert (Task task);

    @Update
    void update(Task task);

    @Delete
    void delete(Task task);
}
```

This interface defines the methods we will use to access the database and perform respective operations. The operations are self-explanatory. Notice the SQL query statements above `loadTasks()` and `getTaskById()`. The queries return LiveData objects in order for user to harness the power of LiveData explained above.

### TaskItemDao

```java
@Dao
public interface TaskItemDao {

    @Query("SELECT * FROM items WHERE taskId = :taskId")
    LiveData<List<TaskItem>> loadItems(int taskId);

    @Query("SELECT * from items WHERE id = :id")
    LiveData<TaskItem> getItemById(int id);

    @Insert
    void insert(TaskItem item);

    @Update
    void update(TaskItem item);

    @Delete
    void delete(TaskItem item);
}
```

### Type Converters

As you know there are objects that cannot be stored in a SQLite database as-is. Our entities contain two such objects - `Date` and `boolean`. In order for us to be able to store these types in our database we need to convert them into types that the SQLite database will manage to store. When they get back from the database we want them to be converted back into the types that are defined in our entities. Below is the code for `DateConverter` that converts a `java.util.Date` object into a `Long` as well as a `BooleanConverter` that converts a `boolean` into an integer (1 for `true` and 0 for `false`)

#### DateConverter

```java
public class DateConverter {

    @TypeConverter
    public  static Date toDate(Long timestamp){
        return timestamp == null ? null : new Date(timestamp);
    }

    @TypeConverter
    public static Long toTimestamp(Date date){
        return date == null? null : date.getTime();
    }
}
```

#### BooleanConverter

```java
public class BooleanConverter {

    @TypeConverter
    public static int toInteger(boolean bool){
        return bool? 1 : 0;
    }

    @TypeConverter
    public static boolean toBoolean(int i){
        return i == 1;
    }
}
```

Now let's create our database class. The database class as mentioned above lists all the entities, the type converters as well as DAOs.

### Database

```java
@Database(entities = {Task.class, TaskItem.class}, version = 1)
@TypeConverters({DateConverter.class, BooleanConverter.class})
public abstract class YoTasksDb extends RoomDatabase {
    public static final String DATABASE_NAME = "yo-tasks-db";

    public abstract TaskDao taskDao();

    public abstract TaskItemDao taskItemDao();

}
```

The database is created using the following code:

```java
YoTasksDb db = Room.databaseBuilder(application, YoTasksDb.class, YoTasksDb.DATABASE_NAME).build();
```

See full implementation on [github](https://github.com/vince-nyanga/yo-tasks).

# Summary

In this post we spoke about Android Architecture Components (some of them) and discussed how they can help build an app that is free of pain. We used the Room library to create Entities, DAOs and the Database class for our task management app. In the next post we will continue to build on this foundation till we have a fully functional app. Thank you so much for taking your time to read this post.
