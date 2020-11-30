
# Dependency Resolver

`dep` is a command line application that takes a list of dependency declarations
as an input and computes the transitive dependencies of tasks.


## File Format

The direct dependencies are defined in a user-provided file of the following
format:

```
A B C
B C E
C G
D A F
E F
F H
```

Each line starts with a task, followed by a list of tasks that it depends on,
separated by spaces.

In the example above, `A` depends on `B` and `C`.
`B` depends on `C` and `E` etc.


## Getting Started

Once the dependencies are imported, the user can then interactively query for
tasks they are interested in. The program will print all transitive dependencies
of the task.

For example, given the above input file, if the user asks for the transitive
dependencies of `A`, the program should report `B`, `C`, `E`, `F`, `G`, `H`.


### Usage

The application supports some command line parameters to:

- Specify the file to import
- Print version number
- Show a help message

```
Usage: dep [-hV] [-f=<file>] [@<filename>...]
      [@<filename>...]   One or more argument files containing options.
  -f, --file=<file>      Input file
  -h, --help             Show this help message and exit.
  -V, --version          Print version information and exit.
```

If no import file is provided, the user may still add dependencies and query
them in interactive mode.


### Interactive Mode

Interactive mode kicks off after importing the dependency declarations. It
allows the user to manipulate and query the current dependencies. This mode will
wait for user commands and then execute requested actions:

These are the available commands:

- `HELP`: Show available commands.
- `QUIT`: Exit interactive mode.
- `LIST`: Show all current defined dependencies.
- `ADD`: Add dependencies to a task:
- `REMOVE`: Remove all dependencies of a task.
- `RESOLVE`: Calculate all dependencies of a task.


#### Adding Dependencies

Command `ADD` will create the specified task if it doesn't exist yet and then
will append any other specified tasks as direct dependencies:

```
ADD A [B C D...]
```

If the task is already defined and it already depends on the specified tasks
then this command will have no effect.

Examples:

- `ADD Z`: Create task `Z` if it doesn't exist; no additional effect.
- `ADD X Y Z`: Append dependencies `Y` and `Z` to existing dependencies of `X`.


#### Removing Dependencies

Command `REMOVE` will remove all dependencies for the specified task. Multiple
tasks can be optionally specified:

```
REMOVE A [B C D...]
```

If the task is not defined then this command will have no effect.

Examples:

- `REMOVE Z`: Remove all dependencies of task `Z`.
- `REMOVE A B`: Remove all dependencies of tasks `A` and `B`.


#### Resolving Dependencies

Command `RESOLVE` will calculate all transitive dependencies for the specified
task. Multiple tasks can be optionally specified:

```
RESOLVE P [Q R S...]
```

If a task is not defined but it is declared as a dependency of some other task,
then it will resolve as a task with no dependencies itself. But if a task is not
defined and is not a dependency of any other tasks, then it will be resolved as
an unknown task.

If a task resolves to a cyclic dependency, then `dep` will report it to the user
in addition to the resolved dependencies.

Examples:

- `RESOLVE A`: Resolve all dependencies of task `A`.
- `RESOLVE A B C`: Resolve all dependencies of task `A`, `B` and `C`.

### Example

```
dep version 0.0.1

Importing dependencies from file: "C:\etc\gradle\dep\test.dep"...
6 dependencies imported

Entering interactive mode...
 - Enter QUIT to exit; HELP to see all available commands

Please enter a command:
 > list

There are 6 dependencies:
 - Task "A" depends on: [B, C]
 - Task "B" depends on: [C, E]
 - Task "C" depends on: [G]
 - Task "D" depends on: [A, F]
 - Task "E" depends on: [F]
 - Task "F" depends on: [H]

Please enter a command:
 > resolve A

Resolving task "A"...
Result: Task "A" depends on: [B, C, E, F, G, H]

Please enter a command:
 > remove B

Removing dependencies from task "B"...
Result: Task was removed

Please enter a command:
 > remove C

Removing dependencies from task "C"...
Result: Task was removed

Please enter a command:
 > add A F

Adding dependencies to task "A" -> [F]...
Result: Task "A" depends on: [F, B, C]

Please enter a command:
 > resolve A

Resolving task "A"...
Result: Task "A" depends on: [B, C, F, H]

Please enter a command:
 > quit

Exiting...
Interactive mode exited
```
