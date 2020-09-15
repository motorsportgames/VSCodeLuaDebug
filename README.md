# LuaMachine Unreal Engine debugging
## Requirements
* Visual Studio 2019 or 2017
* Visual Studio Code
* [Visual Studio Code Extension](https://marketplace.visualstudio.com/items?itemName=devCAT.lua-debug)
* [Luasocket core, compiled as a dynamic library](#luasocket)
* [LuaMachine for Unreal Engine](https://github.com/rdeioris/LuaMachine)

## Setup
### Visual Studio Code
1. Install the Lua Debugger extension by devcat, you can easily do this by searching within VSCode's Extension manager for `publisher:"devCAT"` and installing the `Lua Debugger` you find.
2. For best results, open the root Lua directory of your game project as the root of your VSCode project.
3. Lua Debugger

### Luasocket
Luasocket is a C-based extension library for Lua that needs to be compiled. 
[https://github.com/diegonehab/luasocket](https://github.com/diegonehab/luasocket)

#### Windows
1. Clone the *luasocket* repo into your `Plugins/LuaMachine/Source/ThirdParty` directory, so it may look like `Plugins/LuaMachine/Source/ThirdParty/luasocket`
2. Open an x64 Native Tools Command Prompt, which is available from your Visual Studio installation
3. Navigate to your `luasocket` directory in your terminal.
4. Build luasocket by running these commands
   
    VS2017
    ```
    msbuild socket.vcxproj -p:Configuration=Release -p:Platform=x64 -p:LUALIBNAME=liblua53_win64.lib -p:LUAINC=../lua -p:LUALIB=../x64
    ```
    VS2019
    ```
    msbuild socket.vcxproj -p:Configuration=Release -p:Platform=x64 -p:PlatformToolset=v142 -p:LUALIBNAME=liblua53_win64.lib -p:LUAINC=../lua -p:LUALIB=../x64
    ```
5. Close the terminal
6. Create a new directory called `socket` inside of your UnrealEngine Binaries folder. E.g.: `UnrealEngine\Engine\Binaries\Win64\socket` and then copy your `luasocket/x64/Release/socket/core.dll` into that new folder. core.dll will be loaded at runtime when you require it in your lua code.
7. Copy all of the .lua files from `luasocket/src` into your Project's root lua directory (this may be your `Content` folder, or it may be something else depending on your settings)

#### macOS
  *not tested*
1. WIP, but the concept is the same as above. Make your core.dylib available to the UE binaries in a socket folder.

#### Linux
*not tested*
1. WIP, but the concept is the same as above. Make your core.so available to the UE binaries in a socket folder.

### Lua Debugger
1. From this repo, grab [dkjson.lua](https://github.com/schetle/VSCodeLuaDebug/blob/master/debuggee/dkjson.lua) and [vscode-debuggee.lua](https://github.com/schetle/VSCodeLuaDebug/blob/master/debuggee/vscode-debuggee.lua), and copy them into your project's root lua directory (again, it may be your `Content` folder, or something else depending on what you've setup)
2. 

### LuaMachine
1. In your LuaState, ensure these are disabled: `Enable Line Hook`, `Enable Call Hook`, `Enable Return Hook`. 
2. Setup your LuaState to run a CodeAsset or Lua file. The contents of this file can be anything, but somewhere you'll want to initialize the debugger with this code:
    ```
    local json = require 'dkjson'
    local debuggee = require 'vscode-debuggee'
    local debuggeeConfig = {
        luaMachineRoot = '<your lua scripts root here>'
    }

    local startResult, breakerType = debuggee.start(json, debuggeeConfig)
    print('debuggee start result: ', startResult, breakerType)
    ```
   **note**: replace `<your lua scripts root here>` with the root of your lua scripts. For example, if you're using Content/Scripts as the root for all your lua scripts, supply `'Scripts'`, if you're just using your entire `Content` directory as the root of your lua scripts then just supply `''`
3. Because of lazy loading, you'll probably want to initialize your LuaState's code early. I do this in my GameInstance so that this is initialized extremely early: `FLuaMachineModule::Get().GetLuaState(LuaState, GetWorld());`, you can do this also in blueprint in your gamemode's begin play, but your debugging session won't happen until it hits this code.

## Running
1. With your lua scripts directory as the root of your VSCode project, run the `wait` configuration. This will constantly wait for a connection from your Unreal Project, and it will automatically restart when you exit Play.
2. Set some breakpoints
3. Play in UE editor.
4. Your breakpoints should get hit.


**note:** if for some reason you don't have the `wait` configuration, ensure you have the devcat Lua Debugger installed. You can put this in your launch.json, but it should already be there from the Lua Debugger extension.
```
{
    "name": "wait",
    "type": "lua",
    "request": "attach",
    "workingDirectory": "${workspaceRoot}",
    "sourceBasePath": "${workspaceRoot}",
    "listenPublicly": false,
    "listenPort": 56789,
    "encoding": "UTF-8"
},
```