# Remote Debugging Raspberry PI's C/C++ Applications with VSCode

Demostrates using VSCode to remote launch and debug application on Raspberry Pi.

In this demo, it assumes the source code is only avaiable and get compiled on host side, and you don't
need to sync them to Raspberry Pi.  If the source code is on the target (Raspberry Pi), 
you can refer to [The Useful RaspberryPi Cross Compile Guide](https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187).

## Prerequisite

## Host

* [Visual Studio Code](https://code.visualstudio.com/download)
* GDB
* RPI cross compiler.  
    - Docker based [sdt/docker-raspberry-pi-cross-compiler](https://github.com/sdt/docker-raspberry-pi-cross-compiler) or
    - Get the gcc-linaro-arm-linux-gnueabihf-raspbian-x64 toolchain from [raspberrypi/tools](https://github.com/raspberrypi/tools)

## Raspberry

* Raspberry Pi board (with network interface).
* Install GDB
* Install GDB server
* Enable SSH on Raspberry Pi.

## Compilation

In this example, it use "rpxc" to compile hello.c and push the executable "hello" to folder "~/program" of Raspberry Pi.

* OSX/Linux

```bash
    rpxc /rpxc/bin/arm-linux-gnueabihf-gcc -g -o ./build/hello ./src/hello.c
    rsync -zvh ./build/hello pi@rpi3:~/program
```

* Windows

TBD

## Remote GDB

* OSX/Linux

.vscode/launch.json

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Remote GDB",
            "type": "cppdbg",
            "request": "launch",
            "program": "/home/pi/program/hello",
            "args": [],
            "stopAtEntry": true,
            "cwd": "/home/pi/program",
            "environment": [],
            "externalConsole": true,
            "pipeTransport": {
                "pipeCwd": "/usr/bin",
                "pipeProgram": "/usr/bin/ssh",
                "pipeArgs": [
                    "pi@rpi3"
                ],
                "debuggerPath": "/usr/bin/gdb"
            },
            "sourceFileMap": {
                // "remote": "local"
                "/build/": "${workspaceFolder}"
            },
            //"logging": { "engineLogging": true, "trace": true, "traceResponse": true },
            "MIMode": "gdb"
        },
    ]
}
```

* Windows

## Remote GDB Server

.vscode/launch.jon

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Remote GDB Server",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceRoot}/build/hello",
            "miDebuggerServerAddress": "localhost:9091",
            "args": [],
            "stopAtEntry": true,
            "cwd": "${workspaceRoot}",
            "environment": [],
            "externalConsole": true,
            // without preLaunchTask, on HOST run: ssh -L9091:localhost:9091 pi@rpi3  gdbserver :9091 ~/program/hello
            "preLaunchTask": "gdbserver",
            //"serverStarted": "Listening on port",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            //"logging": { "engineLogging": true, "trace": true, "traceResponse": true },
            "MIMode": "gdb",
        },
    ]
}
```

.vscode/tasks.jon

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "gdbserver",
            "type": "shell",
            "command": "ssh -L9091:localhost:9091 pi@rpi3 'gdbserver :9091 ~/program/hello'",
            "isBackground": true,
            //"identifier": "gdbserver",
            //"promptOnClose": false,
            //"isShellCommand": true, <-- replace by type = shell
            //"problemMatcher": []
        }
    ]
}
```

![debug any way](images/debug-anyway.png)

* OSX/Linux

* Windows


## Properties of launch.json

Name | Description 
---- | -----
type | cppdbg, gdb
request
program | 
environment |
externalConsole |
additionalSOLibSearchPath | 
setupCommands | 
sourceFileMap |
autorun |
logging | 

## Reference

- [How to Debug C/C++ with VSCode](https://github.com/74th/vscode-debug-specs/tree/master/cpp)
- [Pipe Transport](https://github.com/Microsoft/vscode-cpptools/blob/master/Documentation/Debugger/gdb/PipeTransport.md)
- [Configuring launch.json for C/C++ debugging](https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md)
- [The Useful RaspberryPi Cross Compile Guide](https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187)
- [VSCdoe "type":"cppdbg" or "type":"gdb"](https://github.com/JelleRoets/cross-compile-edison#setting-up-the-debugger)

