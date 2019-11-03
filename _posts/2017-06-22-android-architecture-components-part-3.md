---
title: Android Architecture Components - Part 3
date: 2017-06-22
tags: [Android, Architecture]
---

Welcome to part 3 of our series on Android Architecture Components. In the [previous post](https://vince-nyanga.githubio/android-architecture-components-part-2/) we wrote tests for our database implementation to ensure that our database behaved in the manner we expect of it. In this post we are going to create a `ViewModel` and wire up the UI to add `Task`s to our database and list them.

## Let's get started
We will start by creating a repository class that will connect and interact with our database. As always get the complete code from [github](https://github.com/vince-nyanga/yo-tasks).
### Repository
Below is the code for the `Repository.java` class:
```java
public class DataRepository {

    @Inject
    YoTasksDb db;

    @Inject
    AppExecutors appExecutors;



    public DataRepository(YoTasksDb db, AppExecutors appExecutors){
        this.db = db;
        this.appExecutors = appExecutors;

    }


    public LiveData<List<Task>> getAllTasks(){
        return db.taskDao().loadTasks();
    }

    public void addTask(final Task task){
        appExecutors.databaseIO().execute(new Runnable() {
            @Override
            public void run() {
                db.taskDao().insert(task);
            }
        });

    }

    public void updateTask(final Task task){
        appExecutors.databaseIO().execute(new Runnable() {
            @Override
            public void run() {
                db.taskDao().update(task);
            }
        });

    }

    public void deleteTask(final Task task){
        appExecutors.databaseIO().execute(new Runnable() {
            @Override
            public void run() {
                db.taskDao().delete(task);
            }
        });

    }
    
}
```
### TaskListViewModel
Below is the code for the task list `ViewModel`. The `ViewModel` will connect to the database via the `Repository` class to get the list of tasks wrapped in a `LiveData` object and also to add new tasks to the database.
```java
public class TaskListViewModel extends ViewModel implements Injectable {

    @Inject
    public DataRepository repository;
    

    @Override
    public void inject(TasksComponent tasksComponent) {
        tasksComponent.inject(this);
    }

    public LiveData<List<Task>> getTasks(){
        return repository.getAllTasks();
    }

    public void addTask(String name){
        Task task = new Task();
        task.setDateCreated(new Date());
        task.setName(name);
        repository.addTask(task);
    }
}
```
Now that we are done with the `ViewModel` and `Repository` let's turn our focus to the user interface. The UI will certainly not win best design of the year award but it will help us see how all these components will work together - which is the purpose of this series. We are going to create a `Fragment` that will `Task`s using a `RecyclerView`. Let's first create the view that will hold each `Task` first. Some of the code has been omitted from this post for brevity.

### Task list item
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="task"
            type="com.mwiblo.yotasks.db.entity.Task"/>
        <variable
            name="callback"
            type="com.mwiblo.yotasks.ui.tasks.TaskClickListener"/>
    </data>

    <RelativeLayout
        android:paddingStart="16dp"
        android:paddingLeft="16dp"
        android:paddingEnd="16dp"
        android:paddingRight="16dp"
        android:background="?android:attr/selectableItemBackground"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:minHeight="64dp"
        android:clickable="true"
        android:focusable="true"
        android:onClick="@{() -> callback.onClick(task)}">

        <TextView
            android:id="@+id/date"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:date="@{task.dateCreated}"
            style="@style/TextAppearance.AppCompat.Caption"
            android:layout_alignParentRight="true"
            android:layout_alignParentEnd="true"
            android:layout_marginTop="4dp"/>

        <TextView
            android:layout_below="@+id/date"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:text="@{task.name}"
            style="@style/TextAppearance.AppCompat.Title"/>
        <View
            android:layout_alignParentBottom="true"
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#cccccc"/>
    </RelativeLayout>
</layout>
```
Let's also create the `RecyclerViewAdapter`
### Adapter
```java
public class TasksAdapter extends RecyclerView.Adapter<TasksAdapter.TaskHolder> {

    private List<Task> list;

    @Nullable
    private TaskClickListener clickListener;

    public TasksAdapter(TaskClickListener clickListener){
        this.clickListener = clickListener;
    }

    @Override
    public TaskHolder onCreateViewHolder(ViewGroup container, int i) {
        TaskItemBinding binding = DataBindingUtil.inflate(LayoutInflater.from(container
                .getContext()), R.layout.task_item,container, false);
        binding.setCallback(clickListener);

        return new TaskHolder(binding);
    }

    @Override
    public void onBindViewHolder(TaskHolder taskHolder, int position) {
        taskHolder.binding.setTask(list.get(position));
        taskHolder.binding.executePendingBindings();
    }

    @Override
    public int getItemCount() {
        return list ==null ? 0 : list.size();
    }

    public void setTaskList(final List<Task> newList){
        if(this.list == null){
            this.list = newList;
        }else{
            DiffUtil.DiffResult result = DiffUtil.calculateDiff(new DiffUtil.Callback() {
                @Override
                public int getOldListSize() {
                    return list.size();
                }

                @Override
                public int getNewListSize() {
                    return list.size();
                }

                @Override
                public boolean areItemsTheSame(int oldPosition, int newPosition) {
                     return list.get(oldPosition).getId() == newList.get(newPosition).getId();
                }

                @Override
                public boolean areContentsTheSame(int oldPosition, int newPosition) {
                    return list.get(oldPosition).getId() == newList.get(newPosition).getId();
                }
            });
            list = newList;
            result.dispatchUpdatesTo(this);
        }
    }


    static class TaskHolder extends RecyclerView.ViewHolder{

        final TaskItemBinding binding;

        public TaskHolder(TaskItemBinding binding){
            super(binding.getRoot());
            this.binding = binding;


        }
    }
}
```
Then the `Fragment`:
### Task List Fragment
```java
public class TaskListFragment extends LifecycleFragment {

    private TasksAdapter tasksAdapter;

    private ListFragmentBinding binding;

    private TaskClickListener listener = new TaskClickListener() {
        @Override
        public void onClick(Task task) {
            // non-imp
        }
    };

    private AddTaskCallback callback = new AddTaskCallback() {
        @Override
        public void onClick() {
            // Not a very good idea...
            ((MainActivity)getActivity()).showAddTaskView();
        }
    };

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        binding = DataBindingUtil.inflate(inflater, R.layout.list_fragment, container, false);
        tasksAdapter = new TasksAdapter(listener);
        binding.tasksList.setAdapter(tasksAdapter);
        binding.setIsListEmpty(tasksAdapter.getItemCount() == 0);
        binding.setCallback(callback);
        return binding.getRoot();
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final TaskListViewModel viewModel = ViewModelProviders.of(this, new ViewModelFactory(
                (YoTasksApp)getActivity().getApplication())).get(TaskListViewModel.class);
        viewModel.getTasks().observe(this, new Observer<List<Task>>() {
            @Override
            public void onChanged(@Nullable List<Task> tasks) {
                tasksAdapter.setTaskList(tasks);
                binding.setIsListEmpty(tasks != null ? tasks.isEmpty(): true);
            }
        });
    }
}
```
In our `Fragment` we create our `ViewModel` and hook it up to the lifecycle of the `Fragment` in this line of code:
```java
TaskListViewModel viewModel = ViewModelProviders.of(this, new ViewModelFactory(
                (YoTasksApp)getActivity().getApplication())).get(TaskListViewModel.class);
```
Note that the class extends `LifecycleFragment`. `LifecycleFragment` is a `Fragment` that is also a `LifecycleOwner`. This class is a temporary implementation until Lifecycles are integrated in the support library. If you don't want your `Fragment` to extend `LifecycleFragment` you should implement `LifecycleRegistryOwner` like this:

```java
public class MyFragment extends Fragment implements LifecycleRegistryOwner {
    
    final LifecycleRegistry registry = new LifecycleRegistry(this);

    @Override
    public LifecycleRegistry getLifecycle() {
        return  registry;
    }
}
```
Now we can subscribe to the Task list wrapped around a `LiveData` object:
```java
viewModel.getTasks().observe(this, new Observer<List<Task>>() {
            @Override
            public void onChanged(@Nullable List<Task> tasks) {
                tasksAdapter.setTaskList(tasks);
                binding.setIsListEmpty(tasks != null ? tasks.isEmpty(): true);
            }
        });
```
Since the Room library works with LiveData seamlessly, when there is any change in the data in our database the UI will get notified and update accordingly.

## Summary
In this post we linked our data to the UI using a `ViewModel` class. This is probably the last part in this series unless if something comes up that I will need to talk about. I will however finish the app on github so you may need to keep an eye on it if you are interested in seeing how the app will be put together. The main objective of this series was to introduce the super cool Android Architecture Components. I have learnt a lot about these components (though I don't know it all yet) during this series and I hope you'll be encouraged to look into them further. 
