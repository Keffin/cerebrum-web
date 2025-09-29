---
title: "Virtual threads for testing"
date: "2025-03-22"
summary: "Mimic pods using virtual threads."
toc: false
autonumber: false
math: true
tags: ["java", "threads", "testing"]
showTags: true
hideBackToTop: true
hidePagination: true
---

Have you ever had a scenario where you are attempting to write tests that should mimic pod-like behavior? Well, I have.

Sometimes a pod-like setup is difficult to reproduce and test locally.
E.g. scenarios where multiple instances of your application are attempting to access the same resource. 
I found a good way of testing these scenarios locally is to utilize multi-threading, where you can think of each spawned thread as a sort of pod or application instance.

As someone who also enjoys writing Java, at times, who could come up with a better scenario where I can get my hands dirty and use virtual threads?

Let’s make the example concrete: imagine you have 2 two instances of your application running in some arbitrary environment.

Now, suppose part of your complex domain logic requires running a task such as:
* Archiving or deleting old data
* Updating cached data
* Sending notifications

In these cases cases, you typically want only one pod/application instance to perform the task. Otherwise you risk invalid or inconsistent data states.

To avoid this, applications often use distributed locks, which ensure that important tasks are executed only once. If one pod acquires the lock, other pods are prevented from running the same task.

For the sake of simplicity let's propose that we have some simple interface for the locking mechanism. I can recommend looking into tools such as [Shedlock](https://github.com/lukas-krecan/ShedLock) or [Redis](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/) for this in more practice. But for now, we won't go into implementation detail.

```java
public enum LockStatus {
  LOCK_ACQUIRED,
  LOCK_SKIPPED,
}

public LockStatus lock(String task, String podId);
```

Now what this method does is that it returns `LOCK_ACQUIRED` if it was successful in acquiring the lock and otherwise `LOCK_SKIPPED`.

Then in our application code, a scheduled job can simply call `lock()` and check the return status. If the status is LOCK_ACQUIRED, it proceeds with the task. Straightforward enough.

### Testing
Now to the testing part. You could just test this live in any of your testing environments to verify that it is working, but let's say that you want to write local tests to verify that the locking mechanism is working and only 1 application instance can perform said task, preventing it from causing invalid data states.

For that, I used virtual threads in Java. You could use regular platform threads as well, but where is the fun in that, let's use some of the new java features. Virtual threads allow us to simulate these live concurrent environments without platform threads overhead.

```java
@SpringbootTest
public class IntegrationTest {

  @Autowired
  private SharedResourceAcquirer acquirer;

  private static final String TASK = "acquire_data";

  @Test
  void testLock() {
    ExecutorService vExec = Executors.newVirtualThreadPerTaskExecutor();

    Future<LockStatus> fstPod = vExec.submit(() -> acquirer.lock(TASK, "POD_1_ID"));
    Future<LockStatus> sndPod = vExec.submit(() -> acquirer.lock(TASK, "POD_2_ID"));


    if (fstPod.get() == LOCK_ACQUIRED) {
      assertThat(sndPod.get()).isEqualTo(LOCK_SKIPPED);
    } else {
      assertThat(sndPod.get()).isEqualTo(LOCK_ACQUIRED);
    }

    vExec.shutdown();
  }
}
```

Looking at this test, we spawn two threads to mimic pod-like behavior. Each thread tries to acquire the lock. Only one should succeed (`LOCK_ACQUIRED`), while the other should fail (`LOCK_SKIPPED`). This confirms that only one pod can perform the task, preventing invalid data state.
