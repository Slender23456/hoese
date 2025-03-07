-- Keysystem
local KeySystem = loadstring(game:HttpGet("https://raw.githubusercontent.com/OopssSorry/LuaU-Free-Key-System-UI/main/source.lua"))()
local KeyValid = false
local response = KeySystem:Init({
    Debug=false,
    Title="Slender Hub | Key System",
    Description=nil,
    Link="https://fir3.net/UF854v",
    Discord="https://discord.gg/UFdqC6Nz", -- Link do Discord adicionado
    SaveKey=false,
    Verify=function(key)
        if key=="A9X3B7Z5Q2L8M4C6" then
            KeyValid=true
            return true
        else
            return false
        end
    end,
    GuiParent = game.CoreGui,
})

if not response or not KeyValid then return end

-- Horse Race
if game.PlaceId == 93787311916283 then

    -- Load
    local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/jensonhirst/Orion/main/source')))()

    -- Main Window
    local Window = OrionLib:MakeWindow({
        Name = "Slender Hub",
        HidePremium = true,
        SaveConfig = true,
        ConfigFolder = "Slender",
        IntroEnabled = false
    })

    -- Global Variables
    _G.autoTrain = false
    _G.selectedWorld = 1
    _G.selectedMachine = 1
    _G.selectTrain = "Treadmill_1_1"
    _G.autoHatch = false
    _G.selectedEggWorld = 1
    _G.selectedEggNumber = 1
    _G.selectEgg = "Egg_1_1"
    _G.autoSpin = false
    _G.autoBuyEquipment = false
    _G.selectedCrate = "Crate_2"

    -- ============================
    -- FUNCTIONS
    -- ============================

    -- Function: Auto Train Loop
    function autoTrain()
        while _G.autoTrain do
            game:GetService("ReplicatedStorage").Packages._Index["sleitnick_knit@1.5.1"].knit.Services.TrainService.RE.RunTrain:FireServer(_G.selectTrain)
            wait(0)
        end
    end

    -- Function: Update Train Selection
    function updateTrainSelection()
        _G.selectTrain = "Treadmill_" .. _G.selectedWorld .. "_" .. _G.selectedMachine
        print("Selected:", _G.selectTrain)
    end

    --Function: Auto egg
    function autoHatch()
        while _G.autoHatch do
            local eggName = "Egg_" .. _G.selectedEggWorld .. "_" .. _G.selectedEggNumber
            game:GetService("ReplicatedStorage").Packages._Index["sleitnick_knit@1.5.1"].knit.Services.EggHatchService.RE.Hatch:FireServer(eggName, 1)
            wait(0.1)
        end
    end

    -- Function: Auto Spin
    function autoSpin()
        while _G.autoSpin do
            game:GetService("ReplicatedStorage").Packages._Index["sleitnick_knit@1.5.1"].knit.Services.SpinningWheelService.RF.StartSpin:InvokeServer()
            wait(0.1)
        end
    end

    -- Function: Auto Buy Equipment
    function autoBuyEquipment()
        while _G.autoBuyEquipment do
            game:GetService("ReplicatedStorage").Packages._Index["sleitnick_knit@1.5.1"].knit.Services.ItemCrateService.RE.UnboxCrate:FireServer(_G.selectedCrate)
            wait(0.1)
        end
    end

    -- ============================
    -- GUI - FARM TAB
    -- ============================

    local FarmTab = Window:MakeTab({
        Name = "Farm",
        Icon = "rbxassetid://4483345998",
        PremiumOnly = false
    })

    -- ============================
    -- GUI - EGG TAB
    -- ============================

    local EggTab = Window:MakeTab({
        Name = "Egg",
        Icon = "rbxassetid://4483345998",
        PremiumOnly = false
    })

     -- ============================
    -- GUI - MISC TAB
    -- ============================

    local MiscTab = Window:MakeTab({
        Name = "Misc",
        Icon = "rbxassetid://4483345998",
        PremiumOnly = false
    })

    -- ============================
    -- TOGGLES
    -- ============================

    FarmTab:AddToggle({
        Name = "Auto Train",
        Default = false,
        Callback = function(Value)
            _G.autoTrain = Value
            autoTrain()
        end
    })

    EggTab:AddToggle({
        Name = "Auto Egg",
        Default = false,
        Callback = function(Value)
            _G.autoHatch = Value
            autoHatch()
        end
    })

    MiscTab:AddToggle({
        Name = "Auto Spin",
        Default = false,
        Callback = function(Value)
            _G.autoSpin = Value
            if Value then
                autoSpin()
            end
        end
    })

    MiscTab:AddToggle({
        Name = "Buy Equipment",
        Default = false,
        Callback = function(Value)
            _G.autoBuyEquipment = Value
            if Value then
                autoBuyEquipment()
            end
        end
    })

    -- ============================
    -- DROPDOWNS
    -- ============================

    -- Dropdown: Select World (1 to 4) - Farm Tab
    FarmTab:AddDropdown({
        Name = "Select World",
        Default = "1",
        Options = {"1", "2", "3", "4"},
        Callback = function(Value)
            _G.selectedWorld = tonumber(Value)
            updateTrainSelection()
        end
    })

    -- Dropdown: Select Machine (1 to 12) - Farm Tab
    local machineOptions = {}
    for i = 1, 12 do
        table.insert(machineOptions, tostring(i))
    end

    FarmTab:AddDropdown({
        Name = "Select Training Machine",
        Default = "1",
        Options = machineOptions,
        Callback = function(Value)
            _G.selectedMachine = tonumber(Value)
            updateTrainSelection()
        end
    })

    -- Dropdown: Select Egg World (1 to 4) - Egg Tab
    EggTab:AddDropdown({
        Name = "Select World",
        Default = "1",
        Options = {"1", "2", "3", "4"},
        Callback = function(Value)
            _G.selectedEggWorld = tonumber(Value)
        end
    })

    -- Dropdown: Select Egg Number (1 to 4) - Egg Tab
    EggTab:AddDropdown({
        Name = "Select Egg Number",
        Default = "1",
        Options = {"1", "2", "3", "4"},
        Callback = function(Value)
            _G.selectedEggNumber = tonumber(Value)
        end
    })

    -- Dropdown: Select Crate - Misc Tab
    MiscTab:AddDropdown({
        Name = "Select Equipment",
        Default = "Crate_2",
        Options = {"Snow Equipment", "West Equipment"},
        Callback = function(Value)
            if Value == "Snow Equipment" then
                _G.selectedCrate = "Crate_1"
            elseif Value == "West Equipment" then
                _G.selectedCrate = "Crate_2"
            end
        end
    })
end
