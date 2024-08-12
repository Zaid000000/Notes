A Utility module for quick check of a places networking, do note that the RemoteFunctions argument checker could break a game because they return nil. #Utility #Module
``` lua
local RunService = game:GetService("RunService")

local IS_SERVER = RunService:IsServer()
local IS_CLIENT = RunService:IsClient()


local function ServerSideRemote(Remote)
	if IS_CLIENT then
		return
	end
	Remote.OnServerEvent:Connect(function(Player, ...)
		local args = {...}
		warn("[NetworkHook]: %s" .. Player.Name .. " fired with args: " ,args)
	end)
end

local function ClientSideRemote(Remote)
	if IS_SERVER then
		return
	end
	Remote.OnClientEvent:Connect(function(...)
		local args = {...}
		warn("[NetworkHook]: %s" .. "Server" .. " fired with args: " ,args)
	end)
end

local function HookRemoteEvents(Remote)
	ClientSideRemote(Remote)
	ServerSideRemote(Remote)
end

local function ServerSideRemoteFucntion(Remote)
	if IS_CLIENT then
		return
	end
	Remote.OnServerInvoke = function(...)
		local args = {...}
		warn("[NetworkHook]: %s" .. "Server" .. " fired with args: " ,args)
		return nil
	end
end

local function ClientSideRemoteFucntion(Remote)
	if IS_SERVER then
		return
	end
	Remote.OnClientInvoke = function(Player, ...)
		local args = {...}
		warn("[NetworkHook]: %s".. Player.Name .. " invoked with args: " ,args)
		return nil
	end
end

local function HookRemoteFunctions(Remote)
	ServerSideRemoteFucntion(Remote)
	ClientSideRemoteFucntion(Remote)
end

local RemotesHook = {
	["RemoteEvent"] = HookRemoteEvents,
	["UnreliableRemoteEvent"] = HookRemoteEvents,
	["RemoteFunction"] = HookRemoteFunctions,
}
local function HookRemotes(RemotesPlace: Instance)
	for i, Remote in RemotesPlace:GetDescendants() do
		local Hook = RemotesHook[Remote.ClassName]
		if Hook then
			Hook(Remote)
		end
	end
end


return HookRemotes
```

### How To Use:

*First Create a module in ReplicatedStorage and insert the code block above.*
 
Server:
``` lua
local NetWorkHook = require(game:GetService("ReplicatedStorage").NetworkHook)(game:GetService("ReplicatedStorage"))

```

Client
``` lua
local NetWorkHook = require(game:GetService("ReplicatedStorage").NetworkHook)(game:GetService("ReplicatedStorage"))

```