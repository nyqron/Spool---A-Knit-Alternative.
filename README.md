# Spool - A-Knit-Alternative.

A modern ROBLOX game framework combining services, controllers and built in data persistence, made as an alternative for the now deprecated Knit.

## Why use spool?

**Knit is deprecated.** The Roblox ecosystem has evolved, and game architecture patterns have improved. Spool learns from Knit's strengths while solving real problems:

- **Distributed data safety** - session locking prevents multiple servers from corrupting from destroying/corrupting player data
- **Auto persistence** built in autosave with intelligent spreading (no data store throttling)
- **Clean Networking** - signals, unreliable signals and functions with built in rate limiting and validation
- **Graceful shutdown** - saves all data before server closes, with automatic retry and timeout handling
- **Modern architecture** — Services on server, controllers on client, clean separation of concerns

## Installation

1. Get the latest release from [Releases]
2. Place the Packages folder in ReplicatedStorage
3. Require it in your main script:

```lua
local Spool = require(game.ReplicatedStorage.Packages.Spool)
