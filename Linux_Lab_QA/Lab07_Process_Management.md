# Lab 7: Process Management

**Q1. Display processes owned by the current user.**
```bash
ps
ps -u "$USER"
```
Common columns include PID (process ID), TTY (terminal), TIME (CPU time), and CMD (command). The user-oriented format may also display STAT: `R` running, `S` sleeping, `T` stopped, and `Z` zombie.

**Q2. Display all processes.**
```bash
ps -e
ps -ef
```
`ps -ef` includes UID, PID, PPID, C, STIME, TTY, TIME, and CMD columns.

**Q3. Use `ps aux` to inspect CPU and memory usage.**
```bash
ps aux
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```
Common columns include USER, PID, `%CPU`, `%MEM`, VSZ, RSS, TTY, STAT, START, TIME, and COMMAND.

**Q4. Display parent-child process relationships.**
```bash
ps -ef --forest
ps axjf
pstree -p
```
Install `pstree` if needed: `sudo apt install psmisc`.

**Q5. Run `top` and inspect the process list.**
```bash
top
```
The summary shows uptime, load average, tasks, CPU, memory, and swap. The process list includes PID, USER, PR, NI, VIRT, RES, SHR, S, `%CPU`, `%MEM`, TIME+, and COMMAND. Press `q` to quit.

**Q6. Find a high-CPU or high-memory process in `top` and note its PID.**
```bash
yes > /dev/null &
top
```
In `top`, press `P` to sort by CPU or `M` to sort by memory. Stop only the test process started above when finished:
```bash
kill "$!"
```

**Q7. Run `htop`.**
```bash
htop
```
If it is not installed, run `sudo apt update && sudo apt install htop`. Common keys include F3 (search), F4 (filter), F5 (tree view), F6 (sort), F9 (send a signal), and F10 (quit).

**Q8. Run a long command in the foreground and stop it with Ctrl+C.**
```bash
sleep 300
```
Press Ctrl+C to send SIGINT.

**Q9. Run a command in the background with `&` and inspect the job information.**
```bash
sleep 300 &
```
The shell prints a job number and PID, for example `[1] 12345`.

**Q10. Display the current shell's jobs.**
```bash
jobs
```

**Q11. Bring a job to the foreground and terminate it safely.**
```bash
fg %1
```
Press Ctrl+C, then run `jobs` to confirm that the job has ended.

**Q12. Suspend a foreground process with Ctrl+Z.**
```bash
sleep 300
```
Press Ctrl+Z, then run `jobs` to see the stopped job.

**Q13. Resume a stopped job in the background.**
```bash
bg %1
jobs
```

**Q14. Display the shell job number and PID with `jobs -l`.**
```bash
jobs -l
```

**Q15. Select a process by PID and terminate it with SIGTERM.**
```bash
sleep 500 &
test_pid=$!
echo "$test_pid"
kill "$test_pid"
ps -p "$test_pid"
```
No `ps` entry means the process has exited. It may take a short time to exit.

**Q16. Explain why SIGTERM is generally used before SIGKILL.**
```bash
sleep 500 &
kill -TERM "$!"
```
SIGTERM requests that a process terminate and allows it to perform cleanup. SIGKILL cannot be caught or ignored and prevents cleanup, so use it only if a process does not respond to SIGTERM.

**Q17. If a test process ignores SIGTERM, use SIGKILL as a last resort.**
```bash
sh -c 'trap "" TERM; while :; do sleep 1; done' &
test_pid=$!
kill -TERM "$test_pid"
sleep 1
ps -p "$test_pid"
kill -KILL "$test_pid"
sleep 1
ps -p "$test_pid"
```
This is a test process. Do not send signals to unrelated processes.

**Q18. Use `pkill` to stop a named test process.**
```bash
sleep 600 &
pgrep -a sleep
pkill -x sleep
pgrep -a sleep
```
`pkill` can match multiple processes. Use it only in a test environment and confirm the exact process name and scope first.

**Q19. Manage multiple test processes with `ps`, `jobs`, and signals.**
```bash
sleep 700 &
sleep 800 &
sleep 900 &
jobs -l
ps -o pid,stat,cmd -C sleep
kill -STOP %2
jobs
kill -CONT %2
kill %1
kill %3
jobs
```
Job numbers depend on the current shell; verify them with `jobs -l` before signaling.

**Q20. Complete workflow.**
```bash
ps aux | head
sleep 1000
```
Press Ctrl+Z to suspend the foreground command, then continue:
```bash
bg %1
jobs -l
kill -STOP %1
jobs
kill -CONT %1
jobs
kill %1
jobs
pgrep -a -x sleep
```
Verify that the test process has ended. Do not use a broad `pkill sleep` cleanup command on a shared system.
