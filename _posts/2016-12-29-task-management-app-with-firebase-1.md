---
title: "A simple task management app using firebase - part 1"
date: 2016-12-29
tags: [Firebase, Android]
---

In this series we are going to create a simple android app for managing tasks using firebase as our backend service. The app should be able to do the following:

- Create tasks
- Edit tasks (mark as complete or edit task details)
- Authenticate users
- Share tasks

## What is Firebase?

According to the [firebase website](https://firebase.google.com/ "Firebase"), Firebase is a mobile platform that helps developers quickly develop high-quality apps using a wide range of features. The features include realtime database, authentication, storage, analytics among others. You can visit the firebase website to see all the features. We will use some of these features in the task management app.

## Ok, let's get started

Before we begin I assume that you have Android Studio (2.2 at the time of writing) and the Android sdk installed on your machine.I also assume that you have basic knowledge of Android development.

#### 1. Create project.

Create a new android project in Android Studio and name it Task Manager. When the project is created open the app gradle file and add the design support library in your dependencies: `compile 'com.android.support:design:25.1.0'`.

#### 2. Create the layout

Open _**activitymain.xml**_, remove the available code and add the following code:

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
    tools:context="me.vincenyanga.taskmanager.MainActivity">

    <ListView
        android:id="@+id/task_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginBottom="@dimen/activity_vertical_margin"/>
    <!--Placeholder text that will appear when the list is empty-->
    <TextView
        android:id="@+id/placeholder_txt"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        style="@style/TextAppearance.AppCompat.Headline"
        android:text="@string/no_tasks"
        android:gravity="center"/>

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

Create a new layout named _**tasklistitem.xml**_ and add the following code:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="wrap_content">
    <TextView
        android:id="@+id/task_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="@style/TextAppearance.AppCompat.Title"
        android:layout_marginBottom="8dp"/>
</LinearLayout>
```

#### 3. Create model

Create a new Java class and name it _**Task.java**_ then add the following code:

```java
public class Task {

    private String name;
    private boolean done;

    public Task(){}

    public Task(String name) {
        this.name = name;
        this.done = false;
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

#### 4. Create list adapter

We are now going to create an adapter that will be repsonsible for loading the tasks from a list and displaying them into the ListView. Create a new class named _**TasksAdapter.java**_ and add the following code:

```java
public class TasksAdapter extends ArrayAdapter<Task> {

    private List<Task> tasks;
    private Context context;

    public TasksAdapter(Context context, @NonNull List<Task> tasks) {
        super(context, -1, tasks);
        this.context = context;
        this.tasks = tasks;
    }

    @NonNull
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        if (convertView == null) {
            convertView = inflater.inflate(R.layout.task_list_item, parent, false);
        }

        TextView taskName = (TextView) convertView.findViewById(R.id.task_name);
        Task task = tasks.get(position);
        if (task.isDone()) {
            // If the task is done (completed) we strike through it.
            taskName.setPaintFlags(taskName.getPaintFlags() | Paint.STRIKE_THRU_TEXT_FLAG);
        } else {
            taskName.setPaintFlags(taskName.getPaintFlags() & (~Paint.STRIKE_THRU_TEXT_FLAG));
        }
        taskName.setText(task.getName());

        return convertView;
    }
}
```

#### 4. Wiring everything up

Now let's wire up our app and make sure that the we have the basic functionality working. Open _**MainActivity.java**_ and add the following code:

```java
public class MainActivity extends AppCompatActivity {

    private ListView taskList;
    private FloatingActionButton addButton;
    private TasksAdapter adapter;
    private List<Task> tasks;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        taskList = (ListView) findViewById(R.id.task_list);
        addButton = (FloatingActionButton) findViewById(R.id.add_btn);

        tasks = new ArrayList<>();
        tasks.add(new Task("Finish blog"));
        tasks.add(new Task("Call Baby"));

        adapter = new TasksAdapter(MainActivity.this, tasks);
        taskList.setAdapter(adapter);
        taskList.setEmptyView(findViewById(R.id.placeholder_txt));

        taskList.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int position, long l) {
                Task task = tasks.get(position);
                task.setDone(!task.isDone());
                adapter.notifyDataSetChanged();

            }
        });

    }
}
```

#### 5. Adding tasks

Now that our app's basic functionality is working well let's add the functionality to add a new task. We will create a new screen from which a user will add a new task. First create a new layout called _**newtasklayout.xml**_ and add the following code:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical"
              android:padding="@dimen/activity_vertical_margin">

    <android.support.design.widget.TextInputLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Task name">
        <EditText
            android:id="@+id/task_name_et"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textCapSentences"/>
    </android.support.design.widget.TextInputLayout>

</LinearLayout>
```

Next, create a new class name _**AddTaskFragment.java**_ and add the following:

```java
public class AddTaskFragment extends DialogFragment {

    private EditText taskName;

    public  AddTaskFragment(){}

    public static AddTaskFragment newInstance(){
        AddTaskFragment fragment =new AddTaskFragment();
        return fragment;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        View dialogView = getActivity().getLayoutInflater().inflate(R.layout.new_task_layout,null);
        taskName = (EditText)dialogView.findViewById(R.id.task_name_et);
        AlertDialog.Builder builder =new AlertDialog.Builder(getActivity());
        builder.setTitle("Add task");
        builder.setView(dialogView);
        builder.setNegativeButton("Cancel",null);
        builder.setPositiveButton("Add", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialogInterface, int i) {
                addTask();
            }
        });
        return builder.create();
    }

    private void addTask() {
        String name = taskName.getText().toString();
        if(!TextUtils.isEmpty(name) && getActivity() instanceof  TaskCallbackListener){
            Task task =new Task(name);
            ((TaskCallbackListener)getActivity()).onTaskAdded(task);
        }
    }

    public interface TaskCallbackListener{
        void onTaskAdded(Task task);
    }
}
```

Now, we need to implement `TaskCallbackListener` in _**MainActivity.java**_ . It should look like this:

```java
public class MainActivity extends AppCompatActivity implements AddTaskFragment.TaskCallbackListener {

   //[This code is already there]

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      //[This code is already there]
        addButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                AddTaskFragment.newInstance().show(getFragmentManager(),"new_task");
            }
        });
    }

    @Override
    public void onTaskAdded(Task task) {
        tasks.add(task);
        adapter.notifyDataSetChanged();
    }
}
```

Now run the app and click in the FAB to add new task.

## Conclusion

In this part of the series we created a basic app that can add new tasks and make them as done. However, as might have noticed, the tasks are not being persisted anywhere so when you close the app everything is lost. In the next post we will now start to use firebase to persist our tasks to the realtime database.
