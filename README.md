Download Link: https://assignmentchef.com/product/solved-csci2400-lab-assignment-l5-writing-your-own-unix-shell
<br>
<h1>Introduction</h1>

The purpose of this assignment is to become more familiar with the concepts of process control and signalling. You’ll do this by writing a simple Unix shell program that supports job control.

<h1>Logistics</h1>

You should work in a group of up to two people in solving the problems for this assignment. The only “hand-in” will be electronic. Any clarifications and revisions to the assignment will be posted on the course Moodle page.

<h1>Hand Out Instructions</h1>

You should be able to use the https://hub.csel.io system to complete this lab or your own Linux virtual machine. You’ll also be able to use 32 or 64 bit systems – you won’t be looking at the assembly code.

Download the file shlab-handout.tar from the Moodle “assignment” page.

Start by copying the file shlab-handout.tar to the protected directory (the <em>lab directory</em>) in which you plan to do your work. Then do the following:

<ul>

 <li>Type the command tar xvf shlab-handout.tar to expand the tarfile.</li>

 <li>Type the command make to compile and link some test routines.</li>

 <li>Enter your team member names in the header comment at the top of tsh.cc.</li>

</ul>

Looking at the tsh.cc (<em>tiny shell</em>) file, you will see that it contains a functional skeleton of a simple Unix shell. To help you get started, we have already implemented the less interesting functions; some of these are in the file helper_routines.cc, and these have been separated in another file ot make it easier to focus on the work you need to do in tsh.cc.

Your assignment is to complete the remaining empty functions listed below in tsh.cc. As a sanity check for you, we’ve listed the approximate number of lines of code for each of these functions in our reference solution (which includes lots of comments).

<ul>

 <li>eval: Main routine that parses and interprets the command line. [70 lines]</li>

 <li>builtincmd: Recognizes and interprets the built-in commands: quit, fg, bg, and jobs. [25 lines]</li>

 <li>dobgfg: Implements the bg and fg built-in commands. [50 lines]</li>

 <li>waitfg: Waits for a foreground job to complete. [20 lines]</li>

 <li>sigchldhandler: Catches SIGCHILD signals. 80 lines]</li>

 <li>siginthandler: Catches SIGINT (ctrl-c) signals. [15 lines]</li>

 <li>sigtstphandler: Catches SIGTSTP (ctrl-z) signals. [15 lines]</li>

</ul>

Each time you modify your tsh.cc file, type make to recompile it. To run your shell, type ./tsh to the command line:

unix&gt; ./tsh

tsh&gt; <em>[type commands to your shell here]</em>

<h1>General Overview of Unix Shells</h1>

A <em>shell </em>is an interactive command-line interpreter that runs programs on behalf of the user. A shell repeatedly prints a prompt, waits for a <em>command line </em>on stdin, and then carries out some action, as directed by the contents of the command line.

The command line is a sequence of ASCII text words delimited by whitespace. The first word in the command line is either the name of a built-in command or the pathname of an executable file. The remaining words are command-line arguments. If the first word is a built-in command, the shell immediately executes the command in the current process. Otherwise, the word is assumed to be the pathname of an executable program. In this case, the shell forks a child process, then loads and runs the program in the context of the child. The child processes created as a result of interpreting a single command line are known collectively as a <em>job</em>. In general, a job can consist of multiple child processes connected by Unix pipes.

If the command line ends with an ampersand ”&amp;”, then the job runs in the <em>background</em>, which means that the shell does not wait for the job to terminate before printing the prompt and awaiting the next command line. Otherwise, the job runs in the <em>foreground</em>, which means that the shell waits for the job to terminate before awaiting the next command line. Thus, at any point in time, at most one job can be running in the foreground. However, an arbitrary number of jobs can run in the background.

For example, typing the command line

tsh&gt; <em>jobs</em>

causes the shell to execute the built-in jobs command. Typing the command line

tsh&gt; <em>/bin/ls -l -d</em>

runs the ls program in the foreground. By convention, the shell ensures that when the program begins executing its main routine

int main(int argc, char *argv[])

the argc and argv arguments have the following values:

<ul>

 <li>argc == 3,</li>

 <li>argv[0] == ‘‘/bin/ls’’,</li>

 <li>argv[1]== ‘‘-l’’,</li>

 <li>argv[2]== ‘‘-d’’.</li>

</ul>

Alternatively, typing the command line

tsh&gt; <em>/bin/ls -l -d &amp;</em>

runs the ls program in the background.

Unix shells support the notion of <em>job control</em>, which allows users to move jobs back and forth between background and foreground, and to change the process state (running, stopped, or terminated) of the processes in a job. Typing ctrl-c causes a SIGINT signal to be delivered to each process in the foreground job. The default action for SIGINT is to terminate the process. Similarly, typing ctrl-z causes a SIGTSTP signal to be delivered to each process in the foreground job. The default action for SIGTSTP is to place a process in the stopped state, where it remains until it is awakened by the receipt of a SIGCONT signal. Unix shells also provide various built-in commands that support job control. For example:

<ul>

 <li>jobs: List the running and stopped background jobs.</li>

 <li>bg &lt;job&gt;: Change a stopped background job to a running background job.</li>

 <li>fg &lt;job&gt;: Change a stopped or running background job to a running in the foreground.</li>

 <li>kill &lt;job&gt;: Terminate a job.</li>

</ul>

<h1>The tsh Specification</h1>

Your tsh shell should have the following features:

<ul>

 <li>The prompt should be the string “tsh&gt; ”.</li>

 <li>The command line typed by the user should consist of a name and zero or more arguments, all separated by one or more spaces. If name is a built-in command, then tsh should handle it immediately and wait for the next command line. Otherwise, tsh should assume that name is the path of an executable file, which it loads and runs in the context of an initial child process (In this context, the term <em>job </em>refers to this initial child process).</li>

 <li>tsh need not support pipes (|) or I/O redirection (&lt; and &gt;).</li>

 <li>Typing ctrl-c (ctrl-z) should cause a SIGINT (SIGTSTP) signal to be sent to the current foreground job, as well as any descendents of that job (e.g., any child processes that it forked). If there is no foreground job, then the signal should have no effect.</li>

 <li>If the command line ends with an ampersand &amp;, then tsh should run the job in the background. Otherwise, it should run the job in the foreground.</li>

 <li>Each job can be identified by either a process ID (PID) or a job ID (JID), which is a positive integer assigned by tsh. Job ID’s are used because some scripts need to manipulate certain jobs, and the process ID’s change across runs. JIDs should be denoted on the command line by the prefix ’%’. For example, “%5” denotes JID 5, and “5” denotes PID 5. (We have provided you with all of the routines you need for manipulating the job list.)</li>

 <li>tsh should support the following built-in commands:

  <ul>

   <li>The quit command terminates the shell.</li>

   <li>The jobs command lists all background jobs.</li>

   <li>The bg &lt;job&gt; command restarts &lt;job&gt; by sending it a SIGCONT signal, and then runs it in the background. The &lt;job&gt; argument can be either a PID or a JID.</li>

   <li>The fg &lt;job&gt; command restarts &lt;job&gt; by sending it a SIGCONT signal, and then runs it in the foreground. The &lt;job&gt; argument can be either a PID or a JID.</li>

  </ul></li>

 <li>tsh should reap all of its zombie children. If any job terminates because it receives a signal that it didn’t catch, then tsh should recognize this event and print a message with the job’s PID and a description of the offending signal.</li>

</ul>

<h1>Checking Your Work</h1>

We have provided some tools to help you check your work.

Reference solution. The Linux executable tshref is the reference solution for the shell. Run this program to resolve any questions you have about how your shell should behave. <em>Your shell should emit output that is identical to the reference solution </em>(except for process identifiers (PIDs) that change from run to run).

Shell driver. The sdriver.pl program executes a shell as a child process, sends it commands and signals as directed by a <em>trace file</em>, and captures and displays the output from the shell.

Use the -h argument to find out the usage of sdriver.pl:

unix&gt; ./sdriver.pl -h

Usage: sdriver.pl [-hv] -t &lt;trace&gt; -s &lt;shellprog&gt; -a &lt;args&gt; Options:

-h                                        Print this message

-v                                     Be more verbose

-t &lt;trace&gt;                    Trace file

<table width="351">

 <tbody>

  <tr>

   <td width="112">-s &lt;shell&gt;</td>

   <td width="239">Shell program to test</td>

  </tr>

  <tr>

   <td width="112">-a &lt;args&gt;</td>

   <td width="239">Shell arguments</td>

  </tr>

  <tr>

   <td width="112">-g</td>

   <td width="239">Generate output for autograder</td>

  </tr>

 </tbody>

</table>

We have also provided 16 trace files (trace{01-16}.txt) that you will use in conjunction with the shell driver to test the correctness of your shell. The lower-numbered trace files do very simple tests, and the higher-numbered tests do more complicated tests.

You can run the shell driver on your shell using trace file trace01.txt (for instance) by typing:

unix&gt; <em>./sdriver.pl -t trace01.txt -s ./tsh -a “-p”</em>

(the -a “-p” argument tells your shell not to emit a prompt), or

unix&gt; <em>make test01</em>

Similarly, to compare your result with the reference shell, you can run the trace driver on the reference shell by typing:

unix&gt; <em>./sdriver.pl -t trace01.txt -s ./tshref -a “-p”</em>

or

unix&gt; <em>make rtest01</em>

For your reference, tshref.out gives the output of the reference solution on all races. This might be more convenient for you than manually running the shell driver on all trace files.

The neat thing about the trace files is that they generate the same output you would have gotten had you run your shell interactively (except for an initial comment that identifies the trace). For example:

bass&gt; make test15

./sdriver.pl -t trace15.txt -s ./tsh -a “-p”

#

# trace15.txt – Putting it all together

# tsh&gt; ./bogus

./bogus: Command not found.

tsh&gt; ./myspin 10

Job (9721) terminated by signal 2 tsh&gt; ./myspin 3 &amp; [1] (9723) ./myspin 3 &amp; tsh&gt; ./myspin 4 &amp; [2] (9725) ./myspin 4 &amp; tsh&gt; jobs

[1] (9723) Running               ./myspin 3 &amp; [2] (9725) Running ./myspin 4 &amp; tsh&gt; fg %1

Job [1] (9723) stopped by signal 20

tsh&gt; jobs

[1] (9723) Stopped               ./myspin 3 &amp; [2] (9725) Running ./myspin 4 &amp; tsh&gt; bg %3 %3: No such job tsh&gt; bg %1

<table width="271">

 <tbody>

  <tr>

   <td colspan="2" width="271">[1] (9723) ./myspin 3 &amp;</td>

  </tr>

  <tr>

   <td width="175">tsh&gt; jobs</td>

   <td width="96"> </td>

  </tr>

  <tr>

   <td width="175">[1] (9723) Running</td>

   <td width="96">./myspin 3 &amp;</td>

  </tr>

  <tr>

   <td width="175">[2] (9725) Running tsh&gt; fg %1 tsh&gt; quit bass&gt;</td>

   <td width="96">./myspin 4 &amp;</td>

  </tr>

 </tbody>

</table>

<h1>Hints</h1>

<ul>

 <li>Read every word of the chapter on Exceptional Control Flow in your textbook.</li>

 <li>Use the trace files to guide the development of your shell. Starting with trace01.txt, make sure that your shell produces the <em>identical </em>output as the reference shell. Then move on to trace file trace02.txt, and so on.</li>

 <li>The waitpid, kill, fork, execve, setpgid, and sigprocmask functions will come in very handy. The WUNTRACED and WNOHANG options to waitpid will also be useful. These are described in detail in the text.</li>

 <li>When you implement your signal handlers, be sure to send SIGINT and SIGTSTP signals to the entire foreground process group, using ”-pid” instead of ”pid” in the argument to the kill function. The sdriver.pl program tests for this error.</li>

 <li>One of the tricky parts of the assignment is deciding on the allocation of work between the waitfg and sigchldhandler functions. We recommend the following approach:

  <ul>

   <li>In waitfg, use a busy loop around the sleep function.</li>

   <li>In sigchldhandler, use exactly one call to waitpid.</li>

  </ul></li>

</ul>

While other solutions are possible, such as calling waitpid in both waitfg and sigchldhandler, these can be very confusing. It is simpler to do all reaping in the handler.

<ul>

 <li>In eval, the parent must use sigprocmask to block SIGCHLD signals before it forks the child, and then unblock these signals, again using sigprocmask after it adds the child to the job list by calling addjob. Since children inherit the blocked vectors of their parents, the child must be sure to then unblock SIGCHLD signals before it execs the new program.</li>

</ul>

The parent needs to block the SIGCHLD signals in this way in order to avoid the race condition where the child is reaped by sigchldhandler (and thus removed from the job list) <em>before </em>the parent calls addjob.

<ul>

 <li>Programs such as more, less, vi, and emacs do strange things with the terminal settings. Don’t run these programs from your shell. Stick with simple text-based programs such as /bin/ls, /bin/ps, and /bin/echo.</li>

 <li>When you run your shell from the standard Unix shell, your shell is running in the foreground process group. If your shell then creates a child process, by default that child will also be a member of the foreground process group. Since typing ctrl-c sends a SIGINT to every process in the foreground group, typing ctrl-c will send a SIGINT to your shell, as well as to every process that your shell created, which obviously isn’t correct.</li>

</ul>

Here is the workaround: After the fork, but before the execve, the child process should call setpgid(0, 0), which puts the child in a new process group whose group ID is identical to the child’s PID. This ensures that there will be only one process, your shell, in the foreground process group. When you type ctrl-c, the shell should catch the resulting SIGINT and then forward it to the appropriate foreground job (or more precisely, the process group that contains the foreground job).

<h1>Evaluation</h1>

During your grading session, your solution shell will be tested for correctness on a Linux machine (either your virtual machine or native linux install) using the same shell driver and trace files that were included in your lab directory. Your shell should produce identical output on these traces as the reference shell, with only two exceptions:

<ul>

 <li>The PIDs can (and will) be different.</li>

 <li>The output of the /bin/ps commands in trace11.txt, trace12.txt, and trace13.txt will be different from run to run. However, the running states of any mysplit processes in the output of the /bin/ps command should be identical.</li>

</ul>

The “correctness” part of your assignment will be computed based on how many of the traces you correctly execute using following distribution:

<ul>

 <li>10% – traces 1-3</li>

 <li>30% – traces 4-5</li>

 <li>60% – traces 6-8</li>

 <li>80% – traces 9-10</li>

 <li>90% – traces 11-13</li>

 <li>100% – trace 14-16</li>

</ul>

As with all assignments in CSCI 2400, you will need to be able to explain the function of your code in your grading meeting. Correctness is only part of your final grade.

<h1>Hand In Instructions</h1>

You should only have to change tsh.cc. You need to upload tsh.cc to the Moodle assignment to mark your completion time. You should use the same version of your code during your grading meeting.

Good luck!