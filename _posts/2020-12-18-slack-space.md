# What is Slack Space?

File Slack, also called 'slack space', is the leftover space on a drive where a file is stored. This space remains empty or left over because each cluster on a disk has a storage threshold and files are of varying sizes. Therefore, the files only fill a part of the hard drive portion. 

## How to Utilize File Slack

 The most prevalent benefit occurs in the computer forensics field, as file slack allows users to locate files deleted from sectors. Deleting computer files doesn’t fully delete them – it just moves them. This provides investigators potential clues concerning the data erased by legal suspects on the hard drive.

### Hiding and Viewing Data in Slack Space

You can use slack space to hide data as stated above. You can do this by installing and using `bmap`, which can be found [here](https://github.com/CameronLonsdale/bmap).

Now we can start by making a file that we will use to hide data using the `echo` command:

```
eris@fnord:~/TestChamber$ echo "This will be the file we hide our data in." > slackSpace.txt
eris@fnord:~/TestChamber$ cat slackSpace.txt 
This will be the file we hide our data in.
```

From the above output we made a file containing some text and named it `slackSpace.txt`. 

Next, we can user `bmap` to view view the amount of slack space found within the file:

```
eris@fnord:~/TestChamber$ sudo bmap --mode slack slackSpace.txt
getting from block 4751464
file size was: 43
slack size: 4053
block size: 4096
eris@fnord:~/TestChamber$ 
```

The file size is 43 bytes. The block size is 4096 and if we subtract 43 from 4096, we'll see that there is 4053 bytes of slack space available in this file. If we wanted to hide a secret message within this slack space we can do, like so:

```
eris@fnord:~/TestChamber$ echo "Secret message." | sudo bmap --mode putslack slackSpace.txt
stuffing block 4751464
file size was: 43
slack size: 4053
block size: 4096

eris@fnord:~/TestChamber$ cat slackSpace.txt 
This will be the file we hide our data in.

eris@fnord:~/TestChamber$ ls -l slackSpace.txt 
-rw-rw-r-- 1 eris eris 43 Oct 22 11:30 slackSpace.txt
```

Basically, we used `bmap` in the `putslack --mode` to place the string "Secret message," into the slack space of our text file. Notice if we use `ls -l` the file size remains the same an if we try to read the file we see ONLY the original text.

So we will use `bmap` to view the slack space again:

```
eris@fnord:~/TestChamber$ sudo bmap --mode slack slackSpace.txt
getting from block 4751464
file size was: 43
slack size: 4053
block size: 4096
Secret message.
eris@fnord:~/TestChamber$
```

Finally, in the above output we can see where the data we saw before was presented, mostly unchanged, only the string we added to the slack space of the file is now printed at the end of the output of the command.
