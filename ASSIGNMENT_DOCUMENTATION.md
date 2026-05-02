# Assignment 3 - Complete Documentation

**Student Name**: [Your Full Name]  
**Student ID**: [Your ID]  
**Date Submitted**: [Submission Date]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [30/5/2026, 6:00]
**change id to my id and inster the code in vs**: 

**There are no challenges  **: 

**find the line and inster the code in vs **: 

**esey**: 

**10 min**: 

---

### Entry 2 - [1/6/2026, 7:23]
**slove todo 1 and 2**: 

**need to know how to do it but was easy**: 

**Searching on the Internet**: 

**hard **: 

**2 hour**: 

---

### Entry 3 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

### Entry 4 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[ SharedResources.contextSwitchCount, which is incremented inside incrementContextSwitch() without any synchronization. Since multiple process threads call this method concurrently, two threads may read the same old value and both write back the same incremented value, causing lost updates. This leads to incorrect statistics where the total number of context switches becomes lower than the real number.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[Your answer here - explain your implementation choices]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Your answer here - reference try-finally blocks, lock ordering, etc.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[Your answer here - explain coarse-grained vs fine-grained locking, independence of counters, concurrency implications. Show understanding of when to use each approach. 5-8 sentences expected.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
SharedResources
contextSwitchCount
completedProcessCount
totalWaitingTime

**Why they need protection**: 
These variables are updated by multiple process threads at the same time. Without synchronization, two threads may read the same old value and overwrite each other’s updates, causing lost increments and incorrect statistics. Since these counters are used to compute final scheduling metrics, race conditions would lead to inaccurate results and inconsistent behavior.

**Synchronization mechanism used**: 
I used a ReentrantLock to ensure mutual exclusion. Only one thread at a time can update the shared counters, preventing lost updates and ensuring atomic read‑modify‑write operations.

**Code snippet**:
```java
// Paste your implementation here
// Inside SharedResources
public static final ReentrantLock lock = new ReentrantLock();

public static void incrementContextSwitch() {
    lock.lock();
    try {
        contextSwitchCount++;
    } finally {
        lock.unlock();
    }
}

public static void incrementCompletedProcess() {
    lock.lock();
    try {
        completedProcessCount++;
    } finally {
        lock.unlock();
    }
}

public static void addWaitingTime(long time) {
    lock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        lock.unlock();
    }
}

```

**Justification**: 
 ReentrantLock is the correct choice because these operations require strict mutual exclusion—only one thread should modify the counters at a time. The lock ensures that updates are atomic and prevents race conditions that would corrupt the final statistics. Using a lock also keeps the implementation simple and readable while guaranteeing thread‑safe access to shared numeric variables.

---

### Critical Section #2: Execution Log

**What resource**: 
The shared resource is the executionLog list inside SharedResources, which is implemented as a regular ArrayList.

**Why it needs protection**: 
ArrayList is not thread‑safe, and multiple process threads may attempt to write to the log at the same time. Concurrent modifications can corrupt the internal structure of the list, cause lost log entries, or even trigger runtime exceptions such as ConcurrentModificationException. Since the execution log is used to record scheduling events, any corruption would lead to incomplete or inaccurate logging.

**Synchronization mechanism used**: 
I used the same ReentrantLock that protects the counters. This ensures that only one thread at a time can append to the log, making the write operation atomic and preventing structural corruption.

**Code snippet**:
// Inside SharedResources
public static final ReentrantLock lock = new ReentrantLock();

public static void logExecution(String message) {
    lock.lock();
    try {
        executionLog.add(message);
    } finally {
        lock.unlock();
    }
}


**Justification**: 
A ReentrantLock is appropriate because writing to an ArrayList requires exclusive access—only one thread can safely modify the list at a time. Using the same lock for all shared resources keeps the design simple and prevents interleaving writes that could corrupt the log. This guarantees that every scheduling event is recorded correctly and in a thread‑safe manner

---

### Critical Section #3: CPU Semaphore 

**Purpose of semaphore**: 
The semaphore is used to control how many processes are allowed to execute on the CPU at the same time. Since a real CPU can only run one process per core, the semaphore ensures that only one thread enters the execution section at a time, preserving the correct Round Robin scheduling behavior

**Number of permits and why**:
I used 1 permit, meaning only one process thread can acquire the CPU at any given moment. This models a single‑core CPU, preventing multiple processes from running their time quantum simultaneously, which would break the logic of context switching and waiting time calculation.

**Where implemented**: 
The semaphore is implemented inside the SharedResources class and used in the run() and runToCompletion() methods of the Process class.
It is acquired before execution begins and released in a finally block to avoid deadlocks.

**Code snippet**:
```java
// Paste your implementation here
// Inside SharedResources
public static final Semaphore cpuSemaphore = new Semaphore(1);

// Inside Process.run()
@Override
public void run() {
    try {
        SharedResources.cpuSemaphore.acquire();   // Acquire CPU

        if (startTime == -1) {
            startTime = System.currentTimeMillis();
        }

        SharedResources.incrementContextSwitch();
        SharedResources.logExecution(name + " started quantum execution");

        // ... quantum execution logic ...

    } catch (InterruptedException e) {
        System.out.println("Process interrupted.");
    } finally {
        SharedResources.cpuSemaphore.release();   // Always release CPU
    }
}

// Inside Process.runToCompletion()
public void runToCompletion() {
    try {
        SharedResources.cpuSemaphore.acquire();

        Thread.sleep(remainingTime);
        remainingTime = 0;

        SharedResources.addWaitingTime(getWaitingTime());
        SharedResources.incrementCompletedProcess();

    } catch (InterruptedException e) {
        System.out.println("Process interrupted.");
    } finally {
        SharedResources.cpuSemaphore.release();
    }
}

```

**Effect on program behavior**: 
Using a semaphore ensures that only one process executes at a time, which preserves the correctness of the Round Robin simulation. It prevents overlapping execution, eliminates timing inconsistencies, and ensures that context switches, waiting times, and log entries reflect realistic CPU scheduling behavior. Without the semaphore, multiple threads would run concurrently, producing incorrect statistics and breaking the simulation.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 

**Results**: 

**What this proves**: 

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 

**Actual values**: 

**Analysis**: 

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 

**Example 2**: 

---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
