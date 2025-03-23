-- ValuablesHighlighter.lua
-- Script that highlights valuable items in red, enemies in blue, and adds aimbot when holding Q

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Print function for debugging
local function debugPrint(message)
    print("ValuablesHighlighter: " .. message)
end

debugPrint("Script started")

-- Configuration
local VALUABLE_HIGHLIGHT_COLOR = Color3.fromRGB(255, 0, 0) -- Red for valuables
local ENEMY_HIGHLIGHT_COLOR = Color3.fromRGB(0, 0, 255) -- Blue for enemies
local PLAYER_HIGHLIGHT_COLOR = Color3.fromRGB(0, 255, 0) -- Green for other players
local MAX_DISTANCE = 1000 -- Maximum distance to highlight items
local CAMERA_TRANSITION_TIME = 0.05 -- Faster camera transition for aimbot
local TARGET_KEY = Enum.KeyCode.Q -- Key to hold for aimbot
local TARGET_KEY_ALT = "Q" -- Alternative string representation for key checks
local ALTERNATIVE_KEY = Enum.KeyCode.T -- Alternative key for aimbot (in case Q doesn't work)
local TEXT_SIZE = 18 -- Larger text size

-- List of valuable item names (exact names, ignoring case and spaces)
local VALUABLE_ITEMS = {
    -- Regular Valuables
    "gold bar", "silver bar", "gold watch", "silver watch", 
    "silver cup", "gold cup", "silver plate", "gold plate",
    "stone statue", "silver statue", "gold statue",
    "wooden painting", "silver painting", "gold painting",
    
    -- Tools & Misc Items
    "torch", "metal sheet", "barbed wire", "lantern", "bandage", 
    "snake oil", "camera", "saddle", "lightning rod",
    
    -- Armor
    "helmet", "chestplate", "right shoulder armor", "left shoulder armor",
    
    -- Special Items
    "nikola tesla's brain", "werewolf torso", "werewolf's left arm",
    "werewolf's right arm", "werewolf's left leg", "werewolf's right leg",
    
    -- Fuel & Other Items
    "rope", "barrel", "chair", "wheel", "book", "newspaper", "coal", "banjo", "vase"
}

-- List of enemy names to highlight in blue (adding more variations)
local ENEMY_NAMES = {
    -- Common enemy types
    "zombie", "fast zombie", "armed zombie", "scientist zombie", "zombie banker",
    "outlaw", "horseback outlaw", "vampire", "wolf", "werewolf",
    "captain prescott", "nikola tesla", "corpse", "walker", "rifle outlaw",  -- Added comma
    
    -- Additional matching patterns
    "npc", "enemy", "monster", "hostile", "bandit", "undead",
    
    -- Common model names for enemies in different states
    "character", "player", "model", "humanoid"
}

-- List of entities to avoid targeting with aimbot
local AVOID_AIMBOT_ON = {
    "horse", "unicorn", "mount", "vehicle", "dead", "corpse", 
    "wagon", "cart", "carriage", "coach", "saddle", "steed", "pony", "Horse"
}

-- Storage for highlighted items
local highlightedItems = {}
local targetingEnabled = false
local currentTarget = nil
local camera = workspace.CurrentCamera
local nearestItems = {}

-- Normalize text (remove spaces, lowercase)
local function normalizeText(text)
    return text:lower():gsub("%s+", "")
end

-- Normalize all valuable item names for comparison
local normalizedValuables = {}
for _, name in ipairs(VALUABLE_ITEMS) do
    normalizedValuables[normalizeText(name)] = true
end

-- Normalize all enemy names for comparison
local normalizedEnemies = {}
for _, name in ipairs(ENEMY_NAMES) do
    normalizedEnemies[normalizeText(name)] = true
end

-- Check if an item name contains any valuable name
local function isValuable(itemName)
    if not itemName then return false end
    
    local normalizedName = normalizeText(itemName)
    
    -- Check for exact match after normalization
    if normalizedValuables[normalizedName] then
        return true
    end
    
    -- Also check if the name contains any of the valuable names
    for name, _ in pairs(normalizedValuables) do
        if normalizedName:find(name) then
            return true
        end
    end
    
    return false
end

-- Check if an item name matches an enemy with expanded checks
local function isEnemy(itemName)
    if not itemName then return false end
    
    local normalizedName = normalizeText(itemName)
    
    -- Check for exact match after normalization
    if normalizedEnemies[normalizedName] then
        return true
    end
    
    -- Also check if the name contains any of the enemy names
    for name, _ in pairs(normalizedEnemies) do
        if normalizedName:find(name) then
            return true
        end
    end
    
    -- Additional checks for enemy models
    -- Check if the item has a humanoid (indicates NPC/character)
    if typeof(itemName) == "Instance" then
        if itemName:IsA("Model") and itemName:FindFirstChildOfClass("Humanoid") then
            -- It's likely a character model since it has a humanoid
            return true
        end
    end
    
    return false
end

-- Create highlight for an item
local function highlightItem(item, isEnemyItem, isPlayer)
    if highlightedItems[item] then return end
    
    local highlight = Instance.new("Highlight")
    
    -- Choose color based on item type
    if isPlayer then
        highlight.FillColor = PLAYER_HIGHLIGHT_COLOR
        highlight.OutlineColor = PLAYER_HIGHLIGHT_COLOR
    else
        highlight.FillColor = isEnemyItem and ENEMY_HIGHLIGHT_COLOR or VALUABLE_HIGHLIGHT_COLOR
        highlight.OutlineColor = isEnemyItem and ENEMY_HIGHLIGHT_COLOR or VALUABLE_HIGHLIGHT_COLOR
    end
    
    highlight.FillTransparency = 1 -- Transparent fill (border only)
    highlight.OutlineTransparency = 0 -- Solid border
    highlight.Adornee = item
    highlight.Parent = item
    
    local billboardGui = Instance.new("BillboardGui")
    billboardGui.Size = UDim2.new(0, 200, 0, 50)
    billboardGui.StudsOffset = Vector3.new(0, 2, 0)
    billboardGui.Adornee = item
    billboardGui.AlwaysOnTop = true
    
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = TEXT_SIZE
    
    -- Rename "Walker" to "Zombie" in display text only
    local displayName = item.Name
    if displayName:lower():find("walker") then
        displayName = displayName:gsub("[Ww][Aa][Ll][Kk][Ee][Rr]", "Zombie")
    end
    
    -- For players, try to get their actual player name
    if isPlayer then
        local player = Players:GetPlayerFromCharacter(item)
        if player then
            displayName = player.Name
        end
    end
    
    nameLabel.Text = displayName
    nameLabel.Parent = billboardGui
    
    billboardGui.Parent = item
    
    highlightedItems[item] = {
        highlight = highlight,
        billboardGui = billboardGui,
        isEnemy = isEnemyItem,
        isPlayer = isPlayer,
        displayName = displayName
    }
    
    local highlightType = isPlayer and " (Player)" or (isEnemyItem and " (Enemy)" or " (Valuable)")
    debugPrint("Highlighted: " .. displayName .. highlightType)
end

-- Remove highlight from an item
local function removeHighlight(item)
    if not highlightedItems[item] then return end
    
    pcall(function()
        highlightedItems[item].highlight:Destroy()
        highlightedItems[item].billboardGui:Destroy()
    end)
    
    highlightedItems[item] = nil
end

-- Calculate distance between player and an item
local function getDistanceToItem(item)
    local localPlayer = Players.LocalPlayer
    if not localPlayer or not localPlayer.Character then return 9999999 end
    
    local character = localPlayer.Character
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return 9999999 end
    
    local itemPosition = nil
    
    if item:IsA("BasePart") then
        itemPosition = item.Position
    elseif item:IsA("Model") and item:FindFirstChild("PrimaryPart") and item.PrimaryPart then
        itemPosition = item.PrimaryPart.Position
    elseif item:IsA("Model") and item:FindFirstChild("Head") then
        itemPosition = item.Head.Position
    elseif item:IsA("Model") then
        -- Try to find any part
        for _, child in pairs(item:GetChildren()) do
            if child:IsA("BasePart") then
                itemPosition = child.Position
                break
            end
        end
    end
    
    if not itemPosition then return 9999999 end
    
    return (itemPosition - rootPart.Position).Magnitude
end

-- Check if an entity should be avoided for aimbot
local function shouldAvoidAimbot(item)
    if not item then return true end
    
    -- Check for exact match first (higher priority)
    if item.Name == "Horse" then
        debugPrint("Avoided aimbotting at exact horse match: " .. item.Name)
        return true
    end
    
    local name = item.Name:lower()
    
    -- Check if it's in the avoid list
    for _, avoidName in ipairs(AVOID_AIMBOT_ON) do
        if name:find(avoidName) then
            debugPrint("Avoided aimbotting at: " .. name .. " (matched with " .. avoidName .. ")")
            return true
        end
    end
    
    -- Check if it's another player
    if item:IsA("Model") and Players:GetPlayerFromCharacter(item) then
        debugPrint("Avoided aimbotting at player character")
        return true
    end
    
    -- Check if it's dead (has 0 health)
    if item:IsA("Model") then
        local humanoid = item:FindFirstChildOfClass("Humanoid")
        if humanoid and humanoid.Health <= 0 then
            debugPrint("Avoided aimbotting at dead entity with 0 health")
            return true
        end
        
        -- Additional check for riding animations or rider properties
        if humanoid and (humanoid:GetState() == Enum.HumanoidStateType.Riding) then
            debugPrint("Avoided aimbotting at entity in riding state")
            return true
        end
        
        -- Check if it has a saddle part (indicating it's a horse)
        if item:FindFirstChild("Saddle") then
            debugPrint("Avoided aimbotting at entity with a saddle")
            return true
        end
    end
    
    -- Check parent for horse-related names (for mounted riders)
    if item.Parent and typeof(item.Parent) == "Instance" then
        local parentName = item.Parent.Name:lower()
        for _, avoidName in ipairs(AVOID_AIMBOT_ON) do
            if parentName:find(avoidName) then
                debugPrint("Avoided aimbotting at entity on " .. parentName)
                return true
            end
        end
        
        -- Explicit check for parent being a horse
        if item.Parent.Name == "Horse" then
            debugPrint("Avoided aimbotting at entity on exact horse match")
            return true
        end
    end
    
    -- Check if this entity is a child of a horse or related to a horse
    local function hasHorseInAncestry(obj, depth)
        depth = depth or 0
        if depth > 3 then return false end -- Limit recursion depth
        
        if not obj or not obj.Parent then return false end
        
        -- Check direct parent
        if obj.Parent.Name == "Horse" or string.find(obj.Parent.Name:lower(), "horse") then
            return true
        end
        
        -- Check for saddle in siblings
        for _, sibling in ipairs(obj.Parent:GetChildren()) do
            if sibling.Name == "Saddle" or string.find(sibling.Name:lower(), "saddle") then
                return true
            end
        end
        
        -- Continue checking up the hierarchy
        return hasHorseInAncestry(obj.Parent, depth + 1)
    end
    
    if hasHorseInAncestry(item) then
        debugPrint("Avoided aimbotting at entity with horse in ancestry")
        return true
    end
    
    return false
end

-- Check if there's a clear line of sight to the target
local function hasLineOfSight(startPos, endPos, ignoreList)
    -- Wrap in pcall for safety
    local success, result = pcall(function()
        -- Parameters for the raycast
        local raycastParams = RaycastParams.new()
        raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
        raycastParams.FilterDescendantsInstances = ignoreList or {}
        
        -- Calculate ray direction
        local direction = (endPos - startPos).Unit
        local distance = (endPos - startPos).Magnitude
        
        -- Perform the raycast
        local raycastResult = workspace:Raycast(startPos, direction * distance, raycastParams)
        
        -- If raycast hit nothing, there's a clear line of sight
        if raycastResult == nil then
            return true
        end
        
        -- Check the distance to where the ray hit
        local hitDistance = (raycastResult.Position - startPos).Magnitude
        
        -- If the hit point is very close to the endpoint, we consider it a line of sight
        -- This accounts for hitting the target's collision box
        if math.abs(hitDistance - distance) < 2 then
            return true
        end
        
        -- Otherwise, something is blocking the view
        return false
    end)
    
    -- If there was an error, default to true to prevent breaking functionality
    if not success then
        debugPrint("Line of sight check failed: " .. tostring(result))
        return true
    end
    
    return result
end

-- Find nearest enemy's head
local function findNearestEnemyHead()
    local localPlayer = Players.LocalPlayer
    if not localPlayer or not localPlayer.Character then return nil end
    
    local character = localPlayer.Character
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return nil end
    
    local playerPosition = rootPart.Position
    local closestEnemy = nil
    local closestDistance = MAX_DISTANCE
    local closestHead = nil
    
    -- Build a list of objects to ignore in the line of sight check
    local ignoreList = {character}
    
    -- Create a list of all player characters to avoid targeting
    local playerCharacters = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            playerCharacters[player.Character] = true
        end
    end
    
    -- Check all highlighted enemies
    for item, data in pairs(highlightedItems) do
        if data.isEnemy then
            -- Skip items that should be avoided for aimbot
            if shouldAvoidAimbot(item) then continue end
            
            -- Skip if this is a player character
            if item:IsA("Model") and playerCharacters[item] then
                debugPrint("Skipping player character for aimbot: " .. item.Name)
                continue
            end
            
            -- Skip if this belongs to a player
            if item:IsA("Model") and Players:GetPlayerFromCharacter(item) then
                debugPrint("Skipping player-owned character for aimbot: " .. item.Name)
                continue 
            end
            
            local head = nil
            
            -- Try to find the head
            if item:IsA("Model") then
                head = item:FindFirstChild("Head")
                table.insert(ignoreList, item) -- Add enemy to ignore list for raycast
            elseif item.Parent and item.Parent:IsA("Model") then
                -- Check if parent is a player character first
                if playerCharacters[item.Parent] or Players:GetPlayerFromCharacter(item.Parent) then
                    debugPrint("Skipping player-owned parent model for aimbot: " .. item.Parent.Name)
                    continue
                end
                
                head = item.Parent:FindFirstChild("Head")
                table.insert(ignoreList, item.Parent) -- Add enemy to ignore list for raycast
            end
            
            if head and head:IsA("BasePart") then
                local distance = (head.Position - playerPosition).Magnitude
                
                -- Check if there's a clear line of sight to the head
                if distance < closestDistance and hasLineOfSight(playerPosition, head.Position, ignoreList) then
                    closestDistance = distance
                    closestEnemy = item
                    closestHead = head
                end
            end
        end
    end
    
    return closestHead
end

-- Update camera to aim at target
local function updateCamera()
    if not targetingEnabled then return end
    
    -- Find a target if we don't have one
    if not currentTarget then
        currentTarget = findNearestEnemyHead()
        if not currentTarget then return end
    end
    
    -- Extra safety check to ensure we're not targeting a player
    if currentTarget and currentTarget.Parent then
        if Players:GetPlayerFromCharacter(currentTarget.Parent) then
            currentTarget = nil
            debugPrint("Prevented aiming at player in camera update")
            return
        end
    end
    
    -- More aggressive camera control - force it to aim at target
    pcall(function()
        camera.CFrame = CFrame.new(camera.CFrame.Position, currentTarget.Position)
    end)
end

-- Function to scan for valuable and enemy items
local function scanItems()
    -- Clean up invalid highlights
    for item in pairs(highlightedItems) do
        if not item or not item:IsDescendantOf(game) then
            removeHighlight(item)
        end
    end
    
    -- List of places to check for enemies and valuables
    local placesToCheck = {
        workspace,
        workspace:FindFirstChild("Entities"),
        workspace:FindFirstChild("NPCs"),
        workspace:FindFirstChild("Enemies"),
        workspace:FindFirstChild("Characters"),
        workspace:FindFirstChild("Monsters"),
        workspace:FindFirstChild("Zombies"),
        workspace:FindFirstChild("Players"), -- Sometimes NPCs are in a Players folder
        workspace:FindFirstChild("Models"),
        workspace:FindFirstChild("Walkers") -- Added explicit check for Walkers folder
    }
    
    -- First, highlight other players
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer and player.Character then
            highlightItem(player.Character, false, true) -- Not an enemy, but a player
        end
    end
    
    -- Scan all specified places
    for _, place in ipairs(placesToCheck) do
        if place then
            -- First check direct children
            for _, item in ipairs(place:GetChildren()) do
                -- Skip terrain and already highlighted players
                if item == workspace.Terrain then continue end
                if item:IsA("Model") and Players:GetPlayerFromCharacter(item) and highlightedItems[item] then continue end
                
                -- Check if it's a player character
                if item:IsA("Model") and Players:GetPlayerFromCharacter(item) then
                    if not highlightedItems[item] then
                        highlightItem(item, false, true) -- Not an enemy, but a player
                    end
                -- Check if item is valuable or enemy    
                elseif isValuable(item.Name) then
                    highlightItem(item, false, false) -- not an enemy, not a player
                elseif isEnemy(item.Name) or (item:IsA("Model") and item:FindFirstChildOfClass("Humanoid")) then
                    highlightItem(item, true, false) -- is an enemy, not a player
                end
                
                -- Also check direct children
                for _, child in ipairs(item:GetChildren()) do
                    if isValuable(child.Name) then
                        highlightItem(child, false, false)
                    elseif isEnemy(child.Name) then
                        highlightItem(child, true, false)
                    end
                end
            end
        end
    end
    
    -- Special check for models with humanoids (likely characters/NPCs)
    for _, model in ipairs(workspace:GetDescendants()) do
        if model:IsA("Model") and not highlightedItems[model] then
            -- First check if it's a player
            if Players:GetPlayerFromCharacter(model) then
                highlightItem(model, false, true) -- Not an enemy, but a player
            -- Then check if it's an enemy
            elseif model:FindFirstChildOfClass("Humanoid") then
                -- Check if it also has a head (for aimbot targeting)
                if model:FindFirstChild("Head") then
                    highlightItem(model, true, false) -- Consider it an enemy, not a player
                end
            end
        end
    end
    
    -- Update nearest items list
    local nearestValuables = {}
    local nearestEnemies = {}
    local nearestPlayers = {} -- New list for players
    
    for item, data in pairs(highlightedItems) do
        local distance = getDistanceToItem(item)
        
        -- Skip items closer than 3 studs
        if distance < 3 then
            continue
        end
        
        local itemInfo = {
            name = data.displayName or item.Name,
            distance = distance,
            item = item,
            isEnemy = data.isEnemy,
            isPlayer = data.isPlayer
        }
        
        if data.isPlayer then
            table.insert(nearestPlayers, itemInfo)
        elseif data.isEnemy then
            table.insert(nearestEnemies, itemInfo)
        else
            table.insert(nearestValuables, itemInfo)
        end
    end
    
    -- Sort by distance
    table.sort(nearestValuables, function(a, b)
        return a.distance < b.distance
    end)
    
    table.sort(nearestEnemies, function(a, b)
        return a.distance < b.distance
    end)
    
    table.sort(nearestPlayers, function(a, b)
        return a.distance < b.distance
    end)
    
    -- Combine into a single list while keeping track of categories
    nearestItems = {
        valuables = nearestValuables,
        enemies = nearestEnemies,
        players = nearestPlayers -- Add players to the list
    }
    
    -- If targeting is enabled, update the current target
    if targetingEnabled then
        currentTarget = findNearestEnemyHead()
        if currentTarget then
            updateCamera()
        end
    end
end

-- Create UI for nearest items
local function createNearestItemsUI()
    -- Create the ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "NearestItemsUI"
    screenGui.ResetOnSpawn = false
    
    -- Create the main frame
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 300) -- Reduced height (removed players section)
    mainFrame.Position = UDim2.new(0, 10, 0, 10)
    mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    mainFrame.BackgroundTransparency = 0.2 -- More opaque background
    mainFrame.BorderSizePixel = 2
    mainFrame.BorderColor3 = Color3.fromRGB(255, 255, 255)
    mainFrame.Parent = screenGui
    
    -- Create the title label
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "TitleLabel"
    titleLabel.Size = UDim2.new(1, 0, 0, 45) -- Taller
    titleLabel.Position = UDim2.new(0, 0, 0, 0)
    titleLabel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    titleLabel.BackgroundTransparency = 0
    titleLabel.BorderSizePixel = 0
    titleLabel.Font = Enum.Font.SourceSansBold
    titleLabel.TextSize = 24 -- Larger text
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.Text = "Nearest Items (Hold Q for Aimbot)"
    titleLabel.Parent = mainFrame
    
    -- Create valuables section label
    local valuablesLabel = Instance.new("TextLabel")
    valuablesLabel.Name = "ValuablesLabel"
    valuablesLabel.Size = UDim2.new(1, 0, 0, 35)
    valuablesLabel.Position = UDim2.new(0, 0, 0, 45)
    valuablesLabel.BackgroundColor3 = Color3.fromRGB(180, 0, 0) -- Brighter red for better contrast
    valuablesLabel.BackgroundTransparency = 0.1
    valuablesLabel.BorderSizePixel = 0
    valuablesLabel.Font = Enum.Font.SourceSansBold
    valuablesLabel.TextSize = 20 -- Larger text
    valuablesLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    valuablesLabel.Text = "Valuable Items"
    valuablesLabel.Parent = mainFrame
    
    -- Create labels for the nearest valuable items
    for i = 1, 3 do
        local itemLabel = Instance.new("TextLabel")
        itemLabel.Name = "Valuable" .. i
        itemLabel.Size = UDim2.new(1, 0, 0, 30) -- Taller
        itemLabel.Position = UDim2.new(0, 0, 0, 80 + (i - 1) * 30)
        itemLabel.BackgroundTransparency = 0.7
        itemLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        itemLabel.Font = Enum.Font.SourceSansBold
        itemLabel.TextSize = 19 -- Larger text
        itemLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        itemLabel.TextXAlignment = Enum.TextXAlignment.Left
        itemLabel.Text = "  " .. i .. ". Nothing found"
        itemLabel.TextStrokeTransparency = 0.7 -- Add text stroke for better visibility
        itemLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        itemLabel.Parent = mainFrame
    end
    
    -- Create enemies section label
    local enemiesLabel = Instance.new("TextLabel")
    enemiesLabel.Name = "EnemiesLabel"
    enemiesLabel.Size = UDim2.new(1, 0, 0, 35)
    enemiesLabel.Position = UDim2.new(0, 0, 0, 170)
    enemiesLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 180) -- Brighter blue for better contrast
    enemiesLabel.BackgroundTransparency = 0.1
    enemiesLabel.BorderSizePixel = 0
    enemiesLabel.Font = Enum.Font.SourceSansBold
    enemiesLabel.TextSize = 20 -- Larger text
    enemiesLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    enemiesLabel.Text = "Nearby Enemies"
    enemiesLabel.Parent = mainFrame
    
    -- Create labels for the nearest enemies
    for i = 1, 3 do
        local enemyLabel = Instance.new("TextLabel")
        enemyLabel.Name = "Enemy" .. i
        enemyLabel.Size = UDim2.new(1, 0, 0, 30) -- Taller
        enemyLabel.Position = UDim2.new(0, 0, 0, 205 + (i - 1) * 30)
        enemyLabel.BackgroundTransparency = 0.7
        enemyLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        enemyLabel.Font = Enum.Font.SourceSansBold
        enemyLabel.TextSize = 19 -- Larger text
        enemyLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        enemyLabel.TextXAlignment = Enum.TextXAlignment.Left
        enemyLabel.Text = "  " .. i .. ". Nothing found"
        enemyLabel.TextStrokeTransparency = 0.7 -- Add text stroke for better visibility
        enemyLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        enemyLabel.Parent = mainFrame
    end
    
    -- Create aimbot status label
    local aimbotLabel = Instance.new("TextLabel")
    aimbotLabel.Name = "AimbotStatus"
    aimbotLabel.Size = UDim2.new(1, 0, 0, 35)
    aimbotLabel.Position = UDim2.new(0, 0, 0, 265) -- Adjusted position
    aimbotLabel.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    aimbotLabel.BackgroundTransparency = 0.1 -- More opaque
    aimbotLabel.BorderSizePixel = 2
    aimbotLabel.BorderColor3 = Color3.fromRGB(255, 255, 255)
    aimbotLabel.Font = Enum.Font.SourceSansBold
    aimbotLabel.TextSize = 24 -- Larger texta
    aimbotLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    aimbotLabel.Text = "Aimbot: OFF"
    aimbotLabel.Parent = mainFrame
    
    -- Return the UI elements we'll need to update
    return {
        screenGui = screenGui,
        valuable1 = mainFrame.Valuable1,
        valuable2 = mainFrame.Valuable2,
        valuable3 = mainFrame.Valuable3,
        enemy1 = mainFrame.Enemy1,
        enemy2 = mainFrame.Enemy2,
        enemy3 = mainFrame.Enemy3,
        aimbotStatus = mainFrame.AimbotStatus
    }
end

-- Update the nearest items UI
local function updateNearestItemsUI(ui)
    -- Update valuable items
    for i = 1, 3 do
        if nearestItems.valuables[i] then
            local item = nearestItems.valuables[i]
            local distanceRounded = math.floor(item.distance * 10) / 10
            
            ui["valuable" .. i].Text = "  " .. i .. ". " .. item.name .. " (" .. distanceRounded .. " studs)"
            ui["valuable" .. i].TextColor3 = Color3.fromRGB(255, 80, 80) -- Brighter red for better contrast
        else
            ui["valuable" .. i].Text = "  " .. i .. ". Nothing found"
            ui["valuable" .. i].TextColor3 = Color3.fromRGB(220, 220, 220) -- Brighter text for better visibility
        end
    end
    
    -- Update enemy items
    for i = 1, 3 do
        if nearestItems.enemies[i] then
            local item = nearestItems.enemies[i]
            local distanceRounded = math.floor(item.distance * 10) / 10
            
            -- Check if we have line of sight to this enemy
            local hasLOS = false
            if item.item:IsA("Model") then
                local head = item.item:FindFirstChild("Head")
                if head and Players.LocalPlayer and Players.LocalPlayer.Character then
                    local rootPart = Players.LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    if rootPart then
                        hasLOS = hasLineOfSight(rootPart.Position, head.Position, {Players.LocalPlayer.Character, item.item})
                    end
                end
            end
            
            local text = "  " .. i .. ". " .. item.name .. " (" .. distanceRounded .. " studs)"
            if not hasLOS then
                text = text .. " [Obstructed]"
            end
            
            ui["enemy" .. i].Text = text
            ui["enemy" .. i].TextColor3 = hasLOS and Color3.fromRGB(80, 80, 255) or Color3.fromRGB(150, 150, 220) -- Dimmer for obstructed
        else
            ui["enemy" .. i].Text = "  " .. i .. ". Nothing found"
            ui["enemy" .. i].TextColor3 = Color3.fromRGB(220, 220, 220) -- Brighter text for better visibility
        end
    end
    
    if targetingEnabled then
        local targetText = "Aimbot: ON"
        if currentTarget then
            local targetName = currentTarget.Parent and currentTarget.Parent.Name or "Unknown"
            targetText = "Aiming at: " .. targetName
        else
            targetText = "Aimbot: No visible target"
        end
        ui.aimbotStatus.Text = targetText
        ui.aimbotStatus.BackgroundColor3 = currentTarget and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 200, 0) -- Yellow if no target
    else
        ui.aimbotStatus.Text = "Aimbot: OFF"
        ui.aimbotStatus.BackgroundColor3 = Color3.fromRGB(200, 0, 0) -- Brighter red
    end
end

-- SIMPLIFIED input handling for aimbot
local function setupInputHandling()
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.KeyCode == TARGET_KEY or input.KeyCode == ALTERNATIVE_KEY then
            targetingEnabled = true
            debugPrint("Aimbot enabled")
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if input.KeyCode == TARGET_KEY or input.KeyCode == ALTERNATIVE_KEY then
            targetingEnabled = false
            currentTarget = nil
            debugPrint("Aimbot disabled")
        end
    end)
end

-- Wait for player to load
debugPrint("Waiting for local player...")
local localPlayer
if Players.LocalPlayer then
    localPlayer = Players.LocalPlayer
    debugPrint("Local player already exists: " .. localPlayer.Name)
else
    debugPrint("Waiting for LocalPlayer...")
    repeat wait() until Players.LocalPlayer
    localPlayer = Players.LocalPlayer
    debugPrint("Local player loaded: " .. localPlayer.Name)
end

-- Create the UI
local ui = createNearestItemsUI()
ui.screenGui.Parent = localPlayer:WaitForChild("PlayerGui")

-- Set up input handling for aimbot
setupInputHandling()

-- Main update loop
RunService.Heartbeat:Connect(function()
    scanItems()
    
    -- Direct key check as a backup - Roblox's input system can be unreliable
    local isKeyDown = UserInputService:IsKeyDown(TARGET_KEY) or UserInputService:IsKeyDown(ALTERNATIVE_KEY)
    if isKeyDown ~= targetingEnabled then
        targetingEnabled = isKeyDown
        if not isKeyDown then
            currentTarget = nil
        end
    end
    
    -- If targeting is enabled, always try to find a target
    if targetingEnabled then
        if not currentTarget then
            currentTarget = findNearestEnemyHead()
        end
        updateCamera()
    end
    
    updateNearestItemsUI(ui)
end)

-- Clean up highlights when items are removed
game.DescendantRemoving:Connect(function(instance)
    if highlightedItems[instance] then
        removeHighlight(instance)
    end
end)

-- Check input setup every 5 seconds as a failsafe (decreased from 10 for faster recovery)
spawn(function()
    while wait(5) do
        pcall(function()
            if #_G.inputConnections < 3 then
                debugPrint("Input connections lost, recreating...")
                setupInputHandling()
            end
            
            -- Force check the key state directly
            local isKeyDown = UserInputService:IsKeyDown(TARGET_KEY)
            if isKeyDown and not targetingEnabled then
                targetingEnabled = true
                debugPrint("Fixed targeting state via periodic check")
            end
        end)
    end
end)

-- ContextAction-based backup input detection (even more reliable)
local function setupContextAction()
    pcall(function()
        local ContextActionService = game:GetService("ContextActionService")
        
        -- Clear any existing bindings
        pcall(function() ContextActionService:UnbindAction("AimbotActivate") end)
        
        -- Bind a new action
        ContextActionService:BindAction(
            "AimbotActivate",
            function(_, inputState, inputObject)
                if inputState == Enum.UserInputState.Begin then
                    targetingEnabled = true
                    debugPrint("Aimbot enabled via ContextAction")
                elseif inputState == Enum.UserInputState.End then
                    targetingEnabled = false
                    currentTarget = nil
                    debugPrint("Aimbot disabled via ContextAction")
                end
                return Enum.ContextActionResult.Pass -- Let the input propagate to other handlers
            end,
            false, -- Don't create touch button
            TARGET_KEY, ALTERNATIVE_KEY -- Bind to both keys
        )
        
        debugPrint("ContextAction binding set up successfully")
    end)
end

-- Call context action setup
setupContextAction()

-- Display a note about the alternative key
debugPrint("*** You can also use E as an alternative aimbot key if Q doesn't work! ***")

debugPrint("Script fully initialized and running")
