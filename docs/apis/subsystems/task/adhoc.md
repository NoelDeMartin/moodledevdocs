---
title: Adhoc tasks
tags:
  - API
  - Tasks
  - Subsystem
  - Adhoc
---

Adhoc tasks are typically used when you need to queue something to run in the background either immediately, where they would be executed as soon as possible, or as a one-off task at some future point in time.

Each adhoc task can be called multiple times, with each having its own custom data, and the ability to run as a different user.

Adhoc tasks are great for situations such as:

- perform a pre-configured backup
- migrate large quantities of data between different formats
- send forum posts as an e-mail

## Creating adhoc tasks

To create a new adhoc task you should:

1. create a new class which extends the `\core\task\adhoc_task` class;
2. queue the task

### Task class

The class for your adhoc task, which extends the `\core\task\adhoc_task` class, should be in the `classes/task` directory of your plugin.

<details>

<summary>View example adhoc task</summary>

```php
namespace mod_example\task;

/**
 * An example of an adhoc task.
 */
class do_something extends \core\task\adhoc_task {

    /**
     * Execute the task.
     */
    public function execute() {
        // Call your own api
    }
}
```

</details>

### Queueing the task for execution

The processing of adhoc tasks is handled by the task manager. Typically you will instantiate a copy of your class, setting any relevant data, and then use the manager to queue the task, for example:

```php
// Create the instance
$mytask = new \mod_example\task\do_something();

// Run this task as a specific user:
$mytask->set_userid($someuser->id);

// Set some custom data.
// Note: This data must be serializable.
$mytask->set_custom_data([
    'documentation' => 'present',
    'correct' => true,
])

// Queue the task.
\core\task\manager::queue_adhoc_task($mytask);
```

### Task features

Adhoc tasks include a number of useful features which are important to be aware of.

#### Running as a specific user

Unless otherwise specified, all tasks will run as the CLI admin user. This is often undesirable and, where relevant, you should specify a userid to run the task as.

```php
// Run this task as a specific user:
$mytask->set_userid($someuser->id);
```

#### Set a task to run at a future time

You may need to plan to run a task at a future time - for example you may queue a forum digest task to run at a particular time. This can be accomplished using the `set_next_run_time()` function before queueing the task, for example:

```php
$task = new my_future_task();
$task->set_next_run_time($futuretime);
\core\task\manager::queue_adhoc_task($task);
```

:::note

The `set_next_run_time()` function takes a unix time stamp. Tasks are not _guaranteed_ to run at a specific time, but they will not be run _before_ that time.

:::

#### Ignore duplicate adhoc tasks

In some situations you may only wish to queue an adhoc task if an identical adhoc task does not already exist. This can be useful in situations where you are adding a set of items to a bucket for later processing and only wish to process all items once.

When requested, the `queue_adhoc_task()` function will ignore any task where all of the following match an existing task in the queue:

- `classname` - the class defining the task to be processed
- `component` - the component that the task belongs
- `customdata` - any custom data for this instance
- `userid` - the user that the task will be run as

Duplicate adhoc task detection can be enabled by passing a truthy value as the second argument to `queue_adhoc_task()`, for example:

```php
\core\task\manager::queue_adhoc_task($task, true);
```

:::tip Custom data

If creating tasks which will contain a subset of data which will also be run by another instance of the same task type, you should put the data into a database table rather than the task custom data.

:::

#### Rescheduling an adhoc task

Similar to [ignoring duplicate adhoc tasks](#ignore-duplicate-adhoc-tasks) there are situations which require you re_schedule_ a matching task instead. This may happen, for example, in a situation where there is too much churn of your source data.

The `reschedule_or_queue_adhoc_task()` function will locate any existing task matching the following fields, and update its time with the time from the new entry:

- `classname` - the class defining the task to be processed
- `component` - the component that the task belongs
- `customdata` - any custom data for this instance
- `userid` - the user that the task will be run as

```php
\core\task\manager::reschedule_or_queue_adhoc_task($task);
```
