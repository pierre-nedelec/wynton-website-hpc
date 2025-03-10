<div class="alert alert-warning" role="alert" style="margin-top: 3ex" markdown="1">
⚠️ 2023-04-04: If you get an error "**It looks like the RStudio Server failed during launch ...**" when you run `rsc start`, please [see below for a workaround](#known-issue-under-investigation-since-2023-03-30). It's a problem that affects several users, but not everyone.
</div>

# Work with RStudio

R is available on {{ site.cluster.name }} via a [contributed environment module](/hpc/software/software-repositories.html).  It can be run interactively in the terminal via `R` on a development node, as explain on the how-to '[Work with R]' page.  To run R via the RStudio IDE, there are two options:

1. The **RStudio Server** via the web browser (recommended)

2. The **RStudio Desktop** over X11 Forwarding

Running desktop applications over X11 forwarding works, but depending on your network connection, you will experience more or less lag. For instance, you might see a small delay between a key is pressed and it is echoed at the R prompt. Similarly, plotted graphics might take some time to render. This is also true for the RStudio Desktop. Because of this, we recommend running a personal RStudio Server instance, because that is designed to run remotely.

Regardless which one you use, you will be running the same version of R as you would when running from the terminal and all your installed packages will be available in either.


## Alt 1. Personal RStudio Server

You can run a personal RStudio Server instance, which you, and only you, can access via your local web browser.  Contrary to RStudio Desktop, this approach does not require X11 forwarding. Instead, it does require running _two_ separate SSH connections to the cluster: (i) one to launch RStudio Server, and (ii) one to connect to it.

### Step 1. Launch your own RStudio Server instance

Assuming you are already logged on to a development node, launch your personal RStudio Server instance as:

```sh
[alice@{{ site.devel.name }} ~]$ module load CBI rstudio-server-controller
[alice@{{ site.devel.name }} ~]$ rsc start
alice, your personal RStudio Server 2023.03.0+386 running R 4.2.2 is available on:

  <http://127.0.0.1:20612>

Importantly, if you are running from a remote machine without direct access
to {{ site.devel.hostname }}, you need to set up SSH port forwarding first, which you can do by
running:

  ssh -L 20612:{{ site.devel.hostname }}:20612 alice@{{ site.login.hostname }}

in a second terminal from you local computer.

Any R session started times out after being idle for 120 minutes.
WARNING: You now have 10 minutes, until 2023-03-31 11:04:13-07:00, to
connect and log in to the RStudio Server before everything times out.
Your one-time random password for RStudio Server is: y+IWo7rfl7Z7MRCPI3Z4
```

There are two things you should pay extra attention to here:

1. The one-time random password that was generated

2. The instructions how to log in to the cluster with SSH port forwarding

You will need both below.


#### Known issue under investigation (since 2023-03-30)

Some users report they have problems when they use latest RStudio
Server 2023.03.0+386 (default) that was installed 2023-03-29.  If you
get something like:

```sh
[alice@{{ site.devel.name }} ~]$ rsc start
...
ERROR Unexpected exception: File name too long [system:36]; LOGGED FROM: int main(int, char* const*) src/cpp/server/ServerMain.cpp:941
ERROR: It looks like the RStudio Server failed during launch ['rserver' process (PID 29050 on {{ site.devel.name }}) no longer exists]. This happened with rsc 0.13.4, RStudio Server 2023.03.0+386 (Cherry Blossom) for Linux [/wynton/home/cbi/shared/software/CBI/rstudio-server-2023.03.0-386/bin/rstudio-server], and R 4.2.3 (2023-03-15) -- "Shortstop Beagle" [/wynton/home/cbi/shared/software/CBI/R-4.2.3-gcc10/bin/R]. Post-mortem analysis: The RStudio Server Controller configuration folder (/wynton/protected/home/boblab/alice/.config/rsc) has 48 characters. A configuration folder with more than 40 characters will likely not work with RStudio Server version after 2022.02.4-500. Here we are using version 2023.03.0+386. Try setting 'export _RSC_RSERVER_DATA_DIR_="$(mktemp -d)"' before calling 'rsc start' as a workaround.
Traceback:
1: assert_rserver_running() on line #804 in /wynton/home/cbi/shared/software/CBI/rstudio-server-controller-devel/bin/rsc
2: main() on line #1653 in /wynton/home/cbi/shared/software/CBI/rstudio-server-controller-devel/bin/rsc
Exiting (exit 1)
Shutting down RStudio Server ...
Shutting down RStudio Server ... done
[alice@{{ site.devel.name }} ~]$ 
```

then you are also affected by this bug.  A _workaround_ is to set environment variable `_RSC_RSERVER_DATA_DIR_` (note the leading and trailing underscores) as follows before calling `rsc start`:

```sh
[alice@{{ site.devel.name }} ~]$ module load CBI rstudio-server-controller
[alice@{{ site.devel.name }} ~]$ export _RSC_RSERVER_DATA_DIR_="$(mktemp -d)"
[alice@{{ site.devel.name }} ~]$ rsc start
```

_Last updated: 2023-04-18_


### Step 2. Connect to your personal RStudio Server instance

On your local computer, log into the cluster in a second terminal
following the instruction above.  Make sure to use your own username
and make sure to use the port number (e.g. `20612`) that
was assigned to you.

```sh
{local}$ ssh -L 20612:{{ site.devel.hostname}}:20612 alice@{{ site.login.hostname }}
alice1@{{site.login.hostname }}:s password: XXXXXXXXXXXXXXXXXXX
[alice@{{site.login.name }} ~]$ 
```

### Step 3. Open RStudio Server in your local web browser

If you successfully completed the above two steps, and you made sure to use the correct port, then you should be able to open your personal RStudio Server in your local web browser by going to:

* <http://127.0.0.1:20612/> (note, your port is different)

You will be presented with a 'Sign in to RStudio' web page where you need to enter:

1. Your cluster username (e.g. `alice`)
2. The one-time random password displayed in Step 1 (e.g. `y+IWo7rfl7Z7MRCPI3Z4`)

After clicking 'Sign In', you should be redirected to the RStudio interface.


To terminate the RStudio Server, start by exiting R by typing `quit()` at the R prompt. Then press <kbd>Ctrl-C</kbd> in the terminal where you called `rsc start`.  Alternatively, run `rsc stop` in another terminal, e.g. the second one used in Step 2.




## Alt 2. RStudio Desktop over X11 Forwarding

<div class="alert alert-warning" role="alert" markdown="1">
It is _not_ possible to run RStudio Desktop via X2Go, because X2Go does not support the modern graphical protocols that RStudio Desktop requires.
</div>

To run the RStudio Desktop client, you must be logged via [SSH with X11 forwarding enabled]. Then load the `rstudio` (Desktop) module from the CBI stack and launch `rstudio`:

```sh
[alice@{{ site.devel.name }} ~]$ module load CBI rstudio
[alice@{{ site.devel.name }} ~]$ rstudio
TTY detected. Printing informational message about logging configuration.
Logging configuration loaded from '/etc/rstudio/logging.conf'. Logging to
'{{ site.user.home }}/.local/share/rstudio/log/rdesktop.log'.
```

You should see the RStudio Desktop interface appear after a few seconds.
If you get a blank window, which some macOS users have reported, retry with:

```sh
[alice@{{ site.devel.name }} ~]$ QMLSCENE_DEVICE=softwarecontext rstudio
```

To exit the RStudio Desktop, type `quit()` at the R prompt, or press <kbd>Ctrl-C</kbd> in the above terminal.


[CBI software stack]: /hpc/software/software-repositories.html
[Work with R]: /hpc/howto/r.html
[SSH with X11 forwarding enabled]: /hpc/howto/gui-x11fwd.html#x11-forwarding-over-ssh
