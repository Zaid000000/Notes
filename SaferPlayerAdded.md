
```lua
local Players = game:GetService("Players")

local function PlayerAdded(Player: Player)

	local function CharacterAdded(character)
		
	end	
	CharacterAdded(Player.Character or Player.CharacterAdded:Wait())
	Player.CharacterAdded:Connect(CharacterAdded)
end

Players.PlayerAdded:Connect(PlayerAdded)
for _,player in pairs(Players:GetPlayers()) do
	task.spawn(function() PlayerAdded(player) end)() -- Consider Coroutine.Warp
end
```
