# Using Stata on FarmShare

## 1. Getting started with FarmShare

FarmShare is a cloud computing environment for Stanford affiliates. If you don't have experience with a Unix environment, it can take a little getting used to. There are several different computing clusters on FarmShare (including [rice, wheat, and oat](https://web.stanford.edu/group/farmshare/cgi-bin/wiki/index.php/FarmShare_2#rice.stanford.edu)). The rice cluster is for general use, interactive work.

To get started, you first have to log in to rice. You can do this by using `ssh` on the command line or an SSH client.

To SSH into Farmshare, either on Windows (10) or Mac, you can type (replacing sunetid with your username):

```bash
ssh sunetid@rice.stanford.edu
```

There are multiple login nodes  and this will show up as rice01, rice02, etc. The nodes are load balanced so logging in to rice.stanford.edu will select a node based on current utilization. Alternatively, you can log in to a specific node. You might want to do this when you have a process running on a node already and need to monitor or kill it. If you wanted to log in to rice04:

```bash
ssh sunetid@rice04.stanford.edu
```

When you SSH, you will have to enter your password and it might prompt you to accept the server crededentials. It will also prompt you for two-factor authentication each time you log in.

Alternatively, you can install an SSH client like [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/) or [SecureCRT](https://uit.stanford.edu/software/securecrt).

## 2. Basic Unix

TODO

## 3. Stata

FarmShare has a variety of different software packages available. To list them all:

```bash
module avail
```

When you decide which program to use, you need to load the module. For Stata, this is:

```bash
module load stata-se/16
```

Now that the module is loaded, you can enter Stata with:

```bash
stata
```

This brings up the Stata interface. You can interact with the command line just as you would on your own computer. There is no GUI (graphic user interface) with this method and everything you do in Stata needs to be typed.

If you need a GUI, you need to install an [X11 server implementation](https://uit.stanford.edu/service/sharedcomputing/moreX) in Windows or on Mac. In Linux or Unix, this isn't necessary. When you SSH into rice, you need to change the command to:

```bash
ssh -X sunetid@rice.stanford.edu
```

If you are using an SSH client, view the remote/X11 options and make sure it is set up to forward X11 packets and run your X11 implementation (Xming for Windows or XQuartz for Mac). Then, instead of running `stata`, run `xstata`:

```bash
module load stata-se
xstata
```

## 4. Running batch jobs

If you want to run a Stata do file that will take a while to run, you have the option to run Stata (as described above) and run a do file in the same way you would on your home computer:

```Stata
do my_do_file.do
```

While this provides the same interactive environment that you are used to, it might be a big file that takes a long time to compute or you might have spotty internet. For this reason, you might want to consider running your program in the background as a batch job. A batch job will keep running in the background even if you get disconnected from the rice server.

There are a number of ways to run a batch job (including tmux and screen) but I've found that the easiest is using a nohup command. Assuming you've loaded Stata:

```bash
nohup krenew -t -- stata -b do my_do_file.do &
```

Note: Stata running in batch mode automatically creates a log file `my_do_file.log`.

Breaking it down:
  1. `nohup`: Don't end the process when you end the session
  2. `krenew -t`: Keep your AFS authentication alive
  3. `--`: Separates the krenew command from the stata command (otherwise krenew might think the rest of the command modifies krenew and not stata)
  4. `stata -b do my_do_file.do`: Your Stata batch job
  5. `&`: Run the command in the background

Before you exit the FarmShare session, use `ps` to view the process id and name. You should also write down which FarmShare machine you are on. The process runs on a specific machine so if you want to kill the process later, you have to log back in to the specific machine.

There are a few ways of locating your process if you haven't written it down, including with your username:

```bash
ps -u my_username
```

or with grep:

 ```bash
ps -aux | grep stata
```

Breaking it down:
  1. ps: List currently running processes
  2. u: Provide detailed information about processes
  3. a: List processes of all users
  4. x: List processes in the background
  5. |: A pipe that takes the output of the left side and uses it as input to the function on the right side (grep here)
  5. grep: search the piped in text for the keyword (stata here)
  
Once you have the process id, you can kill it and end your job early:

```bash
kill PID
```
