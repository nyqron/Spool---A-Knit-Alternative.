# Spool - A-Knit-Alternative.

A modern ROBLOX game framework combining services, controllers and built in data persistence, made as an alternative for the now deprecated Knit.

## Why use spool?

**Knit is deprecated.** The Roblox ecosystem has evolved, and game architecture patterns have improved. Spool learns from Knit's strengths while solving real problems:

- **Distributed data safety** - session locking prevents multiple servers from corrupting player data
- **Auto persistence** - built in autosave with intelligent spreading (no data store throttling)
- **Clean Networking** - signals, unreliable signals and functions with built in rate limiting and validation
- **Graceful shutdown** - saves all data before server closes, with automatic retry and timeout handling
- **Modern architecture** — Services on server, controllers on client, clean separation of concerns

## Installation

1. Get the latest release from [Releases](https://github.com/nyqron/Spool---A-Knit-Alternative./releases)
2. Place the Spool folder in ReplicatedStorage
3. Require it in your main script:

```lua
local Spool = require(game.ReplicatedStorage.Spool)
```

## Loading and creating modules

**Loading and creating services**

First off you need to make a Services folder in ServerScriptService, put in your modules, ie a leaderstats module.
*Make sure the Services folder is in ServerScriptService.*

Now you have done that, make a new script in ServerScriptService called something like server.init, and add this inside:

```lua
local Spool = require(game.ReplicatedStorage.Spool)

Spool.AddServices(game.ServerScriptService.Services)

Spool.StartServer()

print("[SPOOL] Server loaded!")
```

You have now started the server! All the modules inside this Folder will now load. Let us move on.

Here is a simple module to put into Services to test it out!
Make a new module called TestService, and inside add:

```lua
local Spool = require(game.ReplicatedStorage.Spool)

local TestService = Spool.CreateService({Name = "TestService"})

function TestService:Start()
	print("Tested Spool on the server!")
end

return TestService
```

Press play and check the output and you shall see a nice print with: "Tested Spool on the server!"

**Loading and creating controllers**

Controllers are client-side modules that run on each player's client. Make a `Controllers` folder in `StarterPlayerScripts`.

Create a new LocalScript in `StarterPlayerScripts` called `client.init` and add:

```lua
local Spool = require(game.ReplicatedStorage.Spool)

Spool.AddControllers(script.Parent)

Spool.StartClient()

print("[SPOOL] Client loaded!")
```

All ModuleScripts inside the Controllers folder will now load automatically.

Here is a simple controller to test it out! Make a new ModuleScript in the Controllers folder called TestController, and inside add:

```lua
local Spool = require(game.ReplicatedStorage.Spool)

local TestController = Spool.CreateController({Name = "TestController"})

function TestController:Start()
	print("Tested Spool on the client!")
end

return TestController
```

Press play and check the output and you shall see a nice print with: "Tested Spool on the client!"

## Creating Data Stores

**Data stores allow you to save player data persistently.**

Create a data store in your service's `Init` function:

```lua
local Spool = require(game.ReplicatedStorage.Spool)

local MyService = Spool.CreateService({Name = "MyService"})

function MyService:Init()
	self.Store = Spool.CreateStore("PlayerData", {
		Level = 1,
		Gold = 0,
		Inventory = {}
	})
end

return MyService
```

The table you pass is the **template** — default values for new players.

**Loading player profiles**

When a player joins, load their data:

```lua
function MyService:Start()
	game.Players.PlayerAdded:Connect(function(player)
		local profile, error = self.Store:Load(player.UserId)
		
		if profile then
			print("Loaded " .. player.Name)
			print("Gold: " .. profile.Data.Gold)
		else
			print("Failed to load: " .. error)
		end
	end)
end
```

**Saving player data**

Modify the data and it autosaves automatically every 60 seconds:

```lua
profile.Data.Gold += 100
profile.Data.Level += 1
table.insert(profile.Data.Inventory, "Sword")
```

**Releasing player data**

When a player leaves, release their profile:

```lua
game.Players.PlayerRemoving:Connect(function(player)
	local profile = self.Store:GetProfile(player.UserId)
	
	if profile then
		profile:Release()
	end
end)
```

## Full Example: Leaderstats Service

Here's a complete, production-ready example that combines everything:

```lua
local Spool = require(game.ReplicatedStorage.Spool)

local Players = game:GetService("Players")

local LeaderstatsService = Spool.CreateService({
	Name = "LeaderstatsService",

	Client = {
		GoldChanged = Spool.Signal()
	}
})

function LeaderstatsService:Init()
	self.Store = Spool.CreateStore("PlayerData", {
		Gold = 0
	})
end

function LeaderstatsService:CreateLeaderstats(player, profile)
	local leaderstats = Instance.new("Folder")
	leaderstats.Name = "leaderstats"
	leaderstats.Parent = player

	local gold = Instance.new("IntValue")
	gold.Name = "Gold"
	gold.Value = profile.Data.Gold
	gold.Parent = leaderstats
end

function LeaderstatsService:LoadPlayer(player)
	local profile = self.Store:Load(player.UserId)

	if not profile then
		if player.Parent == Players then
			player:Kick("Failed to load your data. Please rejoin.")
		end
		return
	end

	-- Release if player left while loading
	if player.Parent ~= Players then
		profile:Release()
		return
	end

	self:CreateLeaderstats(player, profile)

	print("Loaded profile for", player.Name)
	print("Gold:", profile.Data.Gold)
end

function LeaderstatsService:GiveGold(player, amount)
	local profile = self.Store:GetProfile(player.UserId)

	if not profile or not profile.IsActive then
		return
	end

	profile.Data.Gold += amount

	local leaderstats = player:FindFirstChild("leaderstats")
	if not leaderstats then return end

	local gold = leaderstats:FindFirstChild("Gold")
	if not gold then return end

	gold.Value = profile.Data.Gold

	self.Client.GoldChanged:Fire(player, gold.Value)
end

function LeaderstatsService:SavePlayer(player)
	local profile = self.Store:GetProfile(player.UserId)

	if not profile then
		return
	end

	profile:Release()
end

function LeaderstatsService:Start()
	for _, player in Players:GetPlayers() do
		task.spawn(function()
			self:LoadPlayer(player)
		end)
	end

	Players.PlayerAdded:Connect(function(player)
		self:LoadPlayer(player)
	end)

	Players.PlayerRemoving:Connect(function(player)
		self:SavePlayer(player)
	end)
end

return LeaderstatsService
```

This service:
- Creates leaderstats on player join
- Loads and saves player data
- Handles edge cases (player leaving mid-load)
- Fires a signal when gold changes
- Is ready for production
