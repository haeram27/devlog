# VSCODE Task 설정

vscode에서 shell 명령어나 process execution을 단축 메뉴로 실행하는 설정

`Tasks: Run Task` 명령으로 실행할 수 있다.
보통 단축키로 `Shift+Alt+t`를 사용한다.

## 

## 위치

- project: {project-root}/.vscode/tasks.json
- user
  - windows: %APPDATA%\Code\User\tasks.json
  - macOS: $HOME/Library/Application Support/Code/User/tasks.json
  - linux: $HOME/.config/Code/User/tasks.json

## Template

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    // run task shortcut: Shift+Alt+T
    // tasks: configure shortcut to run shell or process command, usually build and test
    // tasks.json path: ~/.config/Code/User(Recommend), ${workspace}/.vscode
    "version": "2.0.0",
    "tasks": [
        {
            "label": "task template",
            // shell(shell command), process(default, exe, .sh)
            "type": "shell", //shell(command run with shell, shell built-in commands are available), process(default, run executable directly without shell)
            "group": "none", //task list section: build, test(unit test), none(nogroup)
            // vscode terminal pannel setting
            "presentation": {
                // open terminal pannel: always(default), silent(when error occurs), never 
                "reveal": "always",
                // terminal instance: new(everytime create), shared(existing terminal), dedicated(default, once create and reuse) 
                "panel": "dedicated",
                "focus": false, // default(false), mouse focus on terminal as soon as task run
                "clear": true, // default(false), clear previous output
                "showReuseMessage": false, // default(true), no ask for closing terminal
                "echo": false // default(true), print executing command line on terminal
            },
            "options": {
                /* current working directory to run command */
                // ${workspaceFolder} : workspace root
                // ${fileDirname} : directory of current file exists
                "cwd": "${workspaceFolder}",
                /* force shell bin to run */
                // "shell": {
                //     "executable": "bash",
                //     "args": [
                //         "-c"
                //     ]
                // }
            },
            /* override setting for linux, higher priority than the setting at task top level, windows, osx also support */
            /*
                "linux": {
                    "command": "echo",
                    "args": [
                        "this is linux"
                    ]
                },
                "windows": {
                    "command": "echo",
                    "args": [
                        "this is windows"
                    ]
                },
                "osx": {
                    "command": "echo",
                    "args": [
                        "this is osx"
                    ]
                },
            */
            /* 
                problemMatcher: match output with regex pattern, usually for build error, link:
                https://code.visualstudio.com/docs/debugtest/tasks#_defining-a-problem-matcher
            */
            "problemMatcher": [], // !! without this empty array, task will fail to run when no tsconfig.json found because default is $tsc for typescript
            "command": "echo",
            "args": [
                "task",
                "run",
                "successfully! \n",
                "shell: ${env:SHELL} \n",
                "'quote is ok'"
            ],
            "dependsOn": [ // run other tasks before this task
                "print-vscode-variables"
            ],
            "dependsOrder": "sequence" // default(parallel)
        },
        {
            "label": "print-vscode-variables",
            "type": "shell",
            "group": "none",
            "linux": {
                "command": "echo",
                "args": [
                    " \n",
                    "userHome = ${userHome} \n",
                    "workspaceFolder = ${workspaceFolder} \n",
                    "workspaceFolderBasename = ${workspaceFolderBasename} \n",
                    "file = ${file} \n",
                    "relativeFile = ${relativeFile} \n",
                    "relativeFileDirname = ${relativeFileDirname} \n",
                    "fileBasename = ${fileBasename} \n",
                    "fileBasenameNoExtension = ${fileBasenameNoExtension} \n",
                    "fileExtname = ${fileExtname} \n",
                    "fileDirname = ${fileDirname} \n",
                    "fileDirnameBasename = ${fileDirnameBasename} \n",
                    "fileWorkspaceFolderBasename = ${fileWorkspaceFolderBasename} \n",
                    "cwd = ${cwd} \n",
                    "lineNumber = ${lineNumber} \n",
                    "execPath = ${execPath} \n",
                    "defaultBuildTask = ${defaultBuildTask} \n",
                    "pathSeparator = ${pathSeparator} \n",
                    "/ = ${/} \n"
                    // "selectedText = ${selectedText} \n" // if variable is empty, the task will fail to run
                ]
            },
            "presentation": {
                "clear": true,
                "showReuseMessage": false,
                "echo": false
            },
            "problemMatcher": []
        },
        {
            "label": "chmod+x",
            "type": "shell",
            "group": "none",
            "presentation": {
                "reveal": "never",
                "clear": true,
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "command": "chmod +x ${file}"
        },
        {
            "label": "Run Free Shell Command",
            "type": "shell",
            "group": "none",
            "presentation": {
                "clear": true,
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "command": "bash",
            "args": [
                "-c",
                "${input:shellCommand}"
            ]
        },
        {
            "label": "Run Bash Script",
            "type": "shell",
            "group": "none",
            "presentation": {
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "command": "bash ${file}",
            "dependsOn": [
                "chmod+x"
            ]
        },
        {
            "label": "Gradle Test",
            "type": "process",
            "group": "test",
            "presentation": {
                "clear": true,
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "command": "gradle",
            "args": [
                "test",
                "--rerun-tasks",
                "--tests",
                "${input:gtest}"
            ]
        },
        {
            "label": "Gradle Build",
            "type": "process",
            "group": "build",
            "presentation": {
                "clear": true,
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "dependsOn": [
                "Gradle Clean",
                "Gradle Build"
            ],
            "dependsOrder": "sequence" // default(parallel)
        },
        {
            "label": "Gradle Custom Command",
            "type": "process",
            "group": "build",
            "presentation": {
                "clear": true,
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "command": "gradle",
            "args": [
                "${input:gradleTasks}"
            ]
        },
        {
            "label": "Git Status Process",
            "type": "process",
            "group": "none",
            "presentation": {
                "clear": true,
                "showReuseMessage": false
            },
            "problemMatcher": [],
            "command": "git",
            "args": [
                "status"
            ],
        }
    ],
    "inputs": [
        {
            "id": "shellCommand",
            "type": "promptString", // popup text input box
            "description": "input command",
            "default": "ls -al"
        },
        {
            "id": "gradleTasks",
            "type": "pickString", // popup select box
            "description": "select task",
            "options": [
                "bootRun",
                "assemble",
                "assemble --refresh-dependencies",
                "clean",
                "clean assemble",
                "build",
                "dependencies",
                "dependencyManagement",
                "test",
                "publishToMavenLocal",
            ],
            "default": "assemble"
        }
    ]
}
```
