---
title: "A simple task management app using firebase - part 3"
date: 2017-01-04
categories: [Firebase, Android]
---

In the [previous post](https://vince-nyanga.github.io/task-management-app-with-firebase-2/) we added the Firebase realtime database to our app and linked it to our ListView using the Firebase UI library. In this post we are going to further improve our app by grouping related tasks. This is better that having a long list of unrelated tasks like we have before. Let's get started.

### Adding task list model

We are going to create a class that will hold a list of tasks. Create a class `TaskList.java` and edit it to look like the one below.

```java
public class TaskList {

    private String name;
    private String owner;

    public TaskList(){}

    public TaskList(String name, String owner) {
        this.name = name;
        this.owner = owner;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getOwner() {
        return owner;
    }

    public void setOwner(String owner) {
        this.owner = owner;
    }

    public Map<String, Object> toMap(){
        Map<String, Object> map = new HashMap<>();
        map.put("name",this.name);
        map.put("owner",this.owner);
        return map;
    }
}
```

We are going to store all task lists in their own node `taskLists` and tasks in their own node `tasks`. Each tasks list has a unique key assigned to it when we push it to firebase. All tasks that belong to a particular task list will be placed under the list's unique key in the `tasks` node. The example below will help clarify my explanation:

```json
"taskLists":{
  "1": {
    "name": "Groceries",
    "owner": "Vince"
  }
},
"tasks":{
  "1": {
    "t1": {
      "name": "Sugar",
      "done": false
    }
}
```

In the `tasks` node we have created a child node `1`, which is the unique key of the task list to which the tasks belong. In our app we will have two activities - one that shows a list of `TaskList`s and the other that show the `Task`s that belong to a given list. This is why I have decided to structure the database this way as this will make it easy to display and manage our data.

### Creating tasks lists activity

Create an activity called `TasksListsActivity.java` and edit it's layout file to look like this:

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="me.vincenyanga.taskmanager.TasksListsActivity">

    <ListView
        android:id="@+id/task_lists"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginBottom="@dimen/activity_vertical_margin"/>

    <TextView
        android:id="@+id/placeholder_txt"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        style="@style/TextAppearance.AppCompat.Headline"
        android:text="@string/no_lists"
        android:gravity="center"
        android:visibility="gone"/>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/add_btn"
        app:fabSize="normal"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_input_add"
        android:layout_gravity="bottom|right"
        android:tint="@android:color/white"
        android:layout_marginBottom="@dimen/activity_vertical_margin"/>
</FrameLayout>
```

Now create a new layout file called `list_item.xml` and edit it to look like this:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:id="@+id/name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="@style/TextAppearance.AppCompat.Title"/>
    <TextView
        android:id="@+id/owner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        style="@style/TextAppearance.AppCompat.Caption"/>

</LinearLayout>
```

### Fragment to add new task list

Create a new layout file and call it `new_task_list_layout.xml`:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"
              android:padding="@dimen/activity_vertical_margin">

    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="List name">

        <EditText
            android:id="@+id/task_list_name_et"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textCapSentences"/>
    </android.support.design.widget.TextInputLayout>

</LinearLayout>
```

Create a new class `AddTaskListFragment.java`. This is the fragment that we will use to create a new task list. Edit it to look like the one below:

```java
public class AddTaskListFragment extends DialogFragment {

    EditText name;

    public AddTaskListFragment(){}

    public static  AddTaskListFragment  newInstance(){
        AddTaskListFragment fragment = new AddTaskListFragment();
        return fragment;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        View dialogView = getActivity().getLayoutInflater().inflate(R.layout.new_task_list_layout,
                null);
        name = (EditText)dialogView.findViewById(R.id.task_list_name_et);
        AlertDialog.Builder builder =new AlertDialog.Builder(getActivity());
        builder.setTitle("Add task list");
        builder.setView(dialogView);
        builder.setNegativeButton("Cancel",null);
        builder.setPositiveButton("Add", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialogInterface, int i) {
                addTaskList();
            }
        });
        return builder.create();
    }

    private void addTaskList() {
        if(!TextUtils.isEmpty(name.getText().toString()) && getActivity() instanceof TasksListCallbackListener){
            TaskList taskList = new TaskList(name.getText().toString(),"Vince");
            ((TasksListCallbackListener)getActivity()).onTaskListAdded(taskList);
        }
    }

    public interface TasksListCallbackListener {
        void onTaskListAdded(TaskList taskList);
    }
}
```

### Refactor MainActivity and edit AndroidManifest

Rename `MainActivity.java` to `TasksActivity.java`. If you don't know how to do it open the project panel, right click on `MainActivity.java` and select _**Refactor**_ -> _**Rename**_. After refactoring let's go to the `AndroidManifest.xml` file and change our launcher activity. Edit the `AndroidManifest.xml` to look like this:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="<YOUR_PACKAGE_NAME>">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".TasksActivity">

        </activity>
        <activity android:name=".TasksListsActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### Edit TasksListsActivity

Open `TasksListsActivity.java` and edit it to look like this:

```java
public class TasksListsActivity extends AppCompatActivity implements AddTaskListFragment
        .TasksListCallbackListener {

    private ListView taskLists;
    private FloatingActionButton addButton;


    private DatabaseReference taskListsRef;
    private FirebaseListAdapter<TaskList> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tasks_lists);

        taskLists = (ListView) findViewById(R.id.task_lists);
        addButton = (FloatingActionButton) findViewById(R.id.add_btn);

        taskListsRef = FirebaseDatabase.getInstance().getReference().child("taskLists");
        adapter = new FirebaseListAdapter<TaskList>(TasksListsActivity.this, TaskList.class, R
                .layout
                .list_item, taskListsRef) {
            @Override
            protected void populateView(View view, TaskList taskList, int i) {
                TextView name = (TextView) view.findViewById(R.id.name);
                TextView owner = (TextView) view.findViewById(R.id.owner);

                name.setText(taskList.getName());
                owner.setText("Created by " + taskList.getOwner());
            }
        };

        taskLists.setAdapter(adapter);
        taskLists.setEmptyView(findViewById(R.id.placeholder_txt));

        taskLists.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long l) {
                TaskList taskList = adapter.getItem(position);
                String key = adapter.getRef(position).getKey();
                navigateToDetails(taskList, key);
            }
        });

        addButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                AddTaskListFragment.newInstance().show(getFragmentManager(),"new_list");
            }
        });

    }

    private void navigateToDetails(TaskList taskList, String key) {
        Intent intent = new Intent(TasksListsActivity.this, TasksActivity.class);
        intent.putExtra("name",taskList.getName());
        intent.putExtra("key",key);
        startActivity(intent);
    }

    @Override
    public void onTaskListAdded(final TaskList taskList) {
        final DatabaseReference newListRef = taskListsRef.push();
        newListRef.setValue(taskList.toMap()).addOnCompleteListener(new OnCompleteListener<Void>() {
            @Override
            public void onComplete(@NonNull Task<Void> task) {
                navigateToDetails(taskList,newListRef.getKey());
            }
        });
    }

    @Override
    protected void onStop() {
        super.onStop();
        adapter.cleanup();
    }
}
```

### Edit TasksActivity

Now let's go to `TasksActivity.java` and declare the following variables:

```java
 private String listName;
 private String listKey;
```

Add the following in the `onCreate()` method:

```java
        Bundle extras = getIntent().getExtras();
        if(extras !=  null){
            this.listKey = extras.getString("key");
            this.listName = extras.getString("name");
        }
        getSupportActionBar().setTitle(listName);
        tasksRef = FirebaseDatabase.getInstance().getReference().child("tasks").child(listKey);
```

Your `TasksActivity.java` should now look like this:

```java
public class TasksActivity extends AppCompatActivity implements AddTaskFragment
        .TaskCallbackListener {

    private ListView taskList;
    private FloatingActionButton addButton;

    private DatabaseReference tasksRef;
    private FirebaseListAdapter<Task> adapter;

    private String listName;
    private String listKey;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Bundle extras = getIntent().getExtras();
        if(extras !=  null){
            this.listKey = extras.getString("key");
            this.listName = extras.getString("name");
        }
        getSupportActionBar().setTitle(listName);
        tasksRef = FirebaseDatabase.getInstance().getReference().child("tasks").child(listKey);
        taskList = (ListView) findViewById(R.id.task_list);
        addButton = (FloatingActionButton) findViewById(R.id.add_btn);


        adapter = new FirebaseListAdapter<Task>(TasksActivity.this, Task.class, R.layout
                .task_list_item, tasksRef) {
            @Override
            protected void populateView(View view, Task task, int i) {
                TextView taskName = (TextView) view.findViewById(R.id.task_name);
                if (task.isDone()) {
                    // If the task is done (completed) we strike through it.
                    taskName.setPaintFlags(taskName.getPaintFlags() | Paint.STRIKE_THRU_TEXT_FLAG);
                } else {
                    taskName.setPaintFlags(taskName.getPaintFlags() & (~Paint
                            .STRIKE_THRU_TEXT_FLAG));
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
                AddTaskFragment.newInstance().show(getFragmentManager(), "new_task");
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

Run the app and see if it works as it should. You should be able to create a new list and add tasks into the list.

## Conclusion

In this post we have added a functionality to group related tasks into task lists. In the next post we will now add user authentication to the application. Once again, thanks for reading.
