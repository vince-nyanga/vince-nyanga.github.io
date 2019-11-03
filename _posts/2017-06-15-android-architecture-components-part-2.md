---
title: Android Architecture Components - Part 2
date: 2017-06-15
tags: [Android, Architecture]
---

Welcome to part 2 of our series on Android Architecture Components. In the [previous post](https://vince-nyanga.github.io/android-architecture-components-part-1/) we introduced the components and created Entities, DAOs and Database class for our task management app. In this post, which is relatively short, we will write tests for our database to make sure everything works as expected before we proceed.

# Testing database
When running tests for the database there is no need to create a full database itself. The Room library allows us to mock the data access layer in our tests. This is possible due to the fact that Data Access Objects (DAOs) don't leak details of the database. There are two ways to test our database, which are:

- **Test on your host development machine**: Room library uses the SQLite Support Library that provides interfaces that match those in the Android Framework classes. This allows you to pass custom implementations of the support library to test your queries and also helps your tests run very quickly. However, the disadvantage of this approach is that the version of SQLite running on the users' devices might not match the version on your host machine leading to unexpected results.
- **Test on an Android device**: This is the recommended approach to testing your database implementation. The best way to do it is to write JUnit tests that run on a device. In our tests we will create an in-memory version of our database.

## Let's get started
Like I said earlier, this post is gonna be very short. We will just write tests for the `TaskDao.java` and `TaskItemDao.java` classes. Check the complete code on [github](https://github.com/vince-nyanga/yo-tasks) since in this post I will just show the code for the tests for brevity. I have added some methods to the `TaskDao` and `TaskItemDao` to help make testing easier.

### TaskDao Test
Below is the code for the test for `TaskDao.java`:

```java
@RunWith(AndroidJUnit4.class)
public class TaskDaoTest {

    private static final String TASK_NAME =  "Groceries";
    private static final String NEW_TASK_NAME = "Shopping";
    private static final int TASK_ID = 1;

    private TaskDao taskDao;
    private YoTasksDb db;

    @Before
    public void setUp(){
        Context context = InstrumentationRegistry.getTargetContext();
        db = Room.inMemoryDatabaseBuilder(context, YoTasksDb.class).build();
        taskDao = db.taskDao();
    }

    @After
    public void tearDown(){
        db.close();
    }

    @Test
    public void insert() throws Exception{
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);
        List<Task> taskList = taskDao.loadTasksSync();
        assertEquals("Task list should be of size = 1", 1,taskList.size());
        assertEquals("Task name should be "+TASK_NAME,TASK_NAME, taskList.get(0).getName());
        assertEquals("Task id should be " + TASK_ID, TASK_ID, taskList.get(0).getId());
    }

    @Test
    public void getTaskByIdSync() throws Exception{
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);
        task = taskDao.getTaskByIdSync(TASK_ID);
        assertNotNull(task);
    }

    @Test
    public void update() throws Exception{
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);
        task = taskDao.getTaskByIdSync(TASK_ID);
        assertNotNull(task);
        task.setName(NEW_TASK_NAME);
        taskDao.update(task);

        Task updatedTask = taskDao.getTaskByIdSync(TASK_ID);
        assertEquals("Task name should be "+NEW_TASK_NAME, NEW_TASK_NAME, updatedTask.getName());
    }

    @Test
    public void delete(){
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);

        List<Task> taskList = taskDao.loadTasksSync();
        assertEquals("Task list should be of size = 1", 1,taskList.size());

        task = taskDao.getTaskByIdSync(TASK_ID);
        taskDao.delete(task);
        taskList = taskDao.loadTasksSync();
        assertEquals("Task list should be of size = 0", 0,taskList.size());

    }

}
```
### TaskItemDao Test
Here is the code for the test for `TaskItemDao.java`:

```java
@RunWith(AndroidJUnit4.class)
public class TaskItemDaoTest {

    private static final String TASK_NAME = "Groceries";
    private static final String ITEM_NAME = "Sugar";
    private static final int ID = 1;

    private TaskItemDao taskItemDao;
    private TaskDao taskDao;
    private YoTasksDb db;

    @Before
    public void setup(){
        Context context = InstrumentationRegistry.getTargetContext();
        db = Room.inMemoryDatabaseBuilder(context, YoTasksDb.class).build();
        taskDao = db.taskDao();
        taskItemDao = db.taskItemDao();
    }

    @After
    public void tearDown(){
        db.close();
    }


    @Test
    public void insert() throws Exception {
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);

        TaskItem item = TestUtil.createItem(ITEM_NAME, ID);
        taskItemDao.insert(item);

        List<TaskItem> taskItems = taskItemDao.loadItemsSync(ID);
        assertEquals("Task items list should be of size 1", 1, taskItems.size());
    }

    @Test
    public void getItemByIdSync() throws Exception {
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);

        TaskItem item = TestUtil.createItem(ITEM_NAME, ID);
        taskItemDao.insert(item);

        item = taskItemDao.getItemByIdSync(ID);
        assertNotNull(item);
    }

    @Test
    public void update() throws Exception {
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);

        TaskItem item = TestUtil.createItem(ITEM_NAME, ID);
        taskItemDao.insert(item);

        item = taskItemDao.getItemByIdSync(ID);
        assertFalse(item.isDone());

        item.setDone(true);
        taskItemDao.update(item);

        item = taskItemDao.getItemByIdSync(ID);
        assertTrue(item.isDone());
    }

    @Test
    public void delete() throws Exception {
        Task task = TestUtil.createTask(TASK_NAME);
        taskDao.insert(task);

        TaskItem item = TestUtil.createItem(ITEM_NAME, ID);
        taskItemDao.insert(item);

        item = taskItemDao.getItemByIdSync(ID);
        taskItemDao.delete(item);

        List<TaskItem> taskItems = taskItemDao.loadItemsSync(ID);
        assertEquals("Task items list should be of size 0", 0, taskItems.size());
    }

}
```
That's all we need to do today. When all our tests pass we can proceed with confidence since we know we won't have issues with our database.

# Summary
In this post we wrote tests for our database implementation to ensure that everything works as expected. Thanks again for taking your time to read. Happy coding!!
