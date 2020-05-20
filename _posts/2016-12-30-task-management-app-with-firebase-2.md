---
title: "A simple task management app using firebase - part 2"
date: 2016-12-30
categories: [Firebase, Android]
---

In the [previous post](https://vince-nyanga.github.io/task-management-app-with-firebase-1/) we created a simple task management application that had the basic functionalities of adding tasks and marking them as completed. However, the tasks were not being persisted anywhere so as soon as the app is closed everything would disappear. In this post we are going to fix that problem. We are going to do the following:

- Connect to Firebase
- Read from and write to the realtime database
- Use Firebase UI library to connnect the app's UI to the realtime database

## Realtime database

The Firebase realtime database is a cloud-hosted NoSQL database that stores data as JSON. Data is synced to all connected devices in a matter of milliseconds. It also has offline support. If you want to know more about the realtime database visit the Firebase website.

### Connecting to Firebase

With your Task Manager project open in Android Studio, go to _**Tools -> Firebase**_. When the Firebase panel is open click on _**Realtime database -> Save and retrieve data**_ then click _**Connect to database**_. You might be prompted to login with your google account. Once you are logged in, a dialog will appear from which you will either create a new Firebase project or connect to an existing one. In our case we will create a new project so make sure the _**Create a new Firebase project**_ radio button is check. Next give your project a name if you don't want it to have the same name as your application. When you are done click _**Connect to Firebase**_. Once you are connected to Firebase select _**Add the realtime database to your app**_. Android Studio will add all the required dependencies in your `build.gradle` files and download a `google-services.json` file containing all the data required by google play services. Now that we are connected to Firebase and have added the realtime database let's get started.

### Database rules

The realtime database comes with a set of rules by default to make sure that user data is secure. To see these rules go to the [firebase console](https://console.firebase.google.com/) and open your Task Manager project. Select database on the menu and go to the rules tab. Your see a json object that looks like this:

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

What the rules above simply mean is that only authenticated users are allowed to read from and write to the database. We will discuss database rules in detail in later posts. For now let's loosen the rules a bit so that we can be able to access the database without the need for authentication, a functionality that will be added in the next post. Edit your rules to look like this and click _**publish**_:

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

This is not a recommended thing to do so you will get a warning from firebase. Let's ignore the warning for now and proceed.

### Firebase UI

Firebase UI is a library that allows us to quickly connect common android UI elements to Firebase APIs like the realtime database, authentication and storage. We are going to use this library to connect our tasks ListView to the realtime database. Add `compile 'com.firebaseui:firebase-ui-database:1.0.1'` to your `app/build.gradle` file and sync. We are now all set to dive into code.

### Edit Task model

Open `Task.java` and add the following method:

```java
     **
     * The Firebase realtime database only supports the following :
     *  String
     *  Long
     *  Double
     *  Boolean
     *  Map<String, Object>
     *  List<Object>
     *
     *  So we convert the Task object into a map before we save to database
     * */
    public Map<String, Object> toMap(){
        Map<String, Object> map = new HashMap<>();
        map.put("name",this.name);
        map.put("done", this.done);
        return map;
    }
```

### Edit MainActivity

Open `MainActivity.java` and make the following changes and remove the following lines of code and all the other parts they affect:

```java
private TasksAdapter adpter;
private List<Task> tasks;
```

Once you have cleaned up all errors that appear after the above change, add the following:

```java
  private DatabaseReference tasksRef;
  private FirebaseListAdapter<Task> adapter;
```

In the `onCreate()` method let's initialise the `taskRef` object:

```java
tasksRef = FirebaseDatabase.getInstance().getReference().child("tasks");
```

The code above connects to the Firebase realtime database at a child node named _**tasks**_. As mentioned above, the realtime database stores data as JSON so we will create a node in which we will store all our tasks. In case we want to store user data we will then create another node `users`. This approach makes it easy to manage our data.

While you are still inside the `onCreate()` method and the following lines of code:

```java
 adapter = new FirebaseListAdapter<Task>(MainActivity.this, Task.class, R.layout.task_list_item, tasksRef) {
            @Override
            protected void populateView(View view, Task task, int i) {
                TextView taskName = (TextView) view.findViewById(R.id.task_name);
                if (task.isDone()) {
                    // If the task is done (completed) we strike through it.
                    taskName.setPaintFlags(taskName.getPaintFlags() | Paint.STRIKE_THRU_TEXT_FLAG);
                } else {
                    taskName.setPaintFlags(taskName.getPaintFlags() & (~Paint.STRIKE_THRU_TEXT_FLAG));
                }
                taskName.setText(task.getName());
            }
        };
taskList.setAdapter(adapter);
```

In the code above we are initialising the `FirebaseListAdapter` provided in the Firebase UI library. This performs the same task as the `TasksAdapter` in part 1 except that instead of taking a list of tasks, it connects to a Firebase database reference. It takes all the tasks from the realtime database and loads them into our ListView and is also responsible for ensuring that the ListView is always in sync with the realtime database.

Now let's update the `onItemClickListener` for our ListView. Add the following code:

```java
  taskList.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long l) {
                Task task = adapter.getItem(position);
                task.setDone(!task.isDone());
                tasksRef.child(adapter.getRef(position).getKey()).updateChildren(task.toMap());
            }
        });
```

The code above changes the `done` attribute of the clicked task and updates the realtime database. Now add the following line of code in the `onTaskAdded()` method to add the new task to the realtime database:

```java
tasksRef.push().setValue(task.toMap());
```

Lastly let's do some cleanup:

```java
@Override
    protected void onDestroy() {
        super.onDestroy();
        adapter.cleanup();
    }
```

Now your `MainActivity.java` should be looking like this:

```java
public class MainActivity extends AppCompatActivity implements AddTaskFragment.TaskCallbackListener {

    private ListView taskList;
    private FloatingActionButton addButton;


    private DatabaseReference tasksRef;
    private FirebaseListAdapter<Task> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        taskList = (ListView) findViewById(R.id.task_list);
        addButton = (FloatingActionButton) findViewById(R.id.add_btn);

        tasksRef = FirebaseDatabase.getInstance().getReference().child("tasks");
        adapter = new FirebaseListAdapter<Task>(MainActivity.this, Task.class, R.layout.task_list_item, tasksRef) {
            @Override
            protected void populateView(View view, Task task, int i) {
                TextView taskName = (TextView) view.findViewById(R.id.task_name);
                if (task.isDone()) {
                    // If the task is done (completed) we strike through it.
                    taskName.setPaintFlags(taskName.getPaintFlags() | Paint.STRIKE_THRU_TEXT_FLAG);
                } else {
                    taskName.setPaintFlags(taskName.getPaintFlags() & (~Paint.STRIKE_THRU_TEXT_FLAG));
                }
                taskName.setText(task.getName());
            }
        };
        taskList.setAdapter(adapter);
        taskList.setEmptyView(findViewById(R.id.placeholder_txt));

        taskList.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long l) {
                Task task = adapter.getItem(position);
                task.setDone(!task.isDone());
                tasksRef.child(adapter.getRef(position).getKey()).updateChildren(task.toMap());
            }
        });

        addButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                AddTaskFragment.newInstance().show(getFragmentManager(),"new_task");
            }
        });
    }

    @Override
    public void onTaskAdded(Task task) {
        tasksRef.push().setValue(task.toMap());
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        adapter.cleanup();
    }
}
```

Run the app and add a new task. If you see the task apearing in your ListView then it means you've done everything right. Now go to your project in the Firebase console and open the database section. You should see the task you have added. If you close your app and open again you will find that the task you added didn't disappear - it's now saved in firebase.

## Conclusion

In this part we connected our app to Firebase and added the realtime database. We also used the Firebase UI library to easily connect our ListView to the realtime database. We successfully added and retrieved tasks to the realtime database. Cool!!! In the next post we are going to add more functionalities to the app. Thanks for reading :)
