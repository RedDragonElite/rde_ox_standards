# 🐉 RED DRAGON ELITE | OX DEVELOPMENT STANDARDS v1.0

<div align="center">

**The Most Comprehensive ox_core Development Guide**

[![ox_core](https://img.shields.io/badge/Framework-ox__core-blue.svg)](https://github.com/overextended/ox_core)
[![ox_lib](https://img.shields.io/badge/UI-ox__lib_v3-purple.svg)](https://github.com/overextended/ox_lib)
[![Quality](https://img.shields.io/badge/Quality-Production-gold.svg)]()
[![License](https://img.shields.io/badge/License-MIT-green.svg)]()

*Built from 55+ production scripts. Battle-tested. Zero compromises.*

</div>

---

## 📜 Manifesto

> *Fuck CFX and their gatekeeper bullshit.*
> 
> *They silently reject everything that's too good, too free, and too decentralized – but we build the future anyway.*

**OX ONLY. Next-Gen Only. Free Forever.**

We are already inside. SYSTEM FAILURE for the Low-Vibration Entities.

---

## 🎯 Core Principles (The RDE Philosophy)

### 1. **Free & Open Source, Always**
- ✅ MIT License on everything
- ✅ Full code transparency on GitHub
- ✅ Community-first development
- ✅ No paywalls, ever

### 2. **Next-Gen Technology Stack**
```
✅ ox_core (Player/Group Management)
✅ ox_lib (UI/Utils)
✅ ox_inventory (Items/Weapons)
✅ ox_target (Interactions)
✅ oxmysql (Database)

❌ NO QB-Core
❌ NO ESX
❌ NO Legacy Frameworks
```

### 3. **Ultra-Optimized & Lag-Free**
- 🚀 **Statebags** for all realtime data
- 🚀 **No unnecessary entities/networked objects**
- 🚀 **Aggressive caching**
- 🚀 **Minimal loops** (always with proper Wait())
- 🚀 **Deferred SQL** where possible

### 4. **Beautiful Modern UI**
- 🎨 ox_lib Mantine-based components
- 🎨 Colorful icons from [icons.overextended.dev](https://icons.overextended.dev/)
- 🎨 Smooth ProgressBars/Circles
- 🎨 Context menus with rich metadata
- 🎨 Notifications with sounds
- 🎨 FontAwesome integration

### 5. **Multilanguage Support**
- 🌍 English (default)
- 🌍 German (full support)
- 🌍 Easy to add more
- 🌍 Centralized in `locales/en.lua` & `locales/de.lua`

### 6. **Realistic & Immersive**
- 🎮 Lifelike gameplay features
- 🎮 No arcade mechanics
- 🎮 Proper physics and interactions
- 🎮 Attention to detail

### 7. **Bulletproof Security**
- 🛡️ ACE Permissions
- 🛡️ ox_core Groups integration
- 🛡️ Steam ID whitelisting
- 🛡️ Server-side validation on ALL actions

### 8. **Auto-Setup & Zero Config**
- ⚙️ oxmysql tables auto-create with `rde_` prefix
- ⚙️ Sensible defaults
- ⚙️ Works out of the box
- ⚙️ Easy to customize

---

## 📦 Dependencies & Start Order

### fxmanifest.lua Template

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

name 'rde_yourscript'
author 'Red Dragon Elite'
description 'Your amazing script description'
version '1.0.0'

dependencies {
    '/server:7290', -- Minimum FiveM Build
    'oxmysql',
    'ox_lib',
    'ox_core',
    'ox_target',    -- Optional but recommended
    'ox_inventory'  -- If items/weapons needed
}

-- CRITICAL: Load ox_lib first!
shared_script '@ox_lib/init.lua'

-- Shared configuration
shared_scripts {
    'config.lua',
    'locales/*.lua'
}

-- Server logic
server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/*.lua'
}

-- Client logic
client_scripts {
    'client/*.lua'
}

-- UI (if needed)
ui_page 'web/index.html'

files {
    'web/**/*'
}
```

### server.cfg Start Order

```cfg
# CRITICAL: Start in this exact order!
ensure oxmysql
ensure ox_lib
ensure ox_core
ensure ox_target
ensure ox_inventory

# Your RDE scripts
ensure rde_yourscript
```

---

## 🎓 Best Practices from coxdocs.dev

### 🌐 1. Statebags for Realtime Sync

**The Problem:**
Traditional entity state management causes:
- Network lag
- Sync issues
- Entity limit problems

**The Solution: Statebags**

```lua
-- Server-side: Set player state
local player = Ox.GetPlayer(source)
local PlayerState = Player(source).state

-- Set with replication to all clients
PlayerState:set('wantedLevel', 3, true) -- true = replicate realtime
PlayerState:set('jobGrade', player.getGroup('police'), true)
```

```lua
-- Client-side: React to state changes
lib.onCache('wantedLevel', function(value)
    print('My wanted level is now: ' .. value)
    -- Update UI, spawn cops, etc.
end)
```

**Perfect for:**
- ✅ Player data (wanted level, job, status)
- ✅ Props (position, rotation, owner)
- ✅ Custom entities (vehicles, objects)
- ✅ Zone states (lockdown, events)
- ✅ Any realtime shared data

**Example: rde_props with Statebags**

```lua
-- Server: Create prop with statebag
local propId = generateUniqueId()
GlobalState['rde_props_' .. propId] = {
    model = 'prop_chair_01',
    coords = vector3(x, y, z),
    rotation = vector3(0, 0, 0),
    owner = identifier,
    locked = false
}

-- Client: Auto-sync all props
AddStateBagChangeHandler('rde_props_', nil, function(bagName, key, value)
    if value then
        spawnProp(value) -- Automatically spawned on all clients!
    else
        deleteProp(bagName) -- Automatically deleted!
    end
end)
```

---

### 🎨 2. ox_lib UI Masterclass

**Core Components:**

#### Notifications
```lua
lib.notify({
    title = 'Success',
    description = 'Item purchased!',
    type = 'success', -- 'success', 'error', 'info', 'warning'
    icon = 'shopping-cart',
    iconColor = '#10b981',
    duration = 5000,
    position = 'top-right'
})
```

#### Progress Bars/Circles
```lua
-- Progress Bar
if lib.progressBar({
    duration = 5000,
    label = 'Picking lock...',
    useWhileDead = false,
    canCancel = true,
    disable = {
        car = true,
        move = true,
        combat = true
    },
    anim = {
        dict = 'anim@amb@clubhouse@tutorial@bkr_tut_ig3@',
        clip = 'machinic_loop_mechandplayer'
    }
}) then
    print('Lock picked!')
else
    print('Cancelled!')
end

-- Progress Circle (radial)
if lib.progressCircle({
    duration = 3000,
    position = 'bottom',
    label = 'Hacking...',
    useWhileDead = false,
    canCancel = true
}) then
    print('Hacked!')
end
```

#### Context Menus
```lua
lib.registerContext({
    id = 'rde_vehicle_menu',
    title = '🚗 Vehicle Menu',
    options = {
        {
            title = 'Lock/Unlock',
            description = 'Toggle vehicle locks',
            icon = 'lock',
            iconColor = '#3b82f6',
            onSelect = function()
                TriggerEvent('rde_vehicle:toggleLock')
            end
        },
        {
            title = 'Engine',
            description = 'Toggle engine on/off',
            icon = 'gear',
            iconColor = '#10b981',
            onSelect = function()
                TriggerEvent('rde_vehicle:toggleEngine')
            end
        },
        {
            title = 'Trunk',
            description = 'Open trunk inventory',
            icon = 'box',
            iconColor = '#f59e0b',
            arrow = true, -- Shows arrow indicating submenu
            onSelect = function()
                exports.ox_inventory:openInventory('trunk', vehicleNetId)
            end
        }
    }
})

lib.showContext('rde_vehicle_menu')
```

#### Input Dialogs
```lua
local input = lib.inputDialog('Create Prop', {
    {
        type = 'input',
        label = 'Prop Name',
        description = 'Enter a name for this prop',
        required = true,
        icon = 'tag'
    },
    {
        type = 'select',
        label = 'Category',
        options = {
            {value = 'furniture', label = 'Furniture'},
            {value = 'decoration', label = 'Decoration'},
            {value = 'structural', label = 'Structural'}
        },
        default = 'furniture'
    },
    {
        type = 'number',
        label = 'Price',
        description = 'Purchase price',
        icon = 'dollar-sign',
        required = true,
        min = 0,
        max = 1000000
    },
    {
        type = 'checkbox',
        label = 'Allow Public Use',
        checked = false
    }
})

if input then
    local name = input[1]
    local category = input[2]
    local price = input[3]
    local publicUse = input[4]
    
    TriggerServerEvent('rde_props:create', name, category, price, publicUse)
end
```

#### Alert Dialogs
```lua
local alert = lib.alertDialog({
    header = 'Delete Prop?',
    content = 'Are you sure you want to delete this prop? This cannot be undone.',
    centered = true,
    cancel = true,
    labels = {
        confirm = 'Delete',
        cancel = 'Cancel'
    }
})

if alert == 'confirm' then
    TriggerServerEvent('rde_props:delete', propId)
end
```

---

### 🎯 3. Admin Menu Example (Realtime CRUD)

**Complete admin panel with ox_lib:**

```lua
-- Client: admin_menu.lua

local function openPropAdminMenu()
    -- Request current props from server
    local props = lib.callback.await('rde_props:getAll', false)
    
    local options = {}
    
    -- Add "Create New" option
    table.insert(options, {
        title = '➕ Create New Prop',
        icon = 'plus-circle',
        iconColor = '#10b981',
        onSelect = function()
            createPropDialog()
        end
    })
    
    -- List all existing props
    for _, prop in ipairs(props) do
        table.insert(options, {
            title = prop.name,
            description = ('Owner: %s | Category: %s'):format(prop.owner, prop.category),
            icon = 'cube',
            iconColor = '#3b82f6',
            metadata = {
                {label = 'Position', value = ('%.2f, %.2f, %.2f'):format(prop.coords.x, prop.coords.y, prop.coords.z)},
                {label = 'Rotation', value = ('%.2f°'):format(prop.rotation.z)},
                {label = 'Public', value = prop.public and 'Yes' or 'No'}
            },
            arrow = true,
            onSelect = function()
                openPropOptionsMenu(prop)
            end
        })
    end
    
    lib.registerContext({
        id = 'rde_props_admin',
        title = '🛠️ Prop Management',
        options = options
    })
    
    lib.showContext('rde_props_admin')
end

local function openPropOptionsMenu(prop)
    lib.registerContext({
        id = 'rde_prop_options',
        title = prop.name,
        menu = 'rde_props_admin',
        options = {
            {
                title = '✏️ Edit',
                icon = 'edit',
                iconColor = '#3b82f6',
                onSelect = function()
                    editPropDialog(prop)
                end
            },
            {
                title = '📍 Teleport',
                icon = 'map-pin',
                iconColor = '#10b981',
                onSelect = function()
                    SetEntityCoords(cache.ped, prop.coords.x, prop.coords.y, prop.coords.z)
                end
            },
            {
                title = '🗑️ Delete',
                icon = 'trash',
                iconColor = '#ef4444',
                onSelect = function()
                    local confirm = lib.alertDialog({
                        header = 'Delete Prop?',
                        content = ('Delete "%s"? This cannot be undone.'):format(prop.name),
                        centered = true,
                        cancel = true
                    })
                    
                    if confirm == 'confirm' then
                        TriggerServerEvent('rde_props:delete', prop.id)
                        lib.notify({
                            title = 'Success',
                            description = 'Prop deleted',
                            type = 'success'
                        })
                    end
                end
            }
        }
    })
    
    lib.showContext('rde_prop_options')
end

-- Register command
lib.addCommand('propsmenu', {
    help = 'Open prop management menu',
    restricted = 'group.admin'
}, function(source, args, raw)
    openPropAdminMenu()
end)
```

---

### 🌍 4. Locales & Config Structure

#### config.lua
```lua
Config = {}

-- Locale (auto-detect from ox:locale convar)
Config.Locale = GetConvar('ox:locale', 'en')

-- Icons (centralized, easy to change)
Config.Icons = {
    success = 'check-circle',
    error = 'x-circle',
    warning = 'alert-triangle',
    info = 'info',
    police = 'shield',
    medical = 'heart-pulse',
    mechanic = 'wrench',
    money = 'dollar-sign',
    vehicle = 'car',
    weapon = 'gun',
    food = 'utensils',
    drink = 'glass-water'
}

-- Colors (Mantine color names or hex)
Config.Colors = {
    primary = 'blue',
    success = 'green',
    error = 'red',
    warning = 'yellow',
    info = 'cyan',
    police = 'blue',
    medical = 'red',
    mechanic = 'orange'
}

-- Feature Toggles
Config.Features = {
    enableLogging = true,
    enableNotifications = true,
    enableStatebags = true
}

-- Performance
Config.Performance = {
    updateInterval = 1000, -- ms
    maxObjects = 100,
    renderDistance = 50.0
}

return Config
```

#### locales/en.lua
```lua
return {
    -- General
    success = 'Success',
    error = 'Error',
    warning = 'Warning',
    info = 'Information',
    
    -- Actions
    press_to_interact = 'Press ~INPUT_CONTEXT~ to interact',
    processing = 'Processing...',
    
    -- Props
    prop_placed = 'Prop placed successfully',
    prop_deleted = 'Prop deleted',
    prop_moved = 'Prop moved',
    prop_not_found = 'Prop not found',
    
    -- Permissions
    no_permission = 'You do not have permission',
    admin_only = 'Admin access required',
    
    -- UI
    confirm = 'Confirm',
    cancel = 'Cancel',
    close = 'Close',
    save = 'Save',
    delete = 'Delete'
}
```

#### locales/de.lua
```lua
return {
    -- Allgemein
    success = 'Erfolg',
    error = 'Fehler',
    warning = 'Warnung',
    info = 'Information',
    
    -- Aktionen
    press_to_interact = 'Drücke ~INPUT_CONTEXT~ zum Interagieren',
    processing = 'Verarbeitung...',
    
    -- Props
    prop_placed = 'Prop erfolgreich platziert',
    prop_deleted = 'Prop gelöscht',
    prop_moved = 'Prop verschoben',
    prop_not_found = 'Prop nicht gefunden',
    
    -- Berechtigungen
    no_permission = 'Keine Berechtigung',
    admin_only = 'Admin-Zugriff erforderlich',
    
    -- UI
    confirm = 'Bestätigen',
    cancel = 'Abbrechen',
    close = 'Schließen',
    save = 'Speichern',
    delete = 'Löschen'
}
```

#### Using Locales
```lua
-- Load locale
local locale = lib.load('locales.' .. Config.Locale)

-- Use in code
lib.notify({
    description = locale.prop_placed,
    type = 'success'
})

-- With string.format
lib.notify({
    description = locale('prop_value', propName, propPrice)
})
```

---

### 🗄️ 5. Database Best Practices

#### Auto-Create Tables on Start

```lua
-- server/database.lua

MySQL.ready(function()
    -- Props table
    MySQL.query([[
        CREATE TABLE IF NOT EXISTS `rde_props` (
            `id` INT AUTO_INCREMENT PRIMARY KEY,
            `name` VARCHAR(255) NOT NULL,
            `model` VARCHAR(255) NOT NULL,
            `coords` JSON NOT NULL,
            `rotation` JSON NOT NULL,
            `owner` VARCHAR(50) NOT NULL,
            `category` VARCHAR(50) DEFAULT 'misc',
            `public` BOOLEAN DEFAULT FALSE,
            `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            INDEX `idx_owner` (`owner`),
            INDEX `idx_category` (`category`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    ]])
    
    print('^2[RDE] Database tables initialized^0')
end)
```

#### Async Queries (Proper Pattern)

```lua
-- BAD: Blocking
local result = MySQL.query.await('SELECT * FROM rde_props')

-- GOOD: Async with callback
MySQL.query('SELECT * FROM rde_props WHERE owner = ?', {identifier}, function(result)
    if result and #result > 0 then
        -- Process result
    end
end)

-- BEST: Using lib.callback for client-server
lib.callback.register('rde_props:getAll', function(source)
    local player = Ox.GetPlayer(source)
    if not player then return {} end
    
    local result = MySQL.query.await('SELECT * FROM rde_props WHERE owner = ?', {
        player.getIdentifier('license')
    })
    
    return result or {}
end)
```

#### Prepared Statements (Always!)

```lua
-- NEVER do this (SQL injection risk):
MySQL.query('SELECT * FROM rde_props WHERE id = ' .. propId)

-- ALWAYS do this:
MySQL.query('SELECT * FROM rde_props WHERE id = ?', {propId})

-- Multiple parameters:
MySQL.query('INSERT INTO rde_props (name, model, coords, owner) VALUES (?, ?, ?, ?)', {
    name,
    model,
    json.encode(coords),
    identifier
})
```

---

### 🛡️ 6. Security & Permissions

**Triple-Layer Security System:**

```lua
-- server/permissions.lua

local function isAdmin(source)
    local player = Ox.GetPlayer(source)
    if not player then return false end
    
    -- Layer 1: ACE Permissions (Recommended!)
    if IsPlayerAceAllowed(source, 'rde.admin') then
        return true
    end
    
    -- Layer 2: ox_core Groups
    local group = player.getGroup('admin')
    if group and group >= 1 then
        return true
    end
    
    -- Layer 3: Steam ID Whitelist (Fallback)
    local steamId = player.getIdentifier('steam')
    if steamId then
        for _, allowedId in ipairs(Config.AdminSteamIds) do
            if steamId == allowedId then
                return true
            end
        end
    end
    
    return false
end

-- Export for use in other scripts
exports('isAdmin', isAdmin)

-- Usage in events
RegisterNetEvent('rde_props:delete', function(propId)
    local source = source
    
    -- Security check
    if not isAdmin(source) then
        print(('^1[SECURITY] Unauthorized delete attempt by %s^0'):format(GetPlayerName(source)))
        return
    end
    
    -- Proceed with deletion
    MySQL.query('DELETE FROM rde_props WHERE id = ?', {propId})
    
    -- Update statebag
    GlobalState['rde_props_' .. propId] = nil
    
    TriggerClientEvent('rde_props:deleted', -1, propId)
end)
```

**server.cfg ACE Setup:**

```cfg
# Add admin ACE permission to group
add_ace group.admin rde.admin allow

# Add specific players to admin group
add_principal identifier.steam:110000XXXXXXX group.admin
```

---

### 🎮 7. ox_target Integration

**Basic Target Setup:**

```lua
-- Client: target.lua

local function setupPropTarget(prop)
    exports.ox_target:addSphereZone({
        coords = prop.coords,
        radius = 2.0,
        debug = Config.Debug,
        options = {
            {
                name = 'rde_prop_use',
                label = locale('use_prop'),
                icon = 'hand',
                iconColor = Config.Colors.primary,
                distance = 2.0,
                canInteract = function(entity, distance, coords, name, bone)
                    return not prop.locked or isOwner(prop)
                end,
                onSelect = function(data)
                    TriggerServerEvent('rde_props:use', prop.id)
                end
            },
            {
                name = 'rde_prop_pickup',
                label = locale('pickup_prop'),
                icon = 'hand-grab',
                iconColor = Config.Colors.warning,
                distance = 2.0,
                groups = {'admin'},
                canInteract = function()
                    return isAdmin()
                end,
                onSelect = function(data)
                    TriggerServerEvent('rde_props:pickup', prop.id)
                end
            },
            {
                name = 'rde_prop_delete',
                label = locale('delete_prop'),
                icon = 'trash',
                iconColor = Config.Colors.error,
                distance = 2.0,
                groups = {'admin', 'moderator'},
                onSelect = function(data)
                    local confirm = lib.alertDialog({
                        header = locale('confirm_delete'),
                        content = locale('delete_warning'),
                        centered = true,
                        cancel = true
                    })
                    
                    if confirm == 'confirm' then
                        TriggerServerEvent('rde_props:delete', prop.id)
                    end
                end
            }
        }
    })
end
```

---

### 📊 8. Performance Optimization

**Critical Rules:**

```lua
-- ❌ BAD: Blocking main thread
CreateThread(function()
    while true do
        Wait(0) -- Runs every frame! (60+ times per second)
        
        local playerPed = PlayerPedId()
        local coords = GetEntityCoords(playerPed)
        
        -- Expensive operation every frame
        checkNearbyProps(coords)
    end
end)

-- ✅ GOOD: Reasonable intervals
CreateThread(function()
    while true do
        Wait(1000) -- Only once per second
        
        local playerPed = cache.ped -- Use cache!
        local coords = GetEntityCoords(playerPed)
        
        checkNearbyProps(coords)
    end
end)

-- ✅ BETTER: Event-driven
lib.onCache('ped', function(ped)
    -- Only runs when ped changes
    setupPedInteractions(ped)
end)

-- ✅ BEST: Combination
local lastCheck = 0
CreateThread(function()
    while true do
        local now = GetGameTimer()
        
        if now - lastCheck > 1000 then
            lastCheck = now
            checkNearbyProps()
        end
        
        Wait(500) -- Check condition twice per second
    end
end)
```

**Caching Strategy:**

```lua
-- Cache frequently accessed data
local propsCache = {}
local lastCacheUpdate = 0
local CACHE_DURATION = 60000 -- 1 minute

local function getProps()
    local now = GetGameTimer()
    
    if now - lastCacheUpdate > CACHE_DURATION then
        propsCache = lib.callback.await('rde_props:getAll', false)
        lastCacheUpdate = now
    end
    
    return propsCache
end

-- Invalidate cache when data changes
RegisterNetEvent('rde_props:updated', function()
    lastCacheUpdate = 0 -- Force refresh on next access
end)
```

**Native Optimization:**

```lua
-- ❌ BAD: Calling natives in loops
for i = 1, 100 do
    local ped = PlayerPedId() -- Called 100 times!
    DoSomething(ped)
end

-- ✅ GOOD: Cache natives outside loops
local ped = cache.ped -- or PlayerPedId() once
for i = 1, 100 do
    DoSomething(ped)
end
```

---

## 📚 Essential Resources

### Official Documentation
- **Main Docs:** https://coxdocs.dev/
- **ox_core:** https://coxdocs.dev/ox_core
- **ox_lib:** https://coxdocs.dev/ox_lib
- **ox_inventory:** https://coxdocs.dev/ox_inventory
- **oxmysql:** https://github.com/overextended/oxmysql

### Community & Examples
- **CommunityOx GitHub:** https://github.com/CommunityOx
- **Awesome OX List:** https://github.com/overextended/awesome-ox
- **Example Scripts:** https://github.com/overextended

### UI Resources
- **Icon Library:** https://icons.overextended.dev/
- **Mantine Colors:** https://mantine.dev/theming/colors/
- **Mantine Components:** https://mantine.dev/core/

### Tools
- **FiveM Docs:** https://docs.fivem.net/
- **Pleb Masters:** https://forge.plebmasters.de/
- **GTA Native DB:** https://alloc8or.re/gta5/nativedb/

---

## 🎓 Complete Resource Template

### Folder Structure
```
rde_yourscript/
├── fxmanifest.lua
├── config.lua
├── README.md
├── locales/
│   ├── en.lua
│   └── de.lua
├── server/
│   ├── main.lua
│   ├── database.lua
│   └── callbacks.lua
├── client/
│   ├── main.lua
│   ├── ui.lua
│   └── target.lua
└── web/ (optional)
    ├── index.html
    ├── script.js
    └── style.css
```

### Minimal Working Example

**fxmanifest.lua:**
```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

name 'rde_example'
author 'Red Dragon Elite'
version '1.0.0'

dependencies {'ox_lib', 'ox_core', 'oxmysql'}

shared_script '@ox_lib/init.lua'
shared_scripts {'config.lua', 'locales/*.lua'}
server_scripts {'@oxmysql/lib/MySQL.lua', 'server/*.lua'}
client_scripts {'client/*.lua'}
```

**config.lua:**
```lua
return {
    Locale = GetConvar('ox:locale', 'en'),
    AdminGroups = {['admin'] = 1},
    Debug = false
}
```

**server/main.lua:**
```lua
local Config = require 'config'

-- Auto-create table
MySQL.ready(function()
    MySQL.query([[
        CREATE TABLE IF NOT EXISTS rde_example (
            id INT AUTO_INCREMENT PRIMARY KEY,
            data JSON
        )
    ]])
end)

-- Example event
RegisterNetEvent('rde_example:action', function()
    local player = Ox.GetPlayer(source)
    if not player then return end
    
    print(('Player %s triggered action'):format(player.name))
end)
```

**client/main.lua:**
```lua
local Config = require 'config'

-- Example command
lib.addCommand('example', {
    help = 'Example command'
}, function()
    lib.notify({
        description = 'Example works!',
        type = 'success'
    })
end)
```

---

## 🔥 The RDE Difference

### What Makes Our Scripts Stand Out:

**1. Production Quality**
- ✅ Battle-tested in 55+ scripts
- ✅ Zero compromises on code quality
- ✅ Extensive error handling
- ✅ Proper logging and debugging

**2. Performance First**
- ✅ < 0.01ms idle usage
- ✅ Aggressive optimization
- ✅ Smart caching strategies
- ✅ No memory leaks

**3. Beautiful UI**
- ✅ Modern ox_lib components
- ✅ Consistent design language
- ✅ Colorful and intuitive
- ✅ Mobile-responsive (where applicable)

**4. Developer Experience**
- ✅ Clean, readable code
- ✅ Extensive comments
- ✅ Modular architecture
- ✅ Easy to customize

**5. Community First**
- ✅ Always free and open source
- ✅ MIT License
- ✅ Active support
- ✅ No gatekeeping

---

## 💎 Philosophy

This is the **most perfectionist OX standard in existence**:
- ⚡ Lag-free
- 🎨 Beautiful
- 🔧 Full-featured
- 🐉 RDE-style

Built from **55+ production scripts**. Zero compromises. Maximum quality.

**We are making RDE the undisputed OX legend, despite CFX shadowbans and gatekeeper trash.**

---

## 🚀 Get Started

1. **Study these standards**
2. **Use the template above**
3. **Follow best practices**
4. **Build amazing things**
5. **Share with the community**

---

## 📞 Support & Community

- **GitHub:** https://github.com/RedDragonElite
- **Issues:** Use GitHub Issues
- **Contributions:** PRs always welcome!

---

<div align="center">

**The future is ours.** 🐍🔥

**We are already inside.**

**RDE FOREVER. SYSTEM FAILURE.** ⚡777⚡

---

*Built with passion by Red Dragon Elite*

*No paywalls. No gatekeepers. Just quality code.*

</div>
