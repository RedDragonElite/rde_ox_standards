🔥🐉 .:: RedDragonElite ::. OX DEVELOPMENT STANDARDS v1.0 🖤

**Fuck CFX and their gatekeeper bullshit.**  
They silently reject everything that's too good, too free, and too decentralized – but we build the future anyway.

OX ONLY. Next-Gen Only. Free Forever.  
We are already inside. SYSTEM FAILURE for the Low-Vibration Entities.

#### 1. Core Principles (RDE Philosophy)
- Free, Open-Source, Community-First: MIT-License, everything on GitHub under the RedDragonElite organization.
- Next-Gen Only: Pure ox_core + ox_lib + ox_inventory + ox_target + oxmysql. No QB/ESX garbage.
- Ultra-Optimized & Lagfree: Statebags for all realtime data (no unnecessary entities/networks), caching, minimal loops, defer SQL where possible.
- Beautiful Modern UI: ox_lib Mantine-based – colorful icons (from https://icons.overextended.dev/ or FontAwesome), smooth ProgressBars/Circles, Context-Menus, Notify with sounds.
- Multilanguage: English default, full German support. All strings in locales/en.lua & locales/de.lua, Icons/Colors centralized in Config.
- Realistic & Immersive: Gameplay features (e.g. AI-Police, Props, Jobs) always lifelike and immersive – no arcade crap.
- Bulletproof Security: Ace-Perms + ox_core Groups + SteamID checks for all admin features.
- Auto-Setup: oxmysql tables auto-create with rde_ prefix on resource start (if not exists).

#### 2. Dependencies & Start-Order (fxmanifest.lua)
lua
dependencies {
    '/server:7293',  -- Minimum FiveM Build
    'oxmysql',
    'ox_lib',
    'ox_core',
    'ox_target',     -- Optional but recommended
    'ox_inventory',  -- If items/weapons needed
}

shared_script '@ox_lib/init.lua'  -- Always first!
Start-Order in server.cfg:
start oxmysql
start ox_lib
start ox_core
start ox_target
start ox_inventory
start rde_yourscript

#### 3. Best Practices from coxdocs.dev
- **Statebags Realtime Sync**: Always use for persistent/realtime data (Props, Player-States, Custom Entities).  
  Example:
  
lua
  local PlayerData = Player(source).state
  PlayerData:set('wantedLevel', 3, true)  -- true = replicate to all clients realtime

  -- Client-side
  lib.onCache('wantedLevel', function(value) print('Wanted: '..value) end)
 
  Perfect for rde_props: store props in statebag arrays, realtime move/rotate/delete for everyone.

- **ox_lib UI Masterclass**:
  - Always use lib.notify(), lib.progressBar/Circle(), lib.contextMenu(), lib.dialog().
  - Icons configurable and colorful.
  - Admin menus: Ingame realtime CRUD with lib.registerContext() + Statebag updates.
    Example Admin Prop Menu:
    
lua
    lib.registerContext({
        id = 'rde_props_admin',
        title = locale('admin_props_title'),
        options = {
            { title = locale('place_prop'), icon = '🛠', onSelect = function() TriggerEvent('rde_props:placeMode') end },
            { title = locale('delete_nearby'), icon = '🗑', onSelect = deleteNearbyProps },
        }
    })
    lib.showContext('rde_props_admin')
   

- **Locales & Config**:
  
lua
  -- config.lua
  Config = {}
  Config.Locale = GetConvar('ox:locale', 'en')  -- Auto English/German
  Config.Icons = { police = '🚔', success = '✅', error = '❌' }
  Config.Colors = { primary = 'blue', success = 'green' }
 

- **oxmysql & Auto-Tables**:
  Tables always with rde_ prefix. Auto-create on start:
  
lua
  MySQL.ready(function()
      MySQL.query([[CREATE TABLE IF NOT EXISTS rde_props (
          id INT AUTO_INCREMENT PRIMARY KEY,
          data JSON,
          owner VARCHAR(50)
      )]])
  end)
  `

- Permissions & Auth:
  Combo: ox_core Groups + Ace + SteamID.
 
  local function isAdmin(src)
      local player = Core.GetPlayer(src)
      if not player then return false end
      if (player.getGroup('admin') or 0) >= 1 then return true end
      if IsPlayerAceAllowed(src, 'rde.admin') then return true end
      for _, id in ipairs(GetPlayerIdentifiers(src)) do
          if id:match('steam:') == '1100001xxxxxxxx' then return true end
      end
      return false
  end
  
- Realtime Admin CRUD Ingame:
  ox_lib menus with Add/Edit/Delete → direct Statebag + DB update (replicate true).

- Additional Key Features:
  - ox_target for all interactions.
  - lib.cache for Player/Vehicle data.
  - Always use ox:playerLoaded, ox:playerDeath, etc.
  - Performance: No while true loops without Wait(0/100), defer everything.

#### 4. Essential Resources
- Main Docs: https://coxdocs.dev/
- CommunityOx GitHub: https://github.com/CommunityOx
- Awesome-OX List: https://github.com/overextended/awesome-ox
- UI Icons: https://icons.overextended.dev/
- Mantine Colors/Themes: https://mantine.dev/theming/colors/

This is the most perfectionist OX standard in existence – lagfree, beautiful, full-featured, RDE-style.  
55+ scripts built with it. We are making RDE the undisputed OX legend, despite CFX shadowbans and gatekeeper trash.  

The future is ours. 🐍🔥

We are already inside.   

RDE FOREVER. SYSTEM FAILURE. ⚡️777⚡️ 🐍🔥🖤
