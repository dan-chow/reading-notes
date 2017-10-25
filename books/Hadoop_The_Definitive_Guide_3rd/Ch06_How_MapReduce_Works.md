## Chapter 06: How MapReduce Works

### Anatomy of a MapReduce Job Run

In Hadoop 2.0, a new MapReduce implementation was introduced. The new implementation (called MapReduce 2) is built on a system called YARN. For now, just note that the framework that is used for
execution is set by the mapreduce.framework.name property, which takes the values
local (for the local job runner), classic (for the “classic” MapReduce framework, also
called MapReduce 1, which uses a jobtracker and tasktrackers), and yarn (for the new
framework).

How Hadoop runs a MapReduce job using the classic framework
fig_6_1_How_Hadoop_runs_a_MapReduce_job_using_the_classic_framework.PNG

The job submission process implemented by JobSummitter does the following:
• Asks the jobtracker for a new job ID (by calling getNewJobId() on JobTracker).
• Checks the output specification of the job.
• Computes the input splits for the job.
• Copies the resources needed to run the job, including the job JAR file, the configuration file, and the computed input splits, to the jobtracker’s filesystem in a
directory named after the job ID.
• Tells the jobtracker that the job is ready for execution by calling submitJob() on
JobTracker.


In addition to the map and reduce tasks, two further tasks are created: a job setup task
and a job cleanup task. These are run by tasktrackers and are used to run code to set
up the job before any map tasks run, and to cleanup after all the reduce tasks are
complete.


Tasktrackers run a simple loop that periodically sends heartbeat method calls to the
jobtracker. Heartbeats tell the jobtracker that a tasktracker is alive, but they also double
as a channel for messages. As a part of the heartbeat, a tasktracker will indicate whether
it is ready to run a new task, and if it is, the jobtracker will allocate it a task, which it
communicates to the tasktracker using the heartbeat return value.

To choose a reduce task, the jobtracker simply takes the next in its list of yet-to-be-run
reduce tasks, since there are no data locality considerations. For a map task, however,
it takes into account the tasktracker’s network location and picks a task whose input
split is as close as possible to the tasktracker.

TaskRunner launches a new Java Virtual Machine to run each task in, so that any bugs in the user-defined map and reduce functions don’t affect the
tasktracker (by causing it to crash or hang, for example). However, it is possible to
reuse the JVM between tasks.

The child process communicates with its parent through the umbilical interface. It informs the parent of the task’s progress every few seconds until the task is complete.

Both Streaming and Pipes run special map and reduce tasks for the
purpose of launching the user-supplied executable and communicating with it.
In the case of Streaming, the Streaming task communicates with the process (which
may be written in any language) using standard input and output streams. The Pipes
task, on the other hand, listens on a socket and passes the C++ process a port number
in its environment so that on startup, the C++ process can establish a persistent socket
connection back to the parent Java Pipes task.

The relationship of the Streaming and Pipes executable to the tasktracker and its child
fig_6_2_The_relationship_of_the_Streaming_and_Pipes_executable_to_the_tasktracker_and_its_child.PNG


A job and each of its tasks have a status, which
includes such things as the state of the job or task (e.g., running, successfully completed,
failed), the progress of maps and reduces, the values of the job’s counters, and a status
message or description (which may be set by user code).
When a task is running, it keeps track of its progress, that is, the proportion of the task
completed. For map tasks, this is the proportion of the input that has been processed.
For reduce tasks, it’s a little more complex, but the system can still estimate the proportion of the reduce input processed. It does this by dividing the total progress into
three parts, corresponding to the three phases of the shuffle.


All of the following operations constitute progress:
• Reading an input record (in a mapper or reducer)
• Writing an output record (in a mapper or reducer)
• Setting the status description on a reporter (using Reporter’s setStatus() method)
• Incrementing a counter (using Reporter’s incrCounter() method)
• Calling Reporter’s progress() method

If a task reports progress, it sets a flag to indicate that the status change should be sent
to the tasktracker. The flag is checked in a separate thread every three seconds, and if
set, it notifies the tasktracker of the current task status. Meanwhile, the tasktracker is
sending heartbeats to the jobtracker every five seconds (this is a minimum, as the
heartbeat interval is actually dependent on the size of the cluster; for larger clusters,
the interval is longer), and the status of all the tasks being run by the tasktracker is sent
in the call. Counters are sent less frequently than every five seconds because they can
be relatively high-bandwidth.
The jobtracker combines these updates to produce a global view of the status of all the
jobs being run and their constituent tasks. Finally, as mentioned earlier, the Job receives
the latest status by polling the jobtracker every second. Clients can also use Job’s
getStatus() method to obtain a JobStatus instance, which contains all of the status
information for the job.


How status updates are propagated through the MapReduce 1 system
fig_6_3_How_status_updates_are_propagated_through_the_MapReduce_1_system.PNG


When the jobtracker receives a notification that the last task for a job is complete (this
will be the special job cleanup task), it changes the status for the job to “successful.”
The jobtracker also sends an HTTP job notification if it is configured to do so.

In 2010 a group
at Yahoo! began to design the next generation of MapReduce. The result was YARN,
short for Yet Another Resource Negotiator (or if you prefer recursive acronyms, YARN
Application Resource Negotiator).

YARN remedies the scalability shortcomings of “classic” MapReduce by splitting the
responsibilities of the jobtracker into separate entities. The jobtracker takes care of both
job scheduling (matching tasks with tasktrackers) and task progress monitoring (keeping track of tasks, restarting failed or slow tasks, and doing task bookkeeping, such as
maintaining counter totals).
YARN separates these two roles into two independent daemons: a resource manager
to manage the use of resources across the cluster and an application master to manage
the lifecycle of applications running on the cluster. The idea is that an application
master negotiates with the resource manager for cluster resources—described in terms
of a number of containers, each with a certain memory limit—and then runs application-specific processes in those containers. The containers are overseen by node managers running on cluster nodes, which ensure that the application does not use more
resources than it has been allocated.

How Hadoop runs a MapReduce job using YARN
fig_6_4_How_Hadoop_runs_a_MapReduce_job_using_YARN.PNG

The new job ID is retrieved from the resource manager (rather than
the jobtracker), although in the nomenclature of YARN, it is an application ID. The job client checks the output specification of the job; computes input splits; and copies
job resources to HDFS. Finally, the job is submitted by calling submitApplication() on the resource
manager.

When the resource manager receives a call to its submitApplication(), it hands off the
request to the scheduler. The scheduler allocates a container, and the resource manager
then launches the application master’s process there, under the node manager’s management.
The application master for MapReduce jobs is a Java application whose main class is
MRAppMaster. It initializes the job by creating a number of bookkeeping objects to keep
track of the job’s progress, as it will receive progress and completion reports from the
tasks. Next, it retrieves the input splits computed in the client from the shared
filesystem. It then creates a map task object for each split, as well as a number
of reduce task objects determined by the mapreduce.job.reduces property.
The next thing the application master does is decide how to run the tasks that make
up the MapReduce job. If the job is small, the application master may choose to run
the tasks in the same JVM as itself.

Before any tasks can be run, the job setup method is called (for the job’s OutputCommit
ter) to create the job’s output directory.