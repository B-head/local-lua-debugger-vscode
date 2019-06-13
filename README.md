# Local Lua Debugger for Visual Studio Code

A simple Lua debugger which requires no additional dependencies.

---
## Features
- Debug Lua using stand-alone interpretor or a custom executables
- Supports Lua versions 5.1, 5.2, 5.3 and [LuaJIT](https://luajit.org/)
- Basic debugging features (stepping, inspecting, breakpoints, etc...)
- Conditional breakpoints
- Debug coroutines as separate threads
- Basic support for source maps, such as those generated by [TypescriptToLua](https://typescripttolua.github.io/)

---
## Usage

### Lua Stand-Alone Interpreter
To debug a Lua program using a stand-alone interpreter, set `lua-local.interpreter` in your user or workspace settings:

!["lua-local.interpreter": "lua5.1"](resources/settings.png '"lua-local.interpreter": "lua5.1"')

Alternatively, you can set the interpreter and file to run in `launch.json`:
```json
{
  "configurations": [
    {
      "type": "lua-local",
      "request": "launch",
      "name": "Debug",
      "program": {
        "lua": "lua5.1",
        "file": "main.lua"
      }
    }
  ]
}
```

### Custom Lua Environment
To debug using a custom Lua executable, you must set up your `launch.json` with the name/path of the executable and any additional arguments that may be needed.
```json
{
  "configurations": [
    {
      "type": "lua-local",
      "request": "launch",
      "name": "Debug LÖVE",
      "program": {
        "command": "love"
      },
      "args": [
        "${workspaceFolder}"
      ]
    }
  ]
}
```
You must then manually start the debugger in your Lua code:
```lua
require("lldebugger").start()
```
Note that the path to `lldebugger` will automatically be appended to the `LUA_PATH` environment variable, so it can be found by Lua.

---
## Requirements & Limitations
- The Lua environment must support communication via stdio.
  - Some enviroments may require command line options to support this.
  - Ex. the Corona Simulator requires the `/no-console` flag.
- The Lua environment must be built with the `debug` library, and no other code should attempt to set debug hooks.
- You cannot manually pause debugging while the program is running.
- In Lua 5.1 and LuaJIT, the main thread cannot be accessed while stopped inside of a coroutine.
- Some custom enviroments do not support stopping on runtime errors.
  - The debugger *will* stop on explicit calls to `error()` and `assert()`.
  - To debug a runtime error, you can wrap the code with `lldebugger.call()`:
    ```lua
    require("lldebugger").call(function()
      --code causing runtime error
    end)
    ```

---
## Tips
- You can detect that the debugger extension is attached by inspecting the environment variable `LOCAL_LUA_DEBUGGER_VSCODE`. This is useful for conditionally starting the debugger in custom environments.
    ```lua
    if os.getenv("LOCAL_LUA_DEBUGGER_VSCODE") == "1" then
      require("lldebugger").start()
    end
    ```
