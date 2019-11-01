# ozy

`ozy` is a native Python program that makes it easy for you and your team to share and colloborate on commonly used programs such as Hashicorp's `vault`, `python`, etc. 

## Getting Started
First, [download ozy](convenient download link here). This will cause `ozy` to install itself details details mking `~/.ozy`, installing itself and the `.ozyconf` file there, and adding that directory to `$PATH`. 

Next, you need to initialize `ozy` with a url which contains your team's blessed set of apps:

```bash
$ ozy init https://github.com/myteam/ozy/.ozyconf
``` 

This will cause `ozy` to:
1) Downloads the `.ozyconf`
2) Store it in `$USER/.ozyconf`
3) 

Note, `ozy` will not install the apps listed in the conf file until you use them!

## Running a command
Assume that your `.ozyconf` has Hashicorp's [nomad](nomad url) in it.

```bash
$ nomad --version
ozy is installing version 0.0.10 of nomad from $URL..
Nomad v0.9.4 (a81aa846a45fb8248551b12616287cb57c418cd6)
```

## Running a command through ozy
```bash
$ ozy run nomad 0.9.4 
```

##  Getting info on supported commands
```bash
$ ozy info
ozy cache dir path
ozy conf file path
Support things:
   * nomad $VERSION 
   * overridden vault $VERSION overriden by /path/to/ozy/file
```
## Removing a command
```bash
$ ozy rm nomad
```
The real nomad lives in `~/.cache/ozy/nomad/0.9.4/nomad`. The simlink in `~/.ozy/bin/nomad` points to ozy. `rm` will remove the simlink, which is as good as deleting the executable.  

## Full cleanup
```bash
$ ozy clean
```
This will remove `~/.cache/ozy`! 

## Updating
```bash
$ ozy update
```
This will cause ozy to re-fetch your team's ozy config url, and then update any apps that it finds.
Note that you can do this:
```bash
$ ozy update --dry-run 
```
Rather than upgrade the apps, it will tell you what it would upgrade. 

## Pinning Versions
Individual projects can chose to pin versions of apps by creating a .ozy file in the directory that they are in. When you run ozy in that directory, ozy will use the versions listed in that file in preference to the system wide ozy found in `~/.ozy/.ozyconf`



# Matt's original write-up

Overview
We need to install a bunch of apps, some external, some our own. We want a nice way to express which apps are installed, where, and with which versions. Specifically:

Hashicorp tools: nomad, terraform, vault
Concourse tools: fly
Internal tools: lake, yard
Additionally some projects might need different versions, so the tools should adapt from whence they’re called.

## Goals
Be super super simple to install and initially set up. Ideally something like `bash $(curl https://ozzy.aq.tc/setup)`.
Appear to be the tool as wrapped, so if invoked as `fly` will lookup, download, install and then run the right version of fly. (leininger style)
Allow for a `.ozzy` file in directories that determines what versions of tools are required when running from a directory (or subdirectory therein). If not found, look in ~/.ozzy
Have default versions (fixed on startup) in ~/.ozzy. These versions can be updated as `ozzy update`
Support many different types of tools, with tool installation types pulled from a remote source so we can quickly add new types. Example types:
Hashicorp: just a binary in a URL location (where location is templated on version, OS and not much else)
Fly: similarly , just an executable grabbed from concourse. Versions may be tricksie
Miniconda: a shell script to run with a few magic params to put it in the “right” place. Some complications due to it wanting to source’d into various things...maybe not a priority?
Yard, lake. Each a conda package which can be installed into a virtual env with the right magic, and then executed via path-to-the-virtualenv/bin/python yard etc

## Design

### Sketch:
Python3 no-deps, single file “ozzy”
On startup, if run as “ozzy install” or just as “ozzy” with no params:
Look for ~/.ozzy, if not there go for an install:
Grab github.aq.tc/some/known/master/path/ozzy and put in ~/.ozzy
Make ~/bin if not there already. Warn if ~/bin not on path and offer tips on how to make it so (maybe ~/.ozzy-bin if we don’t want to push that on folks)
Copy yourself to bin dir as ozzy
For all things in ~/.ozzy; symlink ~/bin/<tool> to ~/bin/ozzy
If argv[0] is not “ozzy”, then we’re pretending to be tool argv[0]:
Walk from chdir to root of FS looking for ~/.ozzy. Load as json. Keep going to root, adding more info (deepest ozzy for tool X wins though). Finally add ~/.ozzy (if not already found)
Look for `tools: argv[0]` in json file. It should have a version, an installer type, and maybe some installer-specific info.
Look in ~/.ozzy-bin/tool/version or similar to see if we already have the tool
If not, install it (specific to installer type and installer-specific info).
Exec and replace the binary with ~/.ozzy-bin/tool/version/tool and the remaining arguments
If argv[0] is ozzy, have some ozzy specific options and facilities, like “update”, “help”, “list tools” etc. Also support an “ozzy tool> -- <tool args” which is the same as invoking ozzy with argv[0] as <tool>”
Things to consider:
Ozzy cleanup
Ozzy locking: prevent multiple invocations from trying to install at the same time (flock on the directory)
Locations? ~/.cache/ozzy/ is probably best place for things actually
Error messaging, reporting (pinging stats to an endpoint?)
OPEN SOURCE ALL THE THINGS
