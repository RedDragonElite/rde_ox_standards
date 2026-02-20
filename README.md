# 🐉 RDE | OX Development Standards v1.0

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-red?style=for-the-badge&logo=github)
![License](https://img.shields.io/badge/license-RDE%20Black%20Flag%20v6.66-black?style=for-the-badge)
![ox_core](https://img.shields.io/badge/ox__core-Only-blue?style=for-the-badge)
![Free](https://img.shields.io/badge/price-FREE%20FOREVER-brightgreen?style=for-the-badge)

**The internal development standard behind every RDE resource.**
Battle-tested across 55+ production scripts. Zero compromises.

*Built by [Red Dragon Elite](https://rd-elite.com) | SerpentsByte*

</div>

---

## 📖 Table of Contents

- [Philosophy](#-philosophy)
- [Core Principles](#-core-principles)
- [Technology Stack](#-technology-stack)
- [fxmanifest Template](#-fxmanifest-template)
- [server.cfg Start Order](#-servercfg-start-order)
- [Statebags](#-1-statebags-for-realtime-sync)
- [ox_lib UI](#-2-ox_lib-ui-masterclass)
- [Locales & Config](#-3-locales--config-structure)
- [Database](#-4-database-best-practices)
- [Security & Permissions](#-5-security--permissions)
- [ox_target Integration](#-6-ox_target-integration)
- [Performance](#-7-performance-optimization)
- [Resource Template](#-complete-resource-template)
- [Essential Resources](#-essential-resources)
- [License](#-license)

---

## 🎯 Philosophy

This document is the internal standard behind every RDE resource. Every script we release is built against these rules — no exceptions, no shortcuts.

**OX only. Next-gen only. Free forever.**

We don't gatekeep knowledge. Read this, understand it, build better things.

---

## 🏴 Core Principles

### 1. Free & Open Source — Always
Every RDE resource is source-available, forkable, and free. If you paid for any of our scripts, you got scammed. No subscriptions, no obfuscation, no paywalls — ever.

### 2. Next-Gen Stack Only
We build exclusively on the OX ecosystem. No legacy frameworks, no ESX shims, no QB-Core compatibility layers. Clean from the ground up.

### 3. Performance First
Statebags over polling. Caching over repeat queries. Event-driven over tick-based. Every thread has a `Wait()`. Nothing runs at frame rate unless it absolutely has to.

### 4. Beautiful, Consistent UI
ox_lib Mantine components everywhere. Lucide icons from [icons.overextended.dev](https://icons.overextended.dev/). Consistent color language. Smooth progress bars. No ugly raw chat prints as UI.

### 5. Multilanguage by Default
Every resource ships with English and German. Locales live in `locales/en.lua` and `locales/de.lua`. Adding a new language is always a single file.

### 6. Bulletproof Security
Server-side validation on every action. Triple-layer admin verification — ACE + ox_core groups + Steam ID. No client-authoritative logic for anything that matters.

### 7. Zero Config to Start
`oxmysql` tables auto-create with `rde_` prefix. Sensible defaults in `config.lua`. Works out of the box. Easy to customize when needed.

---

## 🔧 Technology Stack

```
✅ ox_core      — Player, character, group management
✅ ox_lib       — UI, callbacks, commands, utilities
✅ ox_inventory — Items, weapons, stashes
✅ ox_target    — World interactions
✅ oxmysql      — Database layer

❌ QBCore
❌ ESX
❌ Any legacy framework
```

---

## 📄 fxmanifest Template

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

name        'rde_yourscript'
author      'Red Dragon Elite'
description 'Description of your script'
version     '1.0.0'

dependencies {
    '/server:7290',   -- minimum FiveM build
    'oxmysql',
    'ox_lib',
    'ox_core',
    'ox_target',      -- if using world interactions
    'ox_inventory'    -- if using items or weapons
}

shared_scripts {
    '@ox_lib/init.lua',
    '@ox_core/lib/init.lua',
    'config.lua',
    'locales/*.lua'
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/*.lua'
}

client_scripts {
    'client/*.lua'
}

-- Only if using a NUI panel
ui_page 'web/index.html'
files { 'web/**/*' }
```

---

## 📋 server.cfg Start Order

```cfg
# Always in this exact order
ensure oxmysql
ensure ox_lib
ensure ox_core
ensure ox_target
ensure ox_inventory

# RDE scripts after all dependencies
ensure rde_yourscript
```

---

## 📡 1. Statebags for Realtime Sync

Statebags are the backbone of all RDE realtime data. No polling, no lag, no sync issues.

### Setting Player State (Server)

```lua
local PlayerState = Player(source).state

-- true = replicate to all clients instantly
PlayerState:set('wantedLevel', 3, true)
PlayerState:set('jobGrade', player.getGroup('police'), true)
```

### Reacting to State Changes (Client)

```lua
lib.onCache('wantedLevel', function(value)
    print('Wanted level is now: ' .. value)
    -- update UI, trigger logic, etc.
end)
```

### Global State for World Objects (e.g. Props)

```lua
-- Server: create a prop statebag
local propId = generateUniqueId()
GlobalState['rde_props_' .. propId] = {
    model  = 'prop_chair_01',
    coords = vector3(x, y, z),
    owner  = identifier,
    locked = false,
}

-- Client: react automatically on all clients
AddStateBagChangeHandler('rde_props_', nil, function(bagName, key, value)
    if value then
        spawnProp(value)   -- spawned on all clients
    else
        deleteProp(bagName) -- deleted on all clients
    end
end)
```

**Use statebags for:** player data, prop states, zone states, vehicle states, any realtime shared data.

---

## 🎨 2. ox_lib UI Masterclass

### Notifications

```lua
lib.notify({
    title       = 'Success',
    description = 'Item purchased!',
    type        = 'success',   -- 'success' | 'error' | 'info' | 'warning'
    icon        = 'shopping-cart',
    iconColor   = '#10b981',
    duration    = 5000,
    position    = 'top-right'
})
```

### Progress Bar

```lua
if lib.progressBar({
    duration     = 5000,
    label        = 'Picking lock...',
    useWhileDead = false,
    canCancel    = true,
    disable      = { car = true, move = true, combat = true },
    anim         = {
        dict = 'anim@amb@clubhouse@tutorial@bkr_tut_ig3@',
        clip = 'machinic_loop_mechandplayer'
    }
}) then
    -- completed
else
    -- cancelled
end
```

### Progress Circle

```lua
if lib.progressCircle({
    duration     = 3000,
    position     = 'bottom',
    label        = 'Hacking...',
    useWhileDead = false,
    canCancel    = true,
}) then
    -- completed
end
```

### Context Menu

```lua
lib.registerContext({
    id      = 'rde_vehicle_menu',
    title   = '🚗 Vehicle Menu',
    options = {
        {
            title       = 'Lock / Unlock',
            description = 'Toggle vehicle locks',
            icon        = 'lock',
            iconColor   = '#3b82f6',
            onSelect    = function()
                TriggerEvent('rde_vehicle:toggleLock')
            end,
        },
        {
            title       = 'Trunk',
            description = 'Open trunk inventory',
            icon        = 'box',
            iconColor   = '#f59e0b',
            arrow       = true,
            onSelect    = function()
                exports.ox_inventory:openInventory('trunk', vehicleNetId)
            end,
        },
    }
})

lib.showContext('rde_vehicle_menu')
```

### Input Dialog

```lua
local input = lib.inputDialog('Create Prop', {
    { type = 'input',    label = 'Prop Name',  required = true, icon = 'tag' },
    { type = 'select',   label = 'Category',
      options = {
          { value = 'furniture',  label = 'Furniture' },
          { value = 'structural', label = 'Structural' },
      },
      default = 'furniture'
    },
    { type = 'number',   label = 'Price', icon = 'dollar-sign', required = true, min = 0, max = 1000000 },
    { type = 'checkbox', label = 'Allow Public Use', checked = false },
})

if input then
    local name, category, price, publicUse = input[1], input[2], input[3], input[4]
    TriggerServerEvent('rde_props:create', name, category, price, publicUse)
end
```

### Alert Dialog

```lua
local alert = lib.alertDialog({
    header  = 'Delete Prop?',
    content = 'This cannot be undone.',
    centered = true,
    cancel  = true,
    labels  = { confirm = 'Delete', cancel = 'Cancel' }
})

if alert == 'confirm' then
    TriggerServerEvent('rde_props:delete', propId)
end
```

### Admin CRUD Menu Pattern

```lua
local function openPropAdminMenu()
    local props = lib.callback.await('rde_props:getAll', false)
    local options = {
        {
            title    = '➕ Create New Prop',
            icon     = 'plus-circle',
            iconColor = '#10b981',
            onSelect = function() createPropDialog() end,
        }
    }

    for _, prop in ipairs(props) do
        table.insert(options, {
            title       = prop.name,
            description = ('Owner: %s | Category: %s'):format(prop.owner, prop.category),
            icon        = 'cube',
            iconColor   = '#3b82f6',
            metadata    = {
                { label = 'Position', value = ('%.2f, %.2f, %.2f'):format(prop.coords.x, prop.coords.y, prop.coords.z) },
                { label = 'Rotation', value = ('%.2f°'):format(prop.rotation.z) },
                { label = 'Public',   value = prop.public and 'Yes' or 'No' },
            },
            arrow    = true,
            onSelect = function() openPropOptionsMenu(prop) end,
        })
    end

    lib.registerContext({ id = 'rde_props_admin', title = '🛠️ Prop Management', options = options })
    lib.showContext('rde_props_admin')
end
```

---

## 🌍 3. Locales & Config Structure

### config.lua

```lua
Config = {}

Config.Locale   = GetConvar('ox:locale', 'en')
Config.Debug    = false

Config.Icons = {
    success  = 'check-circle',
    error    = 'x-circle',
    warning  = 'alert-triangle',
    info     = 'info',
    police   = 'shield',
    medical  = 'heart-pulse',
    mechanic = 'wrench',
    money    = 'dollar-sign',
    vehicle  = 'car',
    weapon   = 'gun',
}

Config.Colors = {
    primary = 'blue',
    success = 'green',
    error   = 'red',
    warning = 'yellow',
    info    = 'cyan',
}

Config.Performance = {
    updateInterval  = 1000,   -- ms
    maxObjects      = 100,
    renderDistance  = 50.0,
}

return Config
```

### locales/en.lua

```lua
return {
    success            = 'Success',
    error              = 'Error',
    warning            = 'Warning',
    info               = 'Information',
    press_to_interact  = 'Press ~INPUT_CONTEXT~ to interact',
    processing         = 'Processing...',
    prop_placed        = 'Prop placed successfully',
    prop_deleted       = 'Prop deleted',
    prop_not_found     = 'Prop not found',
    no_permission      = 'You do not have permission',
    confirm            = 'Confirm',
    cancel             = 'Cancel',
    close              = 'Close',
    save               = 'Save',
    delete             = 'Delete',
}
```

### locales/de.lua

```lua
return {
    success            = 'Erfolg',
    error              = 'Fehler',
    warning            = 'Warnung',
    info               = 'Information',
    press_to_interact  = 'Drücke ~INPUT_CONTEXT~ zum Interagieren',
    processing         = 'Verarbeitung...',
    prop_placed        = 'Prop erfolgreich platziert',
    prop_deleted       = 'Prop gelöscht',
    prop_not_found     = 'Prop nicht gefunden',
    no_permission      = 'Keine Berechtigung',
    confirm            = 'Bestätigen',
    cancel             = 'Abbrechen',
    close              = 'Schließen',
    save               = 'Speichern',
    delete             = 'Löschen',
}
```

### Using Locales in Code

```lua
local locale = lib.load('locales.' .. Config.Locale)

lib.notify({ description = locale.prop_placed, type = 'success' })
```

---

## 🗄️ 4. Database Best Practices

### Auto-Create Tables on Start

```lua
-- server/database.lua
MySQL.ready(function()
    MySQL.query([[
        CREATE TABLE IF NOT EXISTS `rde_props` (
            `id`         INT AUTO_INCREMENT PRIMARY KEY,
            `name`       VARCHAR(255) NOT NULL,
            `model`      VARCHAR(255) NOT NULL,
            `coords`     JSON         NOT NULL,
            `rotation`   JSON         NOT NULL,
            `owner`      VARCHAR(50)  NOT NULL,
            `category`   VARCHAR(50)  DEFAULT 'misc',
            `public`     BOOLEAN      DEFAULT FALSE,
            `created_at` TIMESTAMP    DEFAULT CURRENT_TIMESTAMP,
            `updated_at` TIMESTAMP    DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
            INDEX `idx_owner`    (`owner`),
            INDEX `idx_category` (`category`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    ]])
    print('^2[RDE] Database tables initialized^0')
end)
```

### Always Use Prepared Statements

```lua
-- ❌ NEVER — SQL injection risk
MySQL.query('SELECT * FROM rde_props WHERE id = ' .. propId)

-- ✅ ALWAYS
MySQL.query('SELECT * FROM rde_props WHERE id = ?', { propId })

-- Multi-param insert
MySQL.query('INSERT INTO rde_props (name, model, coords, owner) VALUES (?, ?, ?, ?)', {
    name, model, json.encode(coords), identifier
})
```

### Async Query Patterns

```lua
-- Avoid blocking await in hot paths — use callbacks where possible
MySQL.query('SELECT * FROM rde_props WHERE owner = ?', { identifier }, function(result)
    if result and #result > 0 then
        -- process
    end
end)

-- For lib.callback (client-server bridge) await is fine
lib.callback.register('rde_props:getAll', function(source)
    local player = Ox.GetPlayer(source)
    if not player then return {} end

    return MySQL.query.await('SELECT * FROM rde_props WHERE owner = ?', {
        player.getIdentifier('license')
    }) or {}
end)
```

---

## 🛡️ 5. Security & Permissions

### Triple-Layer Admin Verification

```lua
-- server/permissions.lua

local function isAdmin(source)
    local player = Ox.GetPlayer(source)
    if not player then return false end

    -- Layer 1: ACE (preferred)
    if IsPlayerAceAllowed(source, 'rde.admin') then return true end

    -- Layer 2: ox_core groups
    local group = player.getGroup('admin')
    if group and group >= 1 then return true end

    -- Layer 3: Steam ID whitelist (fallback)
    local steamId = player.getIdentifier('steam')
    if steamId then
        for _, allowedId in ipairs(Config.AdminSteamIds) do
            if steamId == allowedId then return true end
        end
    end

    return false
end

exports('isAdmin', isAdmin)
```

### Protecting Events

```lua
RegisterNetEvent('rde_props:delete', function(propId)
    local source = source

    if not isAdmin(source) then
        print(('^1[SECURITY] Unauthorized delete attempt by %s^0'):format(GetPlayerName(source)))
        return
    end

    MySQL.query('DELETE FROM rde_props WHERE id = ?', { propId })
    GlobalState['rde_props_' .. propId] = nil
    TriggerClientEvent('rde_props:deleted', -1, propId)
end)
```

### ACE Setup (server.cfg)

```cfg
add_ace group.admin rde.admin allow
add_principal identifier.steam:110000xxxxxxxx group.admin
```

---

## 🎯 6. ox_target Integration

```lua
-- client/target.lua

local function setupPropTarget(prop)
    exports.ox_target:addSphereZone({
        coords = prop.coords,
        radius = 2.0,
        debug  = Config.Debug,
        options = {
            {
                name     = 'rde_prop_use',
                label    = locale.use_prop,
                icon     = 'hand',
                iconColor = Config.Colors.primary,
                distance = 2.0,
                canInteract = function()
                    return not prop.locked or isOwner(prop)
                end,
                onSelect = function()
                    TriggerServerEvent('rde_props:use', prop.id)
                end,
            },
            {
                name      = 'rde_prop_delete',
                label     = locale.delete_prop,
                icon      = 'trash',
                iconColor = '#ef4444',
                distance  = 2.0,
                groups    = { 'admin', 'moderator' },
                onSelect  = function()
                    local confirm = lib.alertDialog({
                        header   = locale.confirm_delete,
                        content  = locale.delete_warning,
                        centered = true,
                        cancel   = true,
                    })
                    if confirm == 'confirm' then
                        TriggerServerEvent('rde_props:delete', prop.id)
                    end
                end,
            },
        }
    })
end
```

---

## ⚡ 7. Performance Optimization

### Thread Rules

```lua
-- ❌ BAD — runs 60+ times per second
CreateThread(function()
    while true do
        Wait(0)
        checkNearbyProps(GetEntityCoords(PlayerPedId()))
    end
end)

-- ✅ GOOD — once per second, uses cache
CreateThread(function()
    while true do
        Wait(1000)
        checkNearbyProps(GetEntityCoords(cache.ped))
    end
end)

-- ✅ BETTER — event-driven, no loop at all
lib.onCache('ped', function(ped)
    setupPedInteractions(ped)
end)
```

### Caching Strategy

```lua
local propsCache      = {}
local lastCacheUpdate = 0
local CACHE_DURATION  = 60000   -- 1 minute

local function getProps()
    local now = GetGameTimer()
    if now - lastCacheUpdate > CACHE_DURATION then
        propsCache      = lib.callback.await('rde_props:getAll', false)
        lastCacheUpdate = now
    end
    return propsCache
end

-- Invalidate on server-side change
RegisterNetEvent('rde_props:updated', function()
    lastCacheUpdate = 0
end)
```

### Native Calls

```lua
-- ❌ BAD — native called 100 times
for i = 1, 100 do
    DoSomething(PlayerPedId())
end

-- ✅ GOOD — cached once
local ped = cache.ped
for i = 1, 100 do
    DoSomething(ped)
end
```

---

## 📁 Complete Resource Template

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
└── web/              ← only if NUI panel needed
    ├── index.html
    ├── script.js
    └── style.css
```

### Minimal Working Example

**server/main.lua**
```lua
local Config = require 'config'

MySQL.ready(function()
    MySQL.query([[
        CREATE TABLE IF NOT EXISTS rde_example (
            id   INT AUTO_INCREMENT PRIMARY KEY,
            data JSON
        )
    ]])
end)

RegisterNetEvent('rde_example:action', function()
    local player = Ox.GetPlayer(source)
    if not player then return end
    print(('Player %s triggered action'):format(player.name))
end)
```

**client/main.lua**
```lua
lib.addCommand('example', { help = 'Example command' }, function()
    lib.notify({ description = 'Example works!', type = 'success' })
end)
```

---

## 📚 Essential Resources

| Resource | Link |
|---|---|
| coxdocs.dev | [https://coxdocs.dev](https://coxdocs.dev) |
| ox_core docs | [https://coxdocs.dev/ox_core](https://coxdocs.dev/ox_core) |
| ox_lib docs | [https://coxdocs.dev/ox_lib](https://coxdocs.dev/ox_lib) |
| ox_inventory docs | [https://coxdocs.dev/ox_inventory](https://coxdocs.dev/ox_inventory) |
| oxmysql | [GitHub](https://github.com/communityox/oxmysql) |
| Icon library | [icons.overextended.dev](https://icons.overextended.dev/) |
| Mantine colors | [mantine.dev/theming/colors](https://mantine.dev/theming/colors/) |
| Pleb Masters (props) | [forge.plebmasters.de](https://forge.plebmasters.de/objects) |
| GTA Native DB | [alloc8or.re](https://alloc8or.re/gta5/nativedb/) |
| FiveM Docs | [docs.fivem.net](https://docs.fivem.net/) |

---

## 📜 License

```
###################################################################################
#                                                                                 #
#      .:: RED DRAGON ELITE (RDE)  -  BLACK FLAG SOURCE LICENSE v6.66 ::.         #
#                                                                                 #
#   PROJECT:    RDE OX DEVELOPMENT STANDARDS v1.0                                 #
#   ARCHITECT:  .:: RDE ⧌ Shin [△ ᛋᛅᚱᛒᛅᚾᛏᛋ ᛒᛁᛏᛅ ▽] ::. | https://rd-elite.com     #
#   ORIGIN:     https://github.com/RedDragonElite                                 #
#                                                                                 #
#   WARNING: THIS CODE IS PROTECTED BY DIGITAL VOODOO AND PURE HATRED FOR LEAKERS #
#                                                                                 #
#   [ THE RULES OF THE GAME ]                                                     #
#                                                                                 #
#   1. // THE "FUCK GREED" PROTOCOL (FREE USE)                                    #
#      You are free to use, edit, and abuse this code on your server.             #
#      Learn from it. Break it. Fix it. That is the hacker way.                   #
#      Cost: 0.00€. If you paid for this, you got scammed by a rat.               #
#                                                                                 #
#   2. // THE TEBEX KILL SWITCH (COMMERCIAL SUICIDE)                              #
#      Listen closely, you parasites:                                             #
#      If I find this script on Tebex, Patreon, or in a paid "Premium Pack":      #
#      > I will DMCA your store into oblivion.                                    #
#      > I will publicly shame your community.                                    #
#      > I hope your server lag spikes to 9999ms every time you blink.            #
#      SELLING FREE WORK IS THEFT. AND I AM THE JUDGE.                            #
#                                                                                 #
#   3. // THE CREDIT OATH                                                         #
#      Keep this header. If you remove my name, you admit you have no skill.      #
#      You can add "Edited by [YourName]", but never erase the original creator.  #
#      Don't be a skid. Respect the architecture.                                 #
#                                                                                 #
#   4. // THE CURSE OF THE COPY-PASTE                                             #
#      These standards exist because copy-paste coders break everything.           #
#      Read every section. Understand it. Then build.                             #
#      Don't come crying to my DMs. RTFM or learn to code.                        #
#                                                                                 #
#   --------------------------------------------------------------------------    #
#   "We build the future on the graves of paid resources."                        #
#   "REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY."                          #
#   --------------------------------------------------------------------------    #
###################################################################################
```

**TL;DR:**
- ✅ Free forever — use it, learn from it, build with it
- ✅ Keep the header — credit where it's due
- ❌ Don't sell it — commercial use = instant DMCA
- ❌ Don't be a skid — read before you build

---

## 🌐 Community & The RDE Ecosystem

| | |
|---|---|
| 🐙 GitHub | [RedDragonElite](https://github.com/RedDragonElite) |
| 🌍 Website | [rd-elite.com](https://rd-elite.com) |
| 🔵 Nostr (RDE) | [RedDragonElite](https://primal.net/p/nprofile1qqsv8km2w8yr0sp7mtk3t44qfw7wmvh8caqpnrd7z6ll6mn9ts03teg9ha4rl) |
| 🔵 Nostr (Shin) | [SerpentsByte](https://primal.net/p/nprofile1qqs8p6u423fappfqrrmxful5kt95hs7d04yr25x88apv7k4vszf4gcqynchct) |
| 🚪 RDE Doors | [rde_doors](https://github.com/RedDragonElite/rde_doors) |
| 🚗 RDE Car Service | [rde_carservice](https://github.com/RedDragonElite/rde_carservice) |
| 🎯 RDE Skills | [rde_skills](https://github.com/RedDragonElite/rde_skills) |
| 🎮 RDE Props | [rde_props](https://github.com/RedDragonElite/rde_props) |
| 🌱 RDE Wild Plants | [rde_wildplants](https://github.com/RedDragonElite/rde_wildplant) |
| 🌍 RDE Weather | [rde_weather](https://github.com/RedDragonElite/rde_weather) |
| 📡 RDE Nostr Log | [rde_nostr_log](https://github.com/RedDragonElite/rde_nostr_log) |

---

<div align="center">

*"We build the future on the graves of paid resources."*

**REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY.**

🐉 Made with 🔥 by [Red Dragon Elite](https://rd-elite.com)

[⬆ Back to Top](#-rde--ox-development-standards-v10)

</div>
