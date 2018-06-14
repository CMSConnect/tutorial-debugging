# Basic debugging example
Basic debugging when jobs are held

## Submit job
Job requires an input file and writes to a "log" directory.

Let's start submitting your job
```
$ condor_submit example01.submit
1 job(s) submitted to cluster 3875796.
```

After a while, you will notice your job is held.

```
$ condor_q 3875796.0
3875796.0   khurtado        6/14 03:54   0+00:00:02 H  0   0.0  connect_wrapper.sh
```

To see the reason, you can type:

```
$ condor_q 3875796.0 -hold


-- Schedd: login.uscms.org : <192.170.227.118:9618?...
 ID         OWNER          HELD_SINCE  HOLD_REASON
3875796.0   khurtado        6/14 03:55 Failed to initialize user log to /home/khurtado/hats/tutorial-debugging/log/job.log.3875796 or /dev/null

1 jobs; 0 completed, 0 removed, 0 idle, 0 running, 1 held, 0 suspended
```

This shows the job failed because the user log could not be written. This is due to the fact the directory "log" was not created. Let us create the directory and release the job:

```
$ mkdir log
$ condor_release 3875796.0
Job 3875796.0 released
```
Your job will run again, but will get hold after some time. Let's see why this time:
```
$ condor_q khurtado -hold


-- Schedd: login.uscms.org : <192.170.227.118:9618?...
 ID         OWNER          HELD_SINCE  HOLD_REASON
3875796.0   khurtado        6/14 05:41 Error from glidein_22979_179320449@wn41.ncp.edu.pk: SHADOW at 192.170.227.118 failed to send file(s) to <111.68.99.146:9982>: error reading from /home/khurtado/hats/tutorial-debugging/message1.txt: (errno 2) No such file or directory; STARTER failed to receive file(s) from <192.170.227.118:9618>
```

Here, condor says it was unable to transfer the file message1.txt. This is correct, beause the input filename should be: message.txt. Let's fix that by editing the job.

In condor, transfer\_input\_files translates into the TransferInput classad. Let's see the current content:

```
$ condor_q 3875796.0 -af TransferInput
example01.sh,message1.txt
```

Let's change message1.txt by message.txt
```
$ condor_qedit 3875796.0 TransferInput '"example01.sh,message.txt"'
Set attribute "TransferInput".

$ condor_q 3875796.0 -af TransferInput
example01.sh,message.txt
```

Note the single quotes and double quotes after. TransferInput is a string, so it needs double quotes. Other values like MaxWallTimeMins are integers, so single quotes alone would be used in that case.

After you have changed this value, you can release your job again:

```
$ condor_release 3875796.0
```

This time, your job should complete successfully.
