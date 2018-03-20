Delve is one of the largest used tools for debugging go applications.

Check Folowing link for more information about delve:
[Info about Delve](https://github.com/derekparker/delve)


## Getting Started

Consider the following project structure:
```
.
├── github.com/me/foo
├── cmd
│   └── foo
│       └── main.go
├── pkg
│   └── baz
│       ├── bar.go
│       └── bar_test.go
```
Now, move into the directory:
```
$ cd github.com/me/foo/cmd/foo
``` 
Now, simply run ```dlv debug```. It will compile and prepare the program for debugging.

Then execute ```break main.main```. This will set a break point at main function.

## Commands

- continue: Runs the program to Next break point. If no break point exists, it will run the program to termination. 

- break: setup a breakpoint. Usage 
```
break foo.main:21        // packagename.method:linenumber
```
- next: Move to next line.

- step: Move inside a func.

- stepout: Move out of func.

- list: shows source code.

- print: prints value of the specified variable.Usage:
```
print X //X is name of variable.
```
- restart: restarts the debugging process. It preserves previously set breakpoints.

- exit: Exits the debugger.

- goroutines: list all goroutines.












using delve to debug API's.????