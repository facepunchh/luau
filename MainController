local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")
local Debris = game:GetService("Debris")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")
local RunService = game:GetService("RunService")
 
--// services
local Player = Players.LocalPlayer
local mainHUD = script:FindFirstAncestorWhichIsA("ScreenGui")
 
-- fallback check for gui
if not mainHUD then 
    warn("main controller: screengui not found")
    return 
end
 
--// modules setup
local db = require(ReplicatedStorage:WaitForChild("CharacterDatabase"))
local UIUtils = require(ReplicatedStorage:WaitForChild("UIUtils"))
 
local Interface = {
    Shop = require(ReplicatedStorage:WaitForChild("ShopUI")),
    Inventory = require(ReplicatedStorage:WaitForChild("InventoryUI")),
    Deck = require(ReplicatedStorage:WaitForChild("DeckUI")),
    Roll = require(ReplicatedStorage:WaitForChild("RollUI")),
    Battle = require(ReplicatedStorage:WaitForChild("BattleUI")),
    Settings = require(ReplicatedStorage:WaitForChild("SettingsUI")),
    Quests = require(ReplicatedStorage:WaitForChild("QuestsUI")),
    Codes = require(ReplicatedStorage:WaitForChild("CodesUI")),
    Upgrades = require(ReplicatedStorage:WaitForChild("UpgradesUI")),
    BattlePass = require(ReplicatedStorage:WaitForChild("BattlePassUI")),
    Tutorial = require(ReplicatedStorage:WaitForChild("TutorialUI"))
}
 
--// state management
local AppState = {
    localInventory = {},
    localItems = {},
    knownCardsSet = {},
    newCardsCount = 0,
    totalRolls = 0,
    AutoRollEnabled = false,
    ActivePack = Player:GetAttribute("ActivePack") or "Normal",
    Volume = 1.0,
    IsBusy = false,
    LuckExpiry = 0
}
 
--// ui
local LeftBar = mainHUD:WaitForChild("LeftBar")
local LuckHUD = mainHUD:FindFirstChild("LuckBoostHUD")
local NotifTemplate = mainHUD:FindFirstChild("NotifTemplate")
 
local Buttons = {
    Inventory = LeftBar:WaitForChild("Inventory"),
    Settings = LeftBar:WaitForChild("Settings"),
    Deck = LeftBar:WaitForChild("Deck"),
    Shop = LeftBar:WaitForChild("Shop"),
    Quests = LeftBar:WaitForChild("Quests"),
    Codes = LeftBar:WaitForChild("Codes")
}
 
--// logic initialization
UIUtils.InitAudio()
mainHUD.IgnoreGuiInset = true
mainHUD.ResetOnSpawn = false
 
--// utility functions
local function formatTime(secs)
    secs = math.max(0, math.floor(secs))
    if secs >= 60 then
        return string.format("%dm %ds", math.floor(secs/60), secs%60)
    end
    return string.format("%ds", secs)
end
 
local function applyButtonEffect(btn, target)
    local icon = btn:FindFirstChild("Icon") or btn
    local scale = icon:FindFirstChild("UIScale") or Instance.new("UIScale", icon)
    
    local info = TweenInfo.new(0.2, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
    TweenService:Create(scale, info, {Scale = target}):Play()
end
 
--// buttons setup
for name, btn in pairs(Buttons) do
    btn.MouseEnter:Connect(function()
        UIUtils.PlaySound("Hover")
        applyButtonEffect(btn, 1.25)
    end)
    
    btn.MouseLeave:Connect(function()
        applyButtonEffect(btn, 1.0)
    end)
    
    btn.MouseButton1Down:Connect(function()
        applyButtonEffect(btn, 0.9)
    end)
end
 
--// systems initialization
local ShopSys = Interface.Shop.Create(mainHUD, UIUtils, Buttons.Shop, Player)
local InventorySys = Interface.Inventory.Create(mainHUD, UIUtils, Buttons.Inventory, AppState, db)
local DeckSys = Interface.Deck.Create(mainHUD, UIUtils, Buttons.Deck, AppState, db)
local RollSys = Interface.Roll.Create(mainHUD, UIUtils, AppState, db)
 
--// global events / notifs
AppState.ShowNotif = function(text, color)
    if not NotifTemplate then return end
    UIUtils.PlaySound("Notif")
    
    local newNotif = NotifTemplate:Clone()
    newNotif.Parent = mainHUD
    newNotif.Visible = true
    
    -- run animation from utils
    UIUtils.AnimateNotify(newNotif, text, color or Color3.fromRGB(255, 255, 255))
    
    task.delay(3.5, function()
        if newNotif then newNotif:Destroy() end
    end)
end
 
--// handle server communication
local pickupEvent = ReplicatedStorage:WaitForChild("ItemPickedUp", 5)
if pickupEvent then
    pickupEvent.OnClientEvent:Connect(function(itemName, count)
        if count > 0 then
            AppState.localItems[itemName] = count
        else
            AppState.localItems[itemName] = nil
        end
        
        if InventorySys and InventorySys.Refresh then
            InventorySys.Refresh(AppState.localItems)
        end
        
        local alertColor = Color3.fromRGB(80, 255, 120)
        if itemName == "Gold" then
            alertColor = Color3.fromRGB(255, 215, 0)
        end
        
        AppState.ShowNotif("Received: " .. itemName .. "!", alertColor)
    end)
end
 
--// attribute sync
Player:GetAttributeChangedSignal("ActivePack"):Connect(function()
    AppState.ActivePack = Player:GetAttribute("ActivePack") or "Normal"
end)
 
--// input handling
UserInputService.InputBegan:Connect(function(input, processed)
    if processed or AppState.IsBusy then return end
    
    if input.KeyCode == Enum.KeyCode.B then
        local playerTeam = {}
        for i = 1, 3 do
            table.insert(playerTeam, {
                Name = "Unit " .. i, 
                HP = 1000, 
                MaxHP = 1000, 
                Dmg = 150
            })
        end
        
        if Interface.Battle and Interface.Battle.Start then
            Interface.Battle.Start(mainHUD, UIUtils, playerTeam, {})
        end
    end
end)
 
--// luck timer update logic
RunService.Heartbeat:Connect(function()
    local now = os.time()
    if AppState.LuckExpiry > now then
        local diff = AppState.LuckExpiry - now
        if LuckHUD then
            LuckHUD.Visible = true
            local label = LuckHUD:FindFirstChild("TimerLabel")
            if label then
                label.Text = "LUCK BOOST: " .. formatTime(diff)
            end
        end
    else
        if LuckHUD and LuckHUD.Visible then
            LuckHUD.Visible = false
        end
    end
end)
 
--// core gui toggles
task.spawn(function()
    while task.wait(1.5) do
        local success, err = pcall(function() 
            StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.PlayerList, false)
            StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.Chat, true)
        end)
        
        if not success then
            -- silent fail to avoid console spam
        end
    end
end)
 
--// log
print("MainController initialized")
