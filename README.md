# Execute


## Overview 

`Execute` provides two methods:
 
* `Application` lets you spawn a new Windows process in order to run an application. 

* `Process` lets you spawn a new process and return its standard output when it's finished.

Use `Application` in order to run a program or a batch file when you are not in need for its output. A typical example is firing up another instance of Dyalog APL. Use `Process` if you need the output of a certain program. A typical example would be the !SubVersion "svn list" command.

Naturally `Process` waits for the program to quit. `Application` doesn't wait by default but you can change that default behaviour. If you do you will get the exit code of that application, if there is any. A return code can be returned by a Dyalog app with

```
      ⎕OFF 123
```

and in a batch file with

```
exit 123
```

## Details 

Imagine you want run a ```foo.exe``` which returns a 0 in case of success and a positive integer in case of an error.

Now in order to kick off an instance of `foo.exe` one could simply execute:

```
      ⎕CMD 'foo.exe'
```

So why should someone bother to use `Execute` instead? Well, there are a couple of good reasons:

* You won't be able to catch the return code from `⎕CMD`.

* Although you can specify "hidden" as additional parameter an app executed this way will often pop up a window anway.

* Firing up  another instance of Dyalog with  `⎕CMD` can result in some strange effects.

The class `Execute` is designed to get you around these obstacles.

## Examples: the "Application" method 

All examples assume that you have a variable `path` in your workspace which points to the Dyalog EXE. A typical path would be:

```
      path
"C:\Program Files (x86)\Dyalog\Dyalog APL 12.1 Unicode\dyalog.exe"
```

### Start a program and wait until it exits

This can be achieved with the default settings of the parameters you might specify:

```
       res←Execute.Application path
```

This starts another Dyalog APL. The method `Execute.Application` waits until the started Dyalog APL exits.

```
      ⎕←Display'rc' 'processInfo' 'result' 'more',[⎕IO+0.1]res
┌→────────────────────────────────────┐
↓ ┌→─┐                                │
│ │rc│          0                     │
│ └──┘                                │
│ ┌→──────────┐ ┌→──────────────────┐ │
│ │processInfo│ │1228 1592 3292 6728│ │
│ └───────────┘ └~──────────────────┘ │
│ ┌→─────┐                            │
│ │result│      245                   │
│ └──────┘                            │
│ ┌→───┐        ┌⊖┐                   │
│ │more│        │ │                   │
│ └────┘        └─┘                   │
└∊────────────────────────────────────┘
```

As you can see the "result" returned by the called application is 245. This is achieved by executing

```
      ⎕OFF 245
```

in that application.

### Start another program but don't wait for it

In order to achieve that we need to change one of the defaults. First create a namespace with all parameters and their default settings:

```
      cs←Execute.DefaultParms
```

This creates a namespace with all parameters you may change. You can list the parameters available by calling the namespace's `List` method:

```
      cs.List
 timeoutAfter  0
 dir            
 hidden        0   
 wait          1   
```

Note that `timeoutAfter←0` means no timeout at all while any positive integer is treated as number of seconds.

Now make your changes:

```
      cs.wait←0
      cs.hidden←1
      cs.timeoutAfter←10
```

Now run the application:

```
    cs Execute.Application path,'  dlgtest.dws'
```

This starts Dyalog which then in turn loads the workspace `DLGTEST`. Although the application writes to the session you won't see the session because of `hidden` being 1. Note that this means that if the app crashes for any reason the user has no other means to finish the application than killing the process in the task manager.


## Examples: the "Process" method 

The `Process` method is designed to return whatever the called program is going to write to stdout. For that reason `Process` always waits until "program" exits. The application returns three items:

1. An indicator whether "program" could be started at all. 0 is okay while 1 means failure.
1. Everything that was written to stdout by "program". This is either a simple string of a vector of strings.
1. The exit code of the pogram called. If there is no exit code this is -1.


### Simple right argument

The following example would work in case the following preconditions are full-filled:

 * You have a !SubVersion client installed on your machine.
 * You have access to the Internet.
 * The path to the !SubVersion executable is contained on the Windows environment variable "PATH".

```
      path←'aplwiki.com/os/dyalog/IniFiles/branches/'
      cmd←'svn list svn://',path
      ↑(1+⎕io)⊃#.Execute.Application cmd
RB-1.6/  
RB-1.6.2/
RB-1.6.3/
RB-1.7.0/
RB-1.7.1/
```

### Nested right argument

Rather then specifying a simple right argument you can also pass a vector of strings. The first one is used in order to identify the program to be executed while the other items are used as input.

The following example works although it's by no means suggested to do this: The WinFile class offers a much better way so solve this. For example, if there are any non-ANSII-characters used in a file- or directory name, than you won't see them: they will show up as question marks. However, to demonstrate the usage of a vector of strings the example is fine.

Our goal is to start a command.com instance, execute the dir command and exit the command.com instance while returning an exit code 9:

```
      cmd←'cmd' 'dir' 'exit 9'
      ↑(1+⎕io)⊃#.Execute.Application cmd
Microsoft Windows [Version 6.1.7600]                           
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

....                              
```

Note that you cannot specify input for a Dyalog session this way.

## Acknowledgement 

Many thanks to Peter-Michael Hager without whom this class wouldn't exist.