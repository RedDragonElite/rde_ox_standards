# 🐉 RDE | OX Development Standards v2.0

<div align="center">

![Version](https://img.shields.io/badge/version-2.0.0-red?style=for-the-badge&logo=github)
![License](https://img.shields.io/badge/license-RDE%20Black%20Flag%20v6.66-black?style=for-the-badge)
![ox_core](https://img.shields.io/badge/ox__core-Only-blue?style=for-the-badge)
![Free](https://img.shields.io/badge/price-FREE%20FOREVER-brightgreen?style=for-the-badge)
![Updated](https://img.shields.io/badge/updated-2026-orange?style=for-the-badge)

**The internal development standard behind every RDE resource.**
Battle-tested in production. Zero compromises. Zero paid bullshit.

*Built by [Red Dragon Elite](https://rd-elite.com) | SerpentsByte*

> ⚡ **v2.0 CHANGELOG:** Overextended officially resumed development in 2026.
> Official docs live at **[overextended.dev](https://overextended.dev)**.
> All patterns updated from production-proven `rde_props` v1.0.1 source.

</div>

---

## 📖 Table of Contents

- [Philosophy](#-philosophy)
- [Core Principles](#-core-principles)
- [Technology Stack](#-technology-stack)
- [Overextended Status 2026](#-overextended-status-2026)
- [fxmanifest Template](#-fxmanifest-template)
- [server.cfg Start Order](#-servercfg-start-order)
- [Module Architecture](#-module-architecture)
- [Config & Locales Pattern](#-config--locales-pattern)
- [Statebags](#-statebags-for-realtime-sync)
- [StateBag Sync — The Right Way](#-statebag-sync--the-right-way)
- [ox_lib UI](#-ox_lib-ui-masterclass)
- [Database Best Practices](#-database-best-practices)
- [Security & Permissions](#-security--permissions)
- [ox_target Integration](#-ox_target-integration)
- [Performance Optimization](#-performance-optimization)
- [Anti-Patterns Hall of Shame](#-anti-patterns-hall-of-shame)
- [Complete Resource Template](#-complete-resource-template)
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

### 5. Multilanguage Built Into Config
Every resource ships with English and German. Locales live **inside `Config.Locales`** as nested tables — not separate files. `Config.GetString(key, ...)` handles everything. See [Config & Locales Pattern](#-config--locales-pattern).

### 6. Bulletproof Security
Server-side validation on EVERY action. Triple-layer admin verification — ACE + ox_core groups + Steam ID. No client-authoritative logic for anything that matters. Rate limiting on placements, notifications, and all destructive events.

### 7. Zero Config to Start
`oxmysql` tables auto-create with `rde_` prefix via `MySQL.query` inside `onResourceStart`. Sensible defaults in `config.lua`. Works out of the box. Easy to customize.

### 8. No Duplicate Broadcasts
**One sync path per state change.** If `UpdateStatebag()` already fires `TriggerClientEvent` to `-1`, you do NOT fire another `TriggerClientEvent` for the same data. This was a production bug in early RDE code — see [Anti-Patterns Hall of Shame](#-anti-patterns-hall-of-shame).

### 9. `Wait()` Never Inside NetEvent Handlers
Blocking a NetEvent handler with `Wait()` stalls the server Lua thread. Any async work (reload loops, retries, timed sequences) wraps in `CreateThread(function() ... end)` immediately. See [Anti-Patterns Hall of Shame](#-anti-patterns-hall-of-shame).

---

## 🔧 Technology Stack

```
✅ ox_core      — Player, character, group management
✅ ox_lib       — UI, callbacks, commands, utilities
✅ ox_inventory — Items, weapons, stashes
✅ ox_target    — World interactions
✅ oxmysql      — Database layer (community-maintained, stable)

❌ QBCore
❌ ESX
❌ Any legacy framework
❌ Polling loops where statebags can do the job
❌ client-authoritative checks for anything sensitive
```

---

## ⚡ Overextended Status 2026

Overextended was established in 2021 as a collaborative effort to build high-quality, secure, free, and open-source FiveM resources. While the project was officially discontinued in 2025, community members continued maintaining oxmysql, ox_lib, ox_inventory, and ox_core. **In 2026, official development resumed** with a renewed focus on community collaboration and delivering critical security updates — though without a return to frequent large-scale feature updates.

**What this means for RDE:**
- Official docs: **[overextended.dev](https://overextended.dev)**
  - ox_core: `overextended.dev/ox_core`
  - ox_lib: `overextended.dev/ox_lib`
  - ox_inventory: `overextended.dev/ox_inventory`
- The stack is **actively maintained and security-patched**. Keep dependencies updated.
- Build targets: `MariaDB 11.4+`, `ox_lib` latest, `ox_core` latest, `oxmysql` latest.

---

## 📄 fxmanifest Template

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

name        'rde_yourscript'
author      'Red Dragon Elite | SerpentsByte'
description 'Description of your script'
version     '1.0.0'

dependencies {
    '/server:7290',  -- minimum FiveM artifact build
    'oxmysql',
    'ox_lib',
    'ox_core',
    'ox_target',     -- only if using world interactions
    'ox_inventory',  -- only if using items or weapons
}

shared_scripts {
    '@ox_lib/init.lua',
    '@ox_core/lib/init.lua',
    'config.lua',       -- Config table (includes locales)
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',
    'server/*.lua'
}

client_scripts {
    'data/items.lua',   -- only if using item definitions
    'client/*.lua'
}

-- Only if using a NUI panel:
-- ui_page 'web/index.html'
-- files { 'web/**/*' }
```

**Key rules:**
- `lua54 'yes'` always — we use modern Lua.
- `@ox_lib/init.lua` and `@ox_core/lib/init.lua` are **shared**, not client-only.
- `@oxmysql/lib/MySQL.lua` is **server-only**.
- List only dependencies the script actually uses. Don't include `ox_target` or `ox_inventory` if unused.

---

## 📋 server.cfg Start Order

```cfg
# Always in this exact order — never deviate
ensure oxmysql
ensure ox_lib
ensure ox_core
ensure ox_target
ensure ox_inventory

# RDE scripts after ALL dependencies
ensure rde_yourscript
```

---

## 🏗️ Module Architecture

Every RDE resource uses the same internal structure:

```
rde_yourscript/
├── fxmanifest.lua
├── config.lua          ← ALL config + ALL locales in one file
├── README.md
├── LICENSE
├── data/
│   └── items.lua       ← ox_inventory item definitions (client-side)
├── server/
│   ├── server.lua      ← main events, commands, startup
│   └── (callbacks.lua) ← lib.callback.register() handlers if many
└── client/
    ├── client.lua      ← main logic, placement, state
    └── (target.lua)    ← ox_target zones if complex
```

**Internal State pattern (both client and server):**

```lua
-- Always use a single State table — no scattered globals
local State = {
    props       = {},       -- keyed by propId
    playerProps = {},       -- keyed by identifier -> { propId = true }
    ready       = false,    -- set true after DB load
    -- client additions:
    player      = { identifier = nil, isAdmin = false },
    entities    = {},       -- propId -> entity handle
    zones       = {},       -- propId -> ox_target zone name
    placing     = false,
}
```

**Internal logging:**

```lua
local function Log(msg, level)
    if not Config.Debug and level ~= 'ERROR' then return end
    local prefix = level == 'ERROR' and '^1' or level == 'WARN' and '^3' or '^2'
    print(string.format('%s[RDE %s]^7 %s', prefix, GetCurrentResourceName():upper(), msg))
end
```

---

## 🌍 Config & Locales Pattern

**v2.0 standard: Locales live inside config.lua as `Config.Locales`.** No separate files. One file to rule them all.

```lua
-- config.lua
local Config = {}

-- ============================================
-- 🐛 DEBUG
-- ============================================
Config.Debug = false

-- ============================================
-- 🔐 SECURITY
-- ============================================
Config.AllowAcePermissions = true

Config.AdminGroups = {
    ['admin']      = true,
    ['superadmin'] = true,
    ['moderator']  = true,
    ['owner']      = true,
}

-- ============================================
-- 🗄️ DATABASE
-- ============================================
Config.DatabaseTable  = 'rde_yourscript'
Config.StatebagPrefix = 'rde_ys_'   -- keep short, keep unique per resource

-- ============================================
-- 🎯 GAMEPLAY LIMITS
-- ============================================
Config.MaxPerPlayer  = 50
Config.AdminLimit    = 500
Config.RenderDistance = 300.0

-- ============================================
-- 📊 PERFORMANCE
-- ============================================
Config.Performance = {
    updateInterval         = 1000,
    maxVisibleObjects      = 200,
    garbageCollectInterval = 300000,  -- 5 min
}

-- ============================================
-- 🎨 COLORS (for UI elements and ox_target)
-- ============================================
Config.Colors = {
    primary = { r = 139, g = 92,  b = 246 },  -- violet / RDE brand
    success = { r = 34,  g = 197, b = 94  },
    error   = { r = 239, g = 68,  b = 68  },
    warning = { r = 245, g = 158, b = 11  },
    info    = { r = 59,  g = 130, b = 246 },
    admin   = { r = 255, g = 215, b = 0   },
}

-- Hex variants for ox_lib UI
Config.HexColors = {
    primary = '#8b5cf6',
    success = '#22c55e',
    error   = '#ef4444',
    warning = '#f59e0b',
    info    = '#3b82f6',
    admin   = '#fbbf24',
}

-- ============================================
-- 🌍 LOCALES (all languages in one block)
-- ============================================
Config.DefaultLanguage = 'en'

Config.Locales = {
    en = {
        -- Generic
        success        = 'Success',
        error          = 'Error',
        warning        = 'Warning',
        no_permission  = 'No permission',
        processing     = 'Processing...',
        confirm        = 'Confirm',
        cancel         = 'Cancel',
        -- Resource-specific (add yours here)
        item_placed    = 'Item placed: **%s**',
        item_deleted   = 'Item deleted',
        limit_reached  = 'Limit reached (%d)',
        cooldown       = 'Wait %ds before placing again',
    },
    de = {
        -- Generic
        success        = 'Erfolg',
        error          = 'Fehler',
        warning        = 'Warnung',
        no_permission  = 'Keine Berechtigung',
        processing     = 'Verarbeite...',
        confirm        = 'Bestätigen',
        cancel         = 'Abbrechen',
        -- Resource-specific
        item_placed    = 'Item platziert: **%s**',
        item_deleted   = 'Item gelöscht',
        limit_reached  = 'Limit erreicht (%d)',
        cooldown       = 'Warte %ds vor der nächsten Platzierung',
    },
}

-- ============================================
-- 🛠️ HELPER: Locale string getter
-- ============================================
function Config.GetString(key, ...)
    local lang = Config.Locales[Config.DefaultLanguage] or Config.Locales['en']
    local str  = lang[key] or key
    if ... then return string.format(str, ...) end
    return str
end

return Config
```

**Usage:**
```lua
local Config = require 'config'

-- Simple
lib.notify({ description = Config.GetString('success'), type = 'success' })

-- With format args
lib.notify({ description = Config.GetString('item_placed', itemName), type = 'success' })
```

---

## 📡 Statebags for Realtime Sync

Statebags are the backbone of all RDE realtime data. No polling, no lag, no sync issues.

### GlobalState for World Objects

```lua
-- Server: broadcast prop to ALL clients
local key = Config.StatebagPrefix .. propId
GlobalState[key] = {
    id        = propId,
    model     = 'prop_chair_01',
    position  = { x = 0.0, y = 0.0, z = 0.0 },
    rotation  = { x = 0.0, y = 0.0, z = 0.0 },
    collision = true,
    createdBy = identifier,
    isAdmin   = false,
}

-- Delete: first tombstone, then nil after delay
GlobalState[key] = { _deleted = true }
SetTimeout(1000, function()
    GlobalState[key] = nil
end)
```

```lua
-- Client: react to statebag changes
AddStateBagChangeHandler(Config.StatebagPrefix, nil, function(bagName, key, value)
    if not value or value._deleted then
        -- handle removal
        local propId = key:gsub(Config.StatebagPrefix, '')
        DeletePropEntity(propId)
    else
        SpawnPropEntity(value)
    end
end)
```

### Player State (Server → All Clients)

```lua
local PlayerState = Player(source).state
PlayerState:set('wantedLevel', 3, true)   -- true = replicate to all clients
PlayerState:set('jobGrade', player.getGroup('police'), true)
```

### Reacting to Player State Changes (Client)

```lua
lib.onCache('ped', function(ped)
    -- fires whenever the local ped changes (model change, respawn, etc.)
    SetupInteractions(ped)
end)
```

---

## 🔄 StateBag Sync — The Right Way

This is the most common source of production bugs in RDE code. Follow this exactly.

### The Golden Rule: One Sync Path

```lua
-- ✅ CORRECT pattern (from rde_props v1.0.1 production)
local function UpdateStatebag(id, data)
    local key = Config.StatebagPrefix .. id
    if data then
        GlobalState[key] = data
        TriggerClientEvent('rde_ys:statebagUpdate', -1, id, data)
    else
        GlobalState[key] = { _deleted = true }
        TriggerClientEvent('rde_ys:statebagDelete', -1, id)
        SetTimeout(1000, function()
            GlobalState[key] = nil
        end)
    end
end

-- ✅ Caller just calls UpdateStatebag — nothing else
RegisterNetEvent('rde_ys:toggleCollision', function(itemId)
    local source = source
    -- ... validation ...
    prop.collision = not prop.collision
    UpdateStatebag(itemId, prop)
    -- NO second TriggerClientEvent here. UpdateStatebag already broadcasts.
end)
```

```lua
-- ❌ WRONG — double broadcast, double entity rebuild on every client
local function UpdateStatebag(id, data)
    GlobalState[Config.StatebagPrefix .. id] = data
    TriggerClientEvent('rde_ys:statebagUpdate', -1, id, data)  -- fires once
end

RegisterNetEvent('rde_ys:toggleCollision', function(itemId)
    -- ...
    UpdateStatebag(itemId, prop)
    TriggerClientEvent('rde_ys:statebagUpdate', -1, itemId, prop)  -- fires AGAIN — BUG
end)
```

### Notification Dedup (Rate Limiter)

Prevent spam notifications from rapid-fire events:

```lua
local NotificationCache = {}

local function Notify(source, title, description, type)
    if source == 0 then return end
    local hash = string.format('%d:%s:%s', source, title, description)
    local now  = GetGameTimer()
    if NotificationCache[hash] and (now - NotificationCache[hash] < 2000) then return end
    NotificationCache[hash] = now
    TriggerClientEvent('ox_lib:notify', source, {
        title       = title,
        description = description,
        type        = type,
    })
end
```

---

## 🎨 ox_lib UI Masterclass

### Notifications

```lua
lib.notify({
    title       = '✅ Success',
    description = 'Item purchased!',
    type        = 'success',   -- 'success' | 'error' | 'info' | 'warning'
    icon        = 'shopping-cart',  -- Lucide icon name
    iconColor   = Config.HexColors.success,
    duration    = 5000,
    position    = 'top-right',
})
```

### TextUI (Immersive HUD)

```lua
-- Show
lib.showTextUI('[🎯 ENTER] → Place | [❌ ESC] → Cancel | [← →] → Rotate Z', {
    position  = 'bottom-center',
    icon      = 'gamepad',
    style     = {
        borderRadius    = '12px',
        backgroundColor = 'rgba(17, 24, 39, 0.95)',
        color           = 'white',
        padding         = '16px 24px',
        border          = '2px solid rgba(139, 92, 246, 0.3)',
        boxShadow       = '0 10px 30px rgba(0, 0, 0, 0.5)',
    },
})

-- Hide
lib.hideTextUI()
```

### Progress Bar (with animation)

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
    },
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
    id      = 'rde_example_menu',
    title   = '🛠️ Example Menu',
    options = {
        {
            title       = 'Action One',
            description = 'Does something',
            icon        = 'zap',
            iconColor   = Config.HexColors.primary,
            onSelect    = function()
                TriggerServerEvent('rde_example:actionOne')
            end,
        },
        {
            title       = 'Open Inventory',
            description = 'Access storage',
            icon        = 'box',
            iconColor   = Config.HexColors.info,
            arrow       = true,
            onSelect    = function()
                exports.ox_inventory:openInventory('stash', 'stash_id')
            end,
        },
    },
})
lib.showContext('rde_example_menu')
```

### Input Dialog

```lua
local input = lib.inputDialog('Create Item', {
    { type = 'input',    label = 'Name',     required = true,  icon = 'tag' },
    { type = 'select',   label = 'Category',
      options = {
          { value = 'furniture',  label = 'Furniture' },
          { value = 'misc',       label = 'Misc'      },
      },
      default = 'misc',
    },
    { type = 'number',   label = 'Price',    icon = 'dollar-sign', required = true, min = 0, max = 1000000 },
    { type = 'checkbox', label = 'Permanent', checked = true },
})

if not input then return end  -- cancelled
local name, category, price, permanent = input[1], input[2], input[3], input[4]
```

### Alert Dialog (Confirm/Cancel)

```lua
local result = lib.alertDialog({
    header   = '🗑️ Delete Item?',
    content  = 'This cannot be undone.',
    centered = true,
    cancel   = true,
    labels   = { confirm = 'Yes, delete', cancel = 'Cancel' },
})

if result == 'confirm' then
    TriggerServerEvent('rde_example:delete', itemId)
end
```

### Skill Check

```lua
-- Single check
if lib.skillCheck({ 'easy', 'easy', 'medium' }, { 'w', 'a', 's', 'd' }) then
    -- success
end

-- Chain (used in lock-crack flows)
local success = true
for i = 1, 3 do
    if not lib.skillCheck({ 'easy', 'medium' }, { 'w', 'a', 's', 'd' }) then
        success = false
        break
    end
end
```

### Callbacks

```lua
-- Server registration
lib.callback.register('rde_example:getData', function(source)
    local player = Ox.GetPlayer(source)
    if not player then return nil end
    return MySQL.query.await('SELECT * FROM rde_example WHERE owner = ?', {
        player.stateId or tostring(player.charId)
    }) or {}
end)

-- Client call (await)
local data = lib.callback.await('rde_example:getData', false)
if data then
    -- use data
end
```

---

## 🗄️ Database Best Practices

### Auto-Create Table on Resource Start

```lua
-- server/server.lua
AddEventHandler('onResourceStart', function(name)
    if name ~= GetCurrentResourceName() then return end
    -- Wait for ox_core if needed
    local attempts = 0
    while not Ox and attempts < 100 do Wait(100); attempts = attempts + 1 end
    if not Ox then Log('ox_core not found!', 'ERROR'); return end
    SetupDatabase()
end)

local function SetupDatabase()
    MySQL.query([[
        CREATE TABLE IF NOT EXISTS `rde_example` (
            `id`         VARCHAR(64)   PRIMARY KEY,
            `model`      VARCHAR(128)  NOT NULL,
            `name`       VARCHAR(128)  NOT NULL,
            `position`   JSON          NOT NULL,
            `rotation`   JSON          NOT NULL,
            `collision`  TINYINT(1)    DEFAULT 1,
            `permanent`  TINYINT(1)    DEFAULT 1,
            `created_by` VARCHAR(64)   NOT NULL,
            `is_admin`   TINYINT(1)    DEFAULT 0,
            `created_at` TIMESTAMP     DEFAULT CURRENT_TIMESTAMP,
            INDEX `idx_created_by` (`created_by`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    ]], function()
        LoadAll(function()
            -- notify all online players after load
            for _, playerId in ipairs(GetPlayers()) do
                TriggerEvent('rde_example:init', tonumber(playerId))
            end
        end)
    end)
end
```

### Always Use Prepared Statements

```lua
-- ❌ NEVER — SQL injection
MySQL.query('SELECT * FROM rde_example WHERE id = ' .. id)

-- ✅ ALWAYS — prepared
MySQL.query('SELECT * FROM rde_example WHERE id = ?', { id })

-- Multi-param insert
MySQL.query(
    'INSERT INTO rde_example (id, model, name, position, created_by) VALUES (?, ?, ?, ?, ?)',
    { id, model, name, json.encode(coords), identifier }
)
```

### TINYINT for Booleans + Helper Functions

```lua
-- DB stores 0/1, Lua uses true/false — always convert at the boundary
local function DbBool(value)
    return (value == true or value == 1) and 1 or 0
end

local function BoolDb(value)
    return value == 1 or value == '1' or value == true
end

-- Write
MySQL.query('UPDATE rde_example SET collision = ? WHERE id = ?', { DbBool(prop.collision), id })

-- Read
prop.collision = BoolDb(row.collision)
```

### Async Patterns

```lua
-- Callbacks: await is fine (it's inside lib.callback which handles coroutines)
lib.callback.register('rde_example:getAll', function(source)
    return MySQL.query.await('SELECT * FROM rde_example WHERE created_by = ?', { identifier }) or {}
end)

-- Hot paths (event handlers, startup loops): use callback form, not .await
MySQL.query('SELECT * FROM rde_example', {}, function(result)
    if not result or #result == 0 then return end
    for _, row in ipairs(result) do
        -- process
    end
end)
```

### ID Generation

```lua
-- Use time+random hex for string primary keys (no AUTO_INCREMENT collision on reload)
local function GenerateId()
    return string.format('rde_%x%x', os.time(), math.random(100000, 999999))
end
```

---

## 🛡️ Security & Permissions

### GetIdentifier — Always From ox_core

```lua
local function GetIdentifier(source)
    local player = Ox.GetPlayer(source)
    if not player then return nil end
    -- ox_core stores charId (integer) — convert to string for DB
    if player.stateId then return player.stateId end
    if player.charId  then return tostring(player.charId) end
    if player.userId  then return tostring(player.userId) end
    -- Fallback (should never be needed on a clean ox_core setup)
    for _, id in ipairs(GetPlayerIdentifiers(source)) do
        if id:find('license:') then return id end
    end
    return nil
end
```

### Triple-Layer Admin Verification

```lua
local function IsAdmin(source)
    if not source or source == 0 then return true end  -- console always admin

    -- Layer 1: ACE permissions (fastest, most reliable)
    if Config.AllowAcePermissions then
        if IsPlayerAceAllowed(source, 'command') then return true end
        if IsPlayerAceAllowed(source, 'admin')   then return true end
    end

    -- Layer 2: ox_core groups
    local player = Ox.GetPlayer(source)
    if not player then return false end

    if player.hasPermission and player.hasPermission('admin') then return true end

    if player.getGroups then
        local groups = player.getGroups()
        for groupName in pairs(groups) do
            if Config.AdminGroups[groupName] then return true end
        end
    end

    return false
end

exports('isAdmin', IsAdmin)
```

### Protecting Events — Always Validate First

```lua
RegisterNetEvent('rde_example:delete', function(itemId)
    local source = source
    local item   = State.items[itemId]
    if not item then return end  -- unknown item

    local identifier = GetIdentifier(source)
    local isAdmin    = IsAdmin(source)

    -- Ownership check
    if item.createdBy ~= identifier and not isAdmin then
        Log(string.format('Unauthorized delete attempt by %s (player %d)', identifier, source), 'WARN')
        Notify(source, '❌ Error', Config.GetString('no_permission'), 'error')
        return
    end

    -- Proceed
    MySQL.query.await('DELETE FROM rde_example WHERE id = ?', { itemId })
    State.items[itemId] = nil
    UpdateStatebag(itemId, nil)
    Notify(source, '✅ Success', Config.GetString('item_deleted'), 'success')
end)
```

### Placement Cooldown (Anti-Spam)

```lua
local PlacementLocks    = {}
local PLACEMENT_COOLDOWN = 2000  -- ms

local function CanPlace(source)
    local now  = GetGameTimer()
    local last = PlacementLocks[source]
    if last and (now - last) < PLACEMENT_COOLDOWN then
        local remaining = math.ceil((PLACEMENT_COOLDOWN - (now - last)) / 1000)
        return false, remaining
    end
    return true, 0
end

local function LockPlacement(source)
    PlacementLocks[source] = GetGameTimer()
end

-- Cleanup thread (every 5 minutes)
CreateThread(function()
    while true do
        Wait(300000)
        local now    = GetGameTimer()
        local cutoff = now - 600000
        for src, ts in pairs(PlacementLocks) do
            if ts < cutoff then PlacementLocks[src] = nil end
        end
    end
end)
```

### ACE Setup (server.cfg)

```cfg
add_ace group.admin  rde.admin  allow
add_ace group.admin  command    allow
add_principal identifier.steam:110000xxxxxxxx group.admin
```

---

## 🎯 ox_target Integration

### Sphere Zone Per Object (Standard Pattern)

```lua
-- client/client.lua or client/target.lua

local function BuildTargetOptions(itemId, itemData)
    local options = {}

    -- Info option (always visible)
    table.insert(options, {
        name      = 'item_info',
        icon      = 'fa-solid fa-info-circle',
        iconColor = Config.HexColors.info,
        label     = 'Information',
        onSelect  = function()
            local fresh = State.items[itemId]
            if not fresh then return end
            lib.notify({
                title       = '📦 Item Info',
                description = string.format('**Model:** %s\n**Owner:** %s', fresh.model, fresh.createdBy or 'Unknown'),
                type        = 'info',
                duration    = 8000,
            })
        end,
    })

    local isOwner   = itemData.createdBy == State.player.identifier
    local canManage = State.player.isAdmin or isOwner

    if canManage then
        -- Delete option
        table.insert(options, {
            name      = 'item_delete',
            icon      = 'fa-solid fa-trash',
            iconColor = Config.HexColors.error,
            label     = 'Delete',
            onSelect  = function()
                local result = lib.alertDialog({
                    header   = '🗑️ Delete Item?',
                    content  = 'This cannot be undone.',
                    centered = true,
                    cancel   = true,
                    labels   = { confirm = 'Yes, delete', cancel = 'Cancel' },
                })
                if result == 'confirm' then
                    TriggerServerEvent('rde_example:delete', itemId)
                end
            end,
        })
    end

    return options
end

local function CreateZone(itemId, entity, itemData)
    -- Remove existing zone first (idempotent)
    if State.zones[itemId] then
        pcall(function() exports.ox_target:removeZone(State.zones[itemId]) end)
        State.zones[itemId] = nil
    end

    local zoneName = ('rde_example_%s'):format(itemId)
    local ok = pcall(function()
        exports.ox_target:addSphereZone({
            coords  = GetEntityCoords(entity),
            radius  = Config.TargetDistance or 2.0,
            debug   = Config.Debug,
            options = BuildTargetOptions(itemId, itemData),
        })
    end)

    if ok then
        State.zones[itemId] = zoneName
    end
end

local function RemoveZone(itemId)
    if not State.zones[itemId] then return end
    pcall(function() exports.ox_target:removeZone(State.zones[itemId]) end)
    State.zones[itemId] = nil
end
```

**Key rules:**
- Always `pcall` around `addSphereZone` / `removeZone` — ox_target zone ops can fail silently.
- Always remove old zone before creating new one — never stack zones on the same entity.
- Zone names must be unique per entity. Use `resourcename_itemId` pattern.

---

## ⚡ Performance Optimization

### Thread Rules

```lua
-- ❌ BAD — runs 60+ times per second, no yield
CreateThread(function()
    while true do
        Wait(0)
        checkNearby(GetEntityCoords(PlayerPedId()))
    end
end)

-- ✅ GOOD — once per second, cached ped
CreateThread(function()
    while true do
        Wait(1000)
        checkNearby(GetEntityCoords(cache.ped))
    end
end)

-- ✅ BETTER — event-driven, no loop at all
lib.onCache('ped', function(ped)
    SetupInteractions(ped)
end)
```

### The ox_lib Cache Object

`cache` is provided by ox_lib and is always current without a native call:

```lua
cache.ped        -- local player ped handle
cache.vehicle    -- current vehicle (nil if on foot)
cache.seat       -- current seat index
cache.playerId   -- server-side player ID
cache.serverId   -- same as playerId

-- ✅ Use cache instead of native calls in loops
local ped = cache.ped       -- not PlayerPedId()
local coords = GetEntityCoords(ped)
```

### Cache Invalidation on Server Changes

```lua
-- Server fires update → client invalidates local cache
local localCache      = {}
local cacheTimestamp  = 0
local CACHE_TTL       = 60000

local function GetCachedData()
    local now = GetGameTimer()
    if now - cacheTimestamp > CACHE_TTL then
        localCache     = lib.callback.await('rde_example:getAll', false) or {}
        cacheTimestamp = now
    end
    return localCache
end

-- Server triggers this after any mutation
RegisterNetEvent('rde_example:invalidateCache', function()
    cacheTimestamp = 0
end)
```

### Model Loading — Always With Timeout Guard

```lua
local function LoadModel(model)
    local hash = type(model) == 'string' and joaat(model) or model
    if not IsModelValid(hash) then
        Log(('Invalid model: %s'):format(tostring(model)), 'ERROR')
        return false
    end
    if HasModelLoaded(hash) then return true end
    RequestModel(hash)
    local timeout = GetGameTimer() + 10000
    while not HasModelLoaded(hash) and GetGameTimer() < timeout do
        Wait(10)
    end
    if not HasModelLoaded(hash) then
        Log(('Model load timeout: %s'):format(tostring(model)), 'ERROR')
        return false
    end
    return true
end
```

### Render Distance Culling

```lua
-- Only render entities within Config.RenderDistance
CreateThread(function()
    while true do
        Wait(Config.Performance.updateInterval)
        local playerCoords = GetEntityCoords(cache.ped)

        for itemId, data in pairs(State.items) do
            local dist = #(playerCoords - vector3(data.position.x, data.position.y, data.position.z))
            if dist <= Config.RenderDistance then
                SpawnEntityIfNeeded(itemId, data)
            else
                DespawnEntityIfNeeded(itemId)
            end
        end
    end
end)
```

### Native Call Caching in Loops

```lua
-- ❌ BAD — native called on every iteration
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

## ☠️ Anti-Patterns Hall of Shame

These are production bugs caught in real RDE code. Never repeat them.

### 1. The Double Broadcast Bug

```lua
-- ❌ BANNED — UpdateStatebag already fires TriggerClientEvent(-1, ...)
-- Calling it again causes every client to rebuild entities TWICE.
UpdateStatebag(id, data)
TriggerClientEvent('rde_example:update', -1, id, data)  -- DUPLICATE. REMOVE.
```

### 2. Wait() Inside a NetEvent Handler

```lua
-- ❌ BANNED — blocks the server Lua coroutine, can freeze all net events
RegisterNetEvent('rde_example:reload', function()
    local source = source
    Wait(500)         -- ← BLOCKS THE THREAD
    LoadAll()
end)

-- ✅ CORRECT — spawn a thread immediately, handler returns
RegisterNetEvent('rde_example:reload', function()
    local source = source
    CreateThread(function()
        Wait(500)
        LoadAll()
    end)
end)
```

### 3. The `or true` Logic Bomb

```lua
-- ❌ BANNED — condition always true, security check bypassed
if not item or true then return end          -- 'or true' makes this always skip
if item.collision == false or true then ...  -- collision is always "true"

-- ✅ Validate your logic: if you see `or true` anywhere, delete it immediately
if not item then return end
if not item.collision then ... end
```

### 4. Duplicate statebag set + nil in same tick

```lua
-- ❌ WRONG — sets nil immediately, clients never see the tombstone
GlobalState[key] = { _deleted = true }
GlobalState[key] = nil   -- ← kills the tombstone before clients process it

-- ✅ CORRECT — tombstone first, nil after delay
GlobalState[key] = { _deleted = true }
SetTimeout(1000, function()
    GlobalState[key] = nil
end)
```

### 5. File-Scope State That Conflicts With Require Cache

```lua
-- ❌ WRONG in shared/config files — local at file scope gets cached by require()
-- If two resources require the same module, they share this table
local sharedBugTable = {}

-- ✅ Always return a fresh table from config modules
local Config = {}
-- ... populate ...
return Config
```

### 6. Polling Instead of Statebags

```lua
-- ❌ BANNED — polling loop for data that could be a statebag
CreateThread(function()
    while true do
        Wait(500)
        local data = lib.callback.await('rde_example:getData', false)
        UpdateUI(data)
    end
end)

-- ✅ CORRECT — statebag change handler, fires only when data actually changes
AddStateBagChangeHandler(Config.StatebagPrefix, nil, function(bagName, key, value)
    UpdateUI(value)
end)
```

---

## 📁 Complete Resource Template

### Folder Structure

```
rde_yourscript/
├── fxmanifest.lua
├── config.lua          ← Config + ALL locales
├── README.md
├── LICENSE
├── data/
│   └── items.lua       ← ox_inventory item defs (client script)
├── server/
│   └── server.lua      ← events, callbacks, startup, commands
└── client/
    └── client.lua      ← main logic, UI, statebag handlers
```

### server/server.lua — Minimal Working Skeleton

```lua
--[[
╔════════════════════════════════════╗
║  rde_yourscript — SERVER v1.0.0    ║
╚════════════════════════════════════╝
]]

local Config = require 'config'

local State = {
    items       = {},
    playerItems = {},
    ready       = false,
}

-- Utilities
local function Log(msg, level)
    if not Config.Debug and level ~= 'ERROR' then return end
    local prefix = level == 'ERROR' and '^1' or '^2'
    print(string.format('%s[RDE YourScript SERVER]^7 %s', prefix, msg))
end

local function DbBool(v)  return (v == true or v == 1) and 1 or 0 end
local function BoolDb(v)  return v == 1 or v == '1' or v == true  end

local function GenerateId()
    return string.format('rde_%x%x', os.time(), math.random(100000, 999999))
end

local function GetIdentifier(source)
    local player = Ox.GetPlayer(source)
    if not player then return nil end
    if player.stateId then return player.stateId end
    if player.charId  then return tostring(player.charId) end
    return nil
end

local function IsAdmin(source)
    if not source or source == 0 then return true end
    if Config.AllowAcePermissions then
        if IsPlayerAceAllowed(source, 'command') then return true end
        if IsPlayerAceAllowed(source, 'admin')   then return true end
    end
    local player = Ox.GetPlayer(source)
    if not player then return false end
    if player.getGroups then
        for groupName in pairs(player.getGroups()) do
            if Config.AdminGroups[groupName] then return true end
        end
    end
    return false
end

-- Statebag
local function UpdateStatebag(id, data)
    local key = Config.StatebagPrefix .. id
    if data then
        GlobalState[key] = data
        TriggerClientEvent('rde_ys:statebagUpdate', -1, id, data)
    else
        GlobalState[key] = { _deleted = true }
        TriggerClientEvent('rde_ys:statebagDelete', -1, id)
        SetTimeout(1000, function() GlobalState[key] = nil end)
    end
end

-- Notifications (dedup)
local NotifCache = {}
local function Notify(source, title, desc, type)
    if source == 0 then return end
    local hash = ('%d:%s:%s'):format(source, title, desc)
    local now  = GetGameTimer()
    if NotifCache[hash] and (now - NotifCache[hash] < 2000) then return end
    NotifCache[hash] = now
    TriggerClientEvent('ox_lib:notify', source, { title = title, description = desc, type = type })
end

-- Database
local function SetupDatabase()
    MySQL.query([[
        CREATE TABLE IF NOT EXISTS `rde_yourscript` (
            `id`         VARCHAR(64)  PRIMARY KEY,
            `model`      VARCHAR(128) NOT NULL,
            `name`       VARCHAR(128) NOT NULL,
            `position`   JSON         NOT NULL,
            `rotation`   JSON         NOT NULL,
            `created_by` VARCHAR(64)  NOT NULL,
            `is_admin`   TINYINT(1)   DEFAULT 0,
            `created_at` TIMESTAMP    DEFAULT CURRENT_TIMESTAMP,
            INDEX `idx_created_by` (`created_by`)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
    ]], function()
        MySQL.query('SELECT * FROM rde_yourscript', {}, function(result)
            for _, row in ipairs(result or {}) do
                local pos = json.decode(row.position)
                local rot = json.decode(row.rotation)
                if pos and rot then
                    local data = {
                        id        = row.id,
                        model     = row.model,
                        name      = row.name,
                        position  = pos,
                        rotation  = rot,
                        createdBy = row.created_by,
                        isAdmin   = BoolDb(row.is_admin),
                    }
                    State.items[row.id] = data
                    if not State.playerItems[row.created_by] then
                        State.playerItems[row.created_by] = {}
                    end
                    State.playerItems[row.created_by][row.id] = true
                    UpdateStatebag(row.id, data)
                end
            end
            State.ready = true
            Log(('Loaded %d items'):format(#(result or {})), 'INFO')
        end)
    end)
end

-- Events
RegisterNetEvent('rde_ys:init', function()
    local source = source
    while not State.ready do Wait(100) end
    local identifier = GetIdentifier(source)
    if not identifier then return end
    TriggerClientEvent('rde_ys:setPlayer', source, {
        identifier = identifier,
        isAdmin    = IsAdmin(source),
    })
end)

-- Startup
AddEventHandler('onResourceStart', function(name)
    if name ~= GetCurrentResourceName() then return end
    local attempts = 0
    while not Ox and attempts < 100 do Wait(100); attempts = attempts + 1 end
    if not Ox then Log('ox_core not found!', 'ERROR'); return end
    SetupDatabase()
end)

-- Shutdown cleanup
AddEventHandler('onResourceStop', function(name)
    if name ~= GetCurrentResourceName() then return end
    for key in pairs(GlobalState) do
        if key:find(Config.StatebagPrefix) then GlobalState[key] = nil end
    end
end)

-- Cleanup thread
CreateThread(function()
    while true do
        Wait(300000)
        local now = GetGameTimer()
        for hash, ts in pairs(NotifCache) do
            if ts < now - 600000 then NotifCache[hash] = nil end
        end
    end
end)

print('^2[RDE | YourScript]^7 Server v1.0.0 loaded ✅')
```

### client/client.lua — Minimal Working Skeleton

```lua
--[[
╔════════════════════════════════════╗
║  rde_yourscript — CLIENT v1.0.0    ║
╚════════════════════════════════════╝
]]

local Config = require 'config'

local State = {
    player   = { identifier = nil, isAdmin = false },
    items    = {},
    entities = {},
    zones    = {},
    ready    = false,
}

local function Log(msg, level)
    if not Config.Debug and level ~= 'ERROR' then return end
    local prefix = level == 'ERROR' and '^1' or '^2'
    print(('%s[RDE YourScript CLIENT]^7 %s'):format(prefix, msg))
end

-- Server sets player context
RegisterNetEvent('rde_ys:setPlayer', function(data)
    State.player = data
    State.ready  = true
    Log(('Player set: %s (admin: %s)'):format(data.identifier, tostring(data.isAdmin)), 'INFO')
end)

-- Statebag handlers
RegisterNetEvent('rde_ys:statebagUpdate', function(id, data)
    State.items[id] = data
    SpawnEntityIfNeeded(id, data)
end)

RegisterNetEvent('rde_ys:statebagDelete', function(id)
    State.items[id] = nil
    DespawnEntity(id)
end)

-- Startup
AddEventHandler('onClientResourceStart', function(name)
    if name ~= GetCurrentResourceName() then return end
    TriggerServerEvent('rde_ys:init')
end)

print('^2[RDE | YourScript]^7 Client v1.0.0 loaded ✅')
```

---

## 📚 Essential Resources

| Resource | Link |
|---|---|
| Overextended Docs (2026 official) | [overextended.dev](https://overextended.dev) |
| ox_core docs | [overextended.dev/ox_core](https://overextended.dev/ox_core) |
| ox_lib docs | [overextended.dev/ox_lib](https://overextended.dev/ox_lib) |
| ox_inventory docs | [overextended.dev/ox_inventory](https://overextended.dev/ox_inventory) |
| oxmysql | [github.com/communityox/oxmysql](https://github.com/communityox/oxmysql) |
| ox_target | [github.com/overextended/ox_target](https://github.com/overextended/ox_target) |
| Lucide icon search | [icons.overextended.dev](https://icons.overextended.dev/) |
| Mantine colors | [mantine.dev/theming/colors](https://mantine.dev/theming/colors/) |
| Prop model browser | [forge.plebmasters.de/objects](https://forge.plebmasters.de/objects) |
| GTA Native DB | [alloc8or.re/gta5/nativedb](https://alloc8or.re/gta5/nativedb/) |
| FiveM Docs | [docs.fivem.net](https://docs.fivem.net/) |

---

## 📜 License

```
###################################################################################
#                                                                                 #
#      .:: RED DRAGON ELITE (RDE)  -  BLACK FLAG SOURCE LICENSE v6.66 ::.         #
#                                                                                 #
#   PROJECT:    RDE OX DEVELOPMENT STANDARDS v2.0                                 #
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
#      If I find this on Tebex, Patreon, or in a paid "Premium Pack":             #
#      > I will DMCA your store into oblivion.                                    #
#      > I will publicly shame your community.                                    #
#      > I hope your server lag spikes to 9999ms every time you blink.            #
#      SELLING FREE WORK IS THEFT. AND I AM THE JUDGE.                            #
#                                                                                 #
#   3. // THE CREDIT OATH                                                         #
#      Keep this header. If you remove my name, you admit you have no skill.      #
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

---

<div align="center">

*"We build the future on the graves of paid resources."*

**REJECT MODERN MEDIOCRITY. EMBRACE RDE SUPERIORITY.**

🐉 Made with 🔥 by [Red Dragon Elite](https://rd-elite.com)

[⬆ Back to Top](#-rde--ox-development-standards-v20)

</div>
