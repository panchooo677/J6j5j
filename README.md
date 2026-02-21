local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- File names for saving
local SAVE_FILE = "ScriptSaver_SavedScripts.json"
local COLOR_FILE = "ScriptSaver_ColorSettings.json"

-- Script Storage
local savedScripts = {}

-- Rainbow mode variables
local rainbowEnabled = false
local rainbowConnection = nil
local rainbowHue = 0

-- Function to save scripts to file
local function saveScriptsToFile()
    local success, err = pcall(function()
        local dataToSave = {}
        for _, scriptData in ipairs(savedScripts) do
            table.insert(dataToSave, {
                id = scriptData.id,
                name = scriptData.name,
                code = scriptData.code
            })
        end
        local jsonData = HttpService:JSONEncode(dataToSave)
        writefile(SAVE_FILE, jsonData)
    end)
    if success then
        print("💾 Scripts saved successfully!")
    else
        warn("❌ Failed to save scripts: " .. tostring(err))
    end
end

-- Function to load scripts from file
local function loadScriptsFromFile()
    local success, result = pcall(function()
        if isfile(SAVE_FILE) then
            local jsonData = readfile(SAVE_FILE)
            return HttpService:JSONDecode(jsonData)
        end
        return {}
    end)
    if success and result then
        print("📂 Loaded " .. #result .. " scripts from file!")
        return result
    else
        warn("⚠️ Could not load scripts (first run or no file exists)")
        return {}
    end
end

-- Create Main GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ScriptSaverGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Destroy existing GUI if present
if player.PlayerGui:FindFirstChild("ScriptSaverGUI") then
    player.PlayerGui:FindFirstChild("ScriptSaverGUI"):Destroy()
end

ScreenGui.Parent = player.PlayerGui

-- Theme Color (Customizable)
local themeColor = Color3.fromRGB(59, 130, 246)

-- Toggle Button (Always Visible + Draggable)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0, 120, 0, 35)
ToggleButton.Position = UDim2.new(1, -130, 0, 50)
ToggleButton.BackgroundColor3 = Color3.fromRGB(139, 92, 246)
ToggleButton.BackgroundTransparency = 0.7
ToggleButton.Text = "👁 Hide UI"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 14
ToggleButton.Parent = ScreenGui

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleButton

local ToggleStroke = Instance.new("UIStroke")
ToggleStroke.Color = Color3.fromRGB(167, 139, 250)
ToggleStroke.Thickness = 2
ToggleStroke.Parent = ToggleButton

-- Main Frame
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 420, 0, 500)
MainFrame.Position = UDim2.new(0.5, -210, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 41, 59)
MainFrame.BackgroundTransparency = 1
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- UI Visibility State
local uiVisible = true

-- FIXED: Toggle Button Dragging - Only tracks the specific input that started on the button
local toggleDragging = false
local toggleDragStart = nil
local toggleStartPos = nil
local toggleDragDistance = 0
local toggleDragInput = nil

ToggleButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        toggleDragging = true
        toggleDragStart = input.Position
        toggleStartPos = ToggleButton.Position
        toggleDragDistance = 0
        toggleDragInput = input
    end
end)

ToggleButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if toggleDragging and toggleDragInput and input == toggleDragInput then
            local delta = input.Position - toggleDragStart
            toggleDragDistance = math.abs(delta.X) + math.abs(delta.Y)
            ToggleButton.Position = UDim2.new(toggleStartPos.X.Scale, toggleStartPos.X.Offset + delta.X, toggleStartPos.Y.Scale, toggleStartPos.Y.Offset + delta.Y)
        end
    end
end)

ToggleButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if input == toggleDragInput then
            if toggleDragDistance < 5 then
                uiVisible = not uiVisible
                MainFrame.Visible = uiVisible
                ToggleButton.Text = uiVisible and "👁 Hide UI" or "👁 Show UI"
            end
            toggleDragging = false
            toggleDragInput = nil
        end
    end
end)

-- Main Frame Styling
local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 16)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(59, 130, 246)
MainStroke.Thickness = 3
MainStroke.Parent = MainFrame

-- Title Bar
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 50)
TitleBar.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
TitleBar.BackgroundTransparency = 1
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 16)
TitleCorner.Parent = TitleBar

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 1, 0)
Title.BackgroundTransparency = 1
Title.Text = "🔥 SCRIPT SAVER"
Title.TextColor3 = Color3.fromRGB(0, 217, 255)
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 20
Title.Parent = TitleBar

-- FIXED: Main Frame Dragging - Only tracks the specific input that started on the title bar
local mainDragging = false
local mainDragStart = nil
local mainStartPos = nil
local mainDragInput = nil

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        mainDragging = true
        mainDragStart = input.Position
        mainStartPos = MainFrame.Position
        mainDragInput = input
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if mainDragging and mainDragInput and input == mainDragInput then
            local delta = input.Position - mainDragStart
            MainFrame.Position = UDim2.new(mainStartPos.X.Scale, mainStartPos.X.Offset + delta.X, mainStartPos.Y.Scale, mainStartPos.Y.Offset + delta.Y)
        end
    end
end)

TitleBar.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if input == mainDragInput then
            mainDragging = false
            mainDragInput = nil
        end
    end
end)

-- Add Script Section
local AddSection = Instance.new("Frame")
AddSection.Name = "AddSection"
AddSection.Size = UDim2.new(1, -20, 0, 115)
AddSection.Position = UDim2.new(0, 10, 0, 55)
AddSection.BackgroundColor3 = Color3.fromRGB(51, 65, 85)
AddSection.BackgroundTransparency = 0.5
AddSection.BorderSizePixel = 0
AddSection.Parent = MainFrame

local AddCorner = Instance.new("UICorner")
AddCorner.CornerRadius = UDim.new(0, 12)
AddCorner.Parent = AddSection

-- Script Name Input
local NameInput = Instance.new("TextBox")
NameInput.Name = "NameInput"
NameInput.Size = UDim2.new(1, -20, 0, 30)
NameInput.Position = UDim2.new(0, 10, 0, 10)
NameInput.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
NameInput.PlaceholderText = "Script Name..."
NameInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
NameInput.Text = ""
NameInput.TextColor3 = Color3.fromRGB(255, 255, 255)
NameInput.Font = Enum.Font.Gotham
NameInput.TextSize = 14
NameInput.TextXAlignment = Enum.TextXAlignment.Left
NameInput.ClearTextOnFocus = false
NameInput.Parent = AddSection

local NameCorner = Instance.new("UICorner")
NameCorner.CornerRadius = UDim.new(0, 8)
NameCorner.Parent = NameInput

local NamePadding = Instance.new("UIPadding")
NamePadding.PaddingLeft = UDim.new(0, 10)
NamePadding.Parent = NameInput

-- Script Code Input
local CodeInput = Instance.new("TextBox")
CodeInput.Name = "CodeInput"
CodeInput.Size = UDim2.new(1, -120, 0, 40)
CodeInput.Position = UDim2.new(0, 10, 0, 50)
CodeInput.BackgroundColor3 = Color3.fromRGB(13, 17, 23)
CodeInput.PlaceholderText = "Paste script code here..."
CodeInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
CodeInput.Text = ""
CodeInput.TextColor3 = Color3.fromRGB(0, 255, 100)
CodeInput.Font = Enum.Font.Code
CodeInput.TextSize = 12
CodeInput.TextXAlignment = Enum.TextXAlignment.Left
CodeInput.TextWrapped = true
CodeInput.ClearTextOnFocus = false
CodeInput.MultiLine = true
CodeInput.Parent = AddSection

local CodeCorner = Instance.new("UICorner")
CodeCorner.CornerRadius = UDim.new(0, 8)
CodeCorner.Parent = CodeInput

local CodePadding = Instance.new("UIPadding")
CodePadding.PaddingLeft = UDim.new(0, 10)
CodePadding.Parent = CodeInput

-- Add Button
local AddButton = Instance.new("TextButton")
AddButton.Name = "AddButton"
AddButton.Size = UDim2.new(0, 100, 0, 40)
AddButton.Position = UDim2.new(1, -110, 0, 50)
AddButton.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
AddButton.Text = "➕ Add"
AddButton.TextColor3 = Color3.fromRGB(255, 255, 255)
AddButton.Font = Enum.Font.GothamBold
AddButton.TextSize = 14
AddButton.Parent = AddSection

local AddBtnCorner = Instance.new("UICorner")
AddBtnCorner.CornerRadius = UDim.new(0, 8)
AddBtnCorner.Parent = AddButton

-- Scripts List Container
local ListContainer = Instance.new("Frame")
ListContainer.Name = "ListContainer"
ListContainer.Size = UDim2.new(1, -20, 1, -200)
ListContainer.Position = UDim2.new(0, 10, 0, 185)
ListContainer.BackgroundColor3 = Color3.fromRGB(51, 65, 85)
ListContainer.BackgroundTransparency = 0.5
ListContainer.BorderSizePixel = 0
ListContainer.ClipsDescendants = true
ListContainer.Parent = MainFrame

local ListCorner = Instance.new("UICorner")
ListCorner.CornerRadius = UDim.new(0, 12)
ListCorner.Parent = ListContainer

local ListTitle = Instance.new("TextLabel")
ListTitle.Size = UDim2.new(1, 0, 0, 30)
ListTitle.BackgroundTransparency = 1
ListTitle.Text = "📂 My Scripts"
ListTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ListTitle.Font = Enum.Font.GothamBold
ListTitle.TextSize = 14
ListTitle.Parent = ListContainer

-- Scrolling Frame for Scripts
local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Name = "ScrollFrame"
ScrollFrame.Size = UDim2.new(1, -10, 1, -35)
ScrollFrame.Position = UDim2.new(0, 5, 0, 30)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel = 0
ScrollFrame.ScrollBarThickness = 5
ScrollFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 217, 255)
ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
ScrollFrame.Parent = ListContainer

local ListLayout = Instance.new("UIListLayout")
ListLayout.SortOrder = Enum.SortOrder.LayoutOrder
ListLayout.Padding = UDim.new(0, 5)
ListLayout.Parent = ScrollFrame

-- Empty Message
local EmptyLabel = Instance.new("TextLabel")
EmptyLabel.Name = "EmptyLabel"
EmptyLabel.Size = UDim2.new(1, 0, 0, 50)
EmptyLabel.BackgroundTransparency = 1
EmptyLabel.Text = "No scripts saved yet! 🚀"
EmptyLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
EmptyLabel.Font = Enum.Font.Gotham
EmptyLabel.TextSize = 14
EmptyLabel.Parent = ScrollFrame

-- Update canvas size function
local function updateCanvasSize()
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, ListLayout.AbsoluteContentSize.Y + 10)
end

ListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateCanvasSize)

-- Color Picker Button
local ColorButton = Instance.new("TextButton")
ColorButton.Name = "ColorButton"
ColorButton.Size = UDim2.new(0, 35, 0, 35)
ColorButton.Position = UDim2.new(1, -45, 0, 7)
ColorButton.BackgroundColor3 = themeColor
ColorButton.Text = "🎨"
ColorButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ColorButton.Font = Enum.Font.GothamBold
ColorButton.TextSize = 16
ColorButton.Parent = TitleBar

local ColorBtnCorner = Instance.new("UICorner")
ColorBtnCorner.CornerRadius = UDim.new(0, 8)
ColorBtnCorner.Parent = ColorButton

-- Color Picker Frame
local ColorPickerFrame = Instance.new("Frame")
ColorPickerFrame.Name = "ColorPickerFrame"
ColorPickerFrame.Size = UDim2.new(0, 320, 0, 400)
ColorPickerFrame.Position = UDim2.new(0.5, -160, 0.5, -200)
ColorPickerFrame.BackgroundColor3 = Color3.fromRGB(30, 41, 59)
ColorPickerFrame.Visible = false
ColorPickerFrame.ZIndex = 10
ColorPickerFrame.Parent = MainFrame

local ColorPickerCorner = Instance.new("UICorner")
ColorPickerCorner.CornerRadius = UDim.new(0, 12)
ColorPickerCorner.Parent = ColorPickerFrame

local ColorPickerStroke = Instance.new("UIStroke")
ColorPickerStroke.Color = themeColor
ColorPickerStroke.Thickness = 2
ColorPickerStroke.Parent = ColorPickerFrame

local ColorPickerTitle = Instance.new("TextLabel")
ColorPickerTitle.Size = UDim2.new(1, 0, 0, 30)
ColorPickerTitle.BackgroundTransparency = 1
ColorPickerTitle.Text = "🎨 Choose UI Color"
ColorPickerTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ColorPickerTitle.Font = Enum.Font.GothamBold
ColorPickerTitle.TextSize = 14
ColorPickerTitle.ZIndex = 10
ColorPickerTitle.Parent = ColorPickerFrame

-- Color Options
local colors = {
    {name = "Red", color = Color3.fromRGB(239, 68, 68)},
    {name = "Crimson", color = Color3.fromRGB(220, 20, 60)},
    {name = "Scarlet", color = Color3.fromRGB(255, 36, 0)},
    {name = "Rose", color = Color3.fromRGB(255, 0, 127)},
    {name = "Coral", color = Color3.fromRGB(255, 127, 80)},
    {name = "Salmon", color = Color3.fromRGB(250, 128, 114)},
    {name = "Orange", color = Color3.fromRGB(249, 115, 22)},
    {name = "Amber", color = Color3.fromRGB(255, 191, 0)},
    {name = "Peach", color = Color3.fromRGB(255, 218, 185)},
    {name = "Rust", color = Color3.fromRGB(183, 65, 14)},
    {name = "Yellow", color = Color3.fromRGB(234, 179, 8)},
    {name = "Gold", color = Color3.fromRGB(255, 215, 0)},
    {name = "Lemon", color = Color3.fromRGB(255, 247, 0)},
    {name = "Cream", color = Color3.fromRGB(255, 253, 208)},
    {name = "Green", color = Color3.fromRGB(34, 197, 94)},
    {name = "Lime", color = Color3.fromRGB(50, 205, 50)},
    {name = "Emerald", color = Color3.fromRGB(0, 201, 87)},
    {name = "Mint", color = Color3.fromRGB(62, 180, 137)},
    {name = "Forest", color = Color3.fromRGB(34, 139, 34)},
    {name = "Olive", color = Color3.fromRGB(128, 128, 0)},
    {name = "Cyan", color = Color3.fromRGB(6, 182, 212)},
    {name = "Teal", color = Color3.fromRGB(0, 128, 128)},
    {name = "Aqua", color = Color3.fromRGB(0, 255, 255)},
    {name = "Turq", color = Color3.fromRGB(64, 224, 208)},
    {name = "Blue", color = Color3.fromRGB(59, 130, 246)},
    {name = "Sky", color = Color3.fromRGB(135, 206, 235)},
    {name = "Navy", color = Color3.fromRGB(0, 0, 128)},
    {name = "Cobalt", color = Color3.fromRGB(0, 71, 171)},
    {name = "Azure", color = Color3.fromRGB(0, 127, 255)},
    {name = "Purple", color = Color3.fromRGB(139, 92, 246)},
    {name = "Indigo", color = Color3.fromRGB(75, 0, 130)},
    {name = "Violet", color = Color3.fromRGB(138, 43, 226)},
    {name = "Lavend", color = Color3.fromRGB(230, 230, 250)},
    {name = "Plum", color = Color3.fromRGB(221, 160, 221)},
    {name = "Pink", color = Color3.fromRGB(236, 72, 153)},
    {name = "Magenta", color = Color3.fromRGB(255, 0, 255)},
    {name = "Fuchsia", color = Color3.fromRGB(255, 0, 255)},
    {name = "HotPink", color = Color3.fromRGB(255, 105, 180)},
    {name = "White", color = Color3.fromRGB(255, 255, 255)},
    {name = "Silver", color = Color3.fromRGB(192, 192, 192)},
    {name = "Gray", color = Color3.fromRGB(128, 128, 128)},
    {name = "Black", color = Color3.fromRGB(0, 0, 0)},
}

local ColorGrid = Instance.new("Frame")
ColorGrid.Size = UDim2.new(1, -20, 0, 300)
ColorGrid.Position = UDim2.new(0, 10, 0, 35)
ColorGrid.BackgroundTransparency = 1
ColorGrid.ZIndex = 10
ColorGrid.Parent = ColorPickerFrame

local ColorGridLayout = Instance.new("UIGridLayout")
ColorGridLayout.CellSize = UDim2.new(0, 65, 0, 28)
ColorGridLayout.CellPadding = UDim2.new(0, 5, 0, 5)
ColorGridLayout.SortOrder = Enum.SortOrder.LayoutOrder
ColorGridLayout.Parent = ColorGrid

-- Function to save color to file
local function saveColorToFile(color, isRainbow)
    local success, err = pcall(function()
        local colorData = {
            isRainbow = isRainbow or false,
            color = {R = math.floor(color.R * 255), G = math.floor(color.G * 255), B = math.floor(color.B * 255)}
        }
        writefile(COLOR_FILE, HttpService:JSONEncode(colorData))
    end)
    if success then
        print("🎨 Color saved!")
    end
end

-- Function to update theme color with smooth animation
local function updateThemeColor(newColor, save)
    themeColor = newColor
    
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    
    TweenService:Create(MainStroke, tweenInfo, {Color = newColor}):Play()
    TweenService:Create(ColorPickerStroke, tweenInfo, {Color = newColor}):Play()
    TweenService:Create(ColorButton, tweenInfo, {BackgroundColor3 = newColor}):Play()
    TweenService:Create(Title, tweenInfo, {TextColor3 = newColor}):Play()
    TweenService:Create(ScrollFrame, tweenInfo, {ScrollBarImageColor3 = newColor}):Play()
    
    if save then
        saveColorToFile(newColor, false)
    end
end

-- Create color buttons
for i, colorData in ipairs(colors) do
    local ColorBtn = Instance.new("TextButton")
    ColorBtn.Name = colorData.name
    ColorBtn.Size = UDim2.new(0, 60, 0, 30)
    ColorBtn.BackgroundColor3 = colorData.color
    ColorBtn.Text = colorData.name
    ColorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    ColorBtn.Font = Enum.Font.GothamBold
    ColorBtn.TextSize = 10
    ColorBtn.ZIndex = 10
    ColorBtn.LayoutOrder = i
    ColorBtn.Parent = ColorGrid
    
    local BtnCorner = Instance.new("UICorner")
    BtnCorner.CornerRadius = UDim.new(0, 6)
    BtnCorner.Parent = ColorBtn
    
    ColorBtn.MouseButton1Click:Connect(function()
        if rainbowConnection then
            rainbowConnection:Disconnect()
            rainbowConnection = nil
        end
        rainbowEnabled = false
        updateThemeColor(colorData.color, true)
        ColorPickerFrame.Visible = false
    end)
end

-- Rainbow Button
local RainbowBtn = Instance.new("TextButton")
RainbowBtn.Name = "Rainbow"
RainbowBtn.Size = UDim2.new(1, -20, 0, 30)
RainbowBtn.Position = UDim2.new(0, 10, 0, 340)
RainbowBtn.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
RainbowBtn.Text = "🌈 Rainbow Mode"
RainbowBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RainbowBtn.Font = Enum.Font.GothamBold
RainbowBtn.TextSize = 12
RainbowBtn.ZIndex = 10
RainbowBtn.Parent = ColorPickerFrame

local RainbowCorner = Instance.new("UICorner")
RainbowCorner.CornerRadius = UDim.new(0, 6)
RainbowCorner.Parent = RainbowBtn

-- Function to start rainbow mode
local function startRainbowMode()
    if rainbowConnection then
        rainbowConnection:Disconnect()
    end
    
    rainbowEnabled = true
    rainbowHue = 0
    
    rainbowConnection = RunService.Heartbeat:Connect(function(dt)
        rainbowHue = (rainbowHue + dt * 0.2) % 1
        local rainbowColor = Color3.fromHSV(rainbowHue, 1, 1)
        
        MainStroke.Color = rainbowColor
        ColorPickerStroke.Color = rainbowColor
        ColorButton.BackgroundColor3 = rainbowColor
        Title.TextColor3 = rainbowColor
        ScrollFrame.ScrollBarImageColor3 = rainbowColor
        RainbowBtn.BackgroundColor3 = rainbowColor
    end)
end

RainbowBtn.MouseButton1Click:Connect(function()
    startRainbowMode()
    saveColorToFile(Color3.fromRGB(255, 0, 0), true)
    ColorPickerFrame.Visible = false
end)

-- Close Color Picker Button
local CloseColorBtn = Instance.new("TextButton")
CloseColorBtn.Size = UDim2.new(1, -20, 0, 25)
CloseColorBtn.Position = UDim2.new(0, 10, 1, -35)
CloseColorBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
CloseColorBtn.Text = "Close"
CloseColorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseColorBtn.Font = Enum.Font.GothamBold
CloseColorBtn.TextSize = 12
CloseColorBtn.ZIndex = 10
CloseColorBtn.Parent = ColorPickerFrame

local CloseColorCorner = Instance.new("UICorner")
CloseColorCorner.CornerRadius = UDim.new(0, 6)
CloseColorCorner.Parent = CloseColorBtn

ColorButton.MouseButton1Click:Connect(function()
    ColorPickerFrame.Visible = not ColorPickerFrame.Visible
end)

CloseColorBtn.MouseButton1Click:Connect(function()
    ColorPickerFrame.Visible = false
end)

-- Edit Script Frame
local EditFrame = Instance.new("Frame")
EditFrame.Name = "EditFrame"
EditFrame.Size = UDim2.new(1, -40, 0, 250)
EditFrame.Position = UDim2.new(0, 20, 0.5, -125)
EditFrame.BackgroundColor3 = Color3.fromRGB(20, 30, 48)
EditFrame.Visible = false
EditFrame.ZIndex = 15
EditFrame.Parent = MainFrame

local EditFrameCorner = Instance.new("UICorner")
EditFrameCorner.CornerRadius = UDim.new(0, 12)
EditFrameCorner.Parent = EditFrame

local EditFrameStroke = Instance.new("UIStroke")
EditFrameStroke.Color = Color3.fromRGB(249, 115, 22)
EditFrameStroke.Thickness = 2
EditFrameStroke.Parent = EditFrame

local EditTitle = Instance.new("TextLabel")
EditTitle.Size = UDim2.new(1, 0, 0, 35)
EditTitle.BackgroundTransparency = 1
EditTitle.Text = "📝 Edit Script"
EditTitle.TextColor3 = Color3.fromRGB(249, 115, 22)
EditTitle.Font = Enum.Font.GothamBold
EditTitle.TextSize = 16
EditTitle.ZIndex = 15
EditTitle.Parent = EditFrame

local EditCodeBox = Instance.new("TextBox")
EditCodeBox.Name = "EditCodeBox"
EditCodeBox.Size = UDim2.new(1, -20, 1, -100)
EditCodeBox.Position = UDim2.new(0, 10, 0, 40)
EditCodeBox.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
EditCodeBox.TextColor3 = Color3.fromRGB(255, 255, 255)
EditCodeBox.Font = Enum.Font.Code
EditCodeBox.TextSize = 14
EditCodeBox.MultiLine = true
EditCodeBox.ClearTextOnFocus = false
EditCodeBox.TextXAlignment = Enum.TextXAlignment.Left
EditCodeBox.TextYAlignment = Enum.TextYAlignment.Top
EditCodeBox.TextWrapped = true
EditCodeBox.ZIndex = 15
EditCodeBox.Parent = EditFrame

local EditCorner = Instance.new("UICorner")
EditCorner.CornerRadius = UDim.new(0, 8)
EditCorner.Parent = EditCodeBox

local EditCodePadding = Instance.new("UIPadding")
EditCodePadding.PaddingLeft = UDim.new(0, 10)
EditCodePadding.PaddingTop = UDim.new(0, 5)
EditCodePadding.Parent = EditCodeBox

-- Edit Frame Buttons
local SaveEditBtn = Instance.new("TextButton")
SaveEditBtn.Name = "SaveEditBtn"
SaveEditBtn.Size = UDim2.new(0, 100, 0, 35)
SaveEditBtn.Position = UDim2.new(1, -110, 1, -45)
SaveEditBtn.BackgroundColor3 = Color3.fromRGB(39, 174, 96)
SaveEditBtn.Text = "💾 Save"
SaveEditBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SaveEditBtn.Font = Enum.Font.GothamBold
SaveEditBtn.TextSize = 14
SaveEditBtn.ZIndex = 15
SaveEditBtn.Parent = EditFrame

local SaveEditCorner = Instance.new("UICorner")
SaveEditCorner.CornerRadius = UDim.new(0, 8)
SaveEditCorner.Parent = SaveEditBtn

local CancelEditBtn = Instance.new("TextButton")
CancelEditBtn.Name = "CancelEditBtn"
CancelEditBtn.Size = UDim2.new(0, 100, 0, 35)
CancelEditBtn.Position = UDim2.new(0, 10, 1, -45)
CancelEditBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
CancelEditBtn.Text = "❌ Cancel"
CancelEditBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CancelEditBtn.Font = Enum.Font.GothamBold
CancelEditBtn.TextSize = 14
CancelEditBtn.ZIndex = 15
CancelEditBtn.Parent = EditFrame

local CancelEditCorner = Instance.new("UICorner")
CancelEditCorner.CornerRadius = UDim.new(0, 8)
CancelEditCorner.Parent = CancelEditBtn

-- Logic Variables
local currentEditingIndex = nil
local refreshScriptList

CancelEditBtn.MouseButton1Click:Connect(function()
    EditFrame.Visible = false
    currentEditingIndex = nil
end)

SaveEditBtn.MouseButton1Click:Connect(function()
    if currentEditingIndex and savedScripts[currentEditingIndex] then
        savedScripts[currentEditingIndex].code = EditCodeBox.Text
        saveScriptsToFile()
        refreshScriptList()
    end
    EditFrame.Visible = false
    currentEditingIndex = nil
end)

-- Confirmation Frame
local ConfirmFrame = Instance.new("Frame")
ConfirmFrame.Name = "ConfirmFrame"
ConfirmFrame.Size = UDim2.new(0, 260, 0, 140)
ConfirmFrame.Position = UDim2.new(0.5, -130, 0.5, -70)
ConfirmFrame.BackgroundColor3 = Color3.fromRGB(30, 41, 59)
ConfirmFrame.Visible = false
ConfirmFrame.ZIndex = 20
ConfirmFrame.Parent = MainFrame

local ConfirmCorner = Instance.new("UICorner")
ConfirmCorner.CornerRadius = UDim.new(0, 12)
ConfirmCorner.Parent = ConfirmFrame

local ConfirmStroke = Instance.new("UIStroke")
ConfirmStroke.Color = Color3.fromRGB(239, 68, 68)
ConfirmStroke.Thickness = 2
ConfirmStroke.Parent = ConfirmFrame

local ConfirmTitle = Instance.new("TextLabel")
ConfirmTitle.Size = UDim2.new(1, 0, 0, 40)
ConfirmTitle.Position = UDim2.new(0, 0, 0, 5)
ConfirmTitle.BackgroundTransparency = 1
ConfirmTitle.Text = "⚠️ Are you sure?"
ConfirmTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmTitle.Font = Enum.Font.GothamBold
ConfirmTitle.TextSize = 18
ConfirmTitle.ZIndex = 20
ConfirmTitle.Parent = ConfirmFrame

local ConfirmDesc = Instance.new("TextLabel")
ConfirmDesc.Size = UDim2.new(1, -20, 0, 30)
ConfirmDesc.Position = UDim2.new(0, 10, 0, 40)
ConfirmDesc.BackgroundTransparency = 1
ConfirmDesc.Text = "Delete this script permanently?"
ConfirmDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
ConfirmDesc.Font = Enum.Font.Gotham
ConfirmDesc.TextSize = 14
ConfirmDesc.TextWrapped = true
ConfirmDesc.ZIndex = 20
ConfirmDesc.Parent = ConfirmFrame

local ConfirmYesBtn = Instance.new("TextButton")
ConfirmYesBtn.Size = UDim2.new(0, 100, 0, 35)
ConfirmYesBtn.Position = UDim2.new(0, 20, 1, -45)
ConfirmYesBtn.BackgroundColor3 = Color3.fromRGB(239, 68, 68)
ConfirmYesBtn.Text = "Yes, Delete"
ConfirmYesBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmYesBtn.Font = Enum.Font.GothamBold
ConfirmYesBtn.TextSize = 14
ConfirmYesBtn.ZIndex = 20
ConfirmYesBtn.Parent = ConfirmFrame

local ConfirmYesCorner = Instance.new("UICorner")
ConfirmYesCorner.CornerRadius = UDim.new(0, 8)
ConfirmYesCorner.Parent = ConfirmYesBtn

local ConfirmNoBtn = Instance.new("TextButton")
ConfirmNoBtn.Size = UDim2.new(0, 100, 0, 35)
ConfirmNoBtn.Position = UDim2.new(1, -120, 1, -45)
ConfirmNoBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
ConfirmNoBtn.Text = "Cancel"
ConfirmNoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ConfirmNoBtn.Font = Enum.Font.GothamBold
ConfirmNoBtn.TextSize = 14
ConfirmNoBtn.ZIndex = 20
ConfirmNoBtn.Parent = ConfirmFrame

local ConfirmNoCorner = Instance.new("UICorner")
ConfirmNoCorner.CornerRadius = UDim.new(0, 8)
ConfirmNoCorner.Parent = ConfirmNoBtn

-- Logic for Deletion
local scriptToDeleteIndex = nil

ConfirmNoBtn.MouseButton1Click:Connect(function()
    ConfirmFrame.Visible = false
    scriptToDeleteIndex = nil
end)

ConfirmYesBtn.MouseButton1Click:Connect(function()
    if scriptToDeleteIndex and savedScripts[scriptToDeleteIndex] then
        table.remove(savedScripts, scriptToDeleteIndex)
        saveScriptsToFile()
        refreshScriptList()
    end
    ConfirmFrame.Visible = false
    scriptToDeleteIndex = nil
end)

-- Resize Handle
local ResizeHandle = Instance.new("TextButton")
ResizeHandle.Name = "ResizeHandle"
ResizeHandle.Size = UDim2.new(0, 20, 0, 20)
ResizeHandle.Position = UDim2.new(1, -20, 1, -20)
ResizeHandle.BackgroundTransparency = 1
ResizeHandle.Text = "◢"
ResizeHandle.TextColor3 = Color3.fromRGB(100, 100, 100)
ResizeHandle.TextSize = 14
ResizeHandle.Rotation = 0
ResizeHandle.Parent = MainFrame

local resizing = false
local resizeStart = nil
local startSize = nil
local resizeInput = nil
local MIN_SIZE_X = 420
local MIN_SIZE_Y = 400

ResizeHandle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        resizing = true
        resizeStart = input.Position
        startSize = MainFrame.AbsoluteSize
        resizeInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if resizing and input == resizeInput then
        local delta = input.Position - resizeStart
        local newWidth = math.max(MIN_SIZE_X, startSize.X + delta.X)
        local newHeight = math.max(MIN_SIZE_Y, startSize.Y + delta.Y)
        MainFrame.Size = UDim2.new(0, newWidth, 0, newHeight)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input == resizeInput then
        resizing = false
        resizeInput = nil
    end
end)

-- Function to refresh the scripts list
refreshScriptList = function()
    for _, child in ipairs(ScrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end

    if #savedScripts == 0 then
        EmptyLabel.Visible = true
    else
        EmptyLabel.Visible = false
    end

    for i, scriptData in ipairs(savedScripts) do
        local ScriptItem = Instance.new("Frame")
        ScriptItem.Name = "ScriptItem_" .. i
        ScriptItem.Size = UDim2.new(1, 0, 0, 40)
        ScriptItem.BackgroundColor3 = Color3.fromRGB(15, 23, 42)
        ScriptItem.BorderSizePixel = 0
        ScriptItem.LayoutOrder = i
        ScriptItem.Parent = ScrollFrame

        local ItemCorner = Instance.new("UICorner")
        ItemCorner.CornerRadius = UDim.new(0, 8)
        ItemCorner.Parent = ScriptItem

        local ScriptName = Instance.new("TextLabel")
        ScriptName.Size = UDim2.new(1, -150, 1, 0)
        ScriptName.Position = UDim2.new(0, 10, 0, 0)
        ScriptName.BackgroundTransparency = 1
        ScriptName.Text = scriptData.name
        ScriptName.TextColor3 = Color3.fromRGB(255, 255, 255)
        ScriptName.Font = Enum.Font.GothamSemibold
        ScriptName.TextSize = 14
        ScriptName.TextXAlignment = Enum.TextXAlignment.Left
        ScriptName.TextTruncate = Enum.TextTruncate.AtEnd
        ScriptName.Parent = ScriptItem

        local BtnContainer = Instance.new("Frame")
        BtnContainer.Size = UDim2.new(0, 140, 1, 0)
        BtnContainer.Position = UDim2.new(1, -140, 0, 0)
        BtnContainer.BackgroundTransparency = 1
        BtnContainer.Parent = ScriptItem

        local BtnLayout = Instance.new("UIListLayout")
        BtnLayout.FillDirection = Enum.FillDirection.Horizontal
        BtnLayout.HorizontalAlignment = Enum.HorizontalAlignment.Right
        BtnLayout.SortOrder = Enum.SortOrder.LayoutOrder
        BtnLayout.Padding = UDim.new(0, 5)
        BtnLayout.Parent = BtnContainer

        local BtnPadding = Instance.new("UIPadding")
        BtnPadding.PaddingRight = UDim.new(0, 5)
        BtnPadding.PaddingTop = UDim.new(0, 5)
        BtnPadding.PaddingBottom = UDim.new(0, 5)
        BtnPadding.Parent = BtnContainer

        local function createSmallBtn(text, color, layoutOrder)
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(0, 30, 0, 30)
            btn.BackgroundColor3 = color
            btn.Text = text
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            btn.TextSize = 12
            btn.Font = Enum.Font.GothamBold
            btn.LayoutOrder = layoutOrder
            btn.Parent = BtnContainer
            
            local c = Instance.new("UICorner")
            c.CornerRadius = UDim.new(0, 6)
            c.Parent = btn
            return btn
        end

        local MoveBtn = createSmallBtn("↕", Color3.fromRGB(59, 130, 246), 1)
        
        MoveBtn.MouseButton1Click:Connect(function()
            if i > 1 then
                local temp = savedScripts[i]
                savedScripts[i] = savedScripts[i-1]
                savedScripts[i-1] = temp
                saveScriptsToFile()
                refreshScriptList()
            end
        end)
        
        MoveBtn.MouseButton2Click:Connect(function()
            if i < #savedScripts then
                local temp = savedScripts[i]
                savedScripts[i] = savedScripts[i+1]
                savedScripts[i+1] = temp
                saveScriptsToFile()
                refreshScriptList()
            end
        end)

        local ExecBtn = createSmallBtn("▶", Color3.fromRGB(39, 174, 96), 2)
        ExecBtn.MouseButton1Click:Connect(function()
            local success, err = pcall(function()
                loadstring(scriptData.code)()
            end)
            if not success then
                warn("Script Execution Error: " .. tostring(err))
            end
        end)
        
        local EditBtn = createSmallBtn("✏️", Color3.fromRGB(249, 115, 22), 3)
        EditBtn.MouseButton1Click:Connect(function()
            currentEditingIndex = i
            EditCodeBox.Text = scriptData.code
            EditFrame.Visible = true
        end)

        local DelBtn = createSmallBtn("🗑", Color3.fromRGB(239, 68, 68), 4)
        DelBtn.MouseButton1Click:Connect(function()
            scriptToDeleteIndex = i
            ConfirmFrame.Visible = true
            ConfirmFrame.Parent = MainFrame
        end)
    end
end

-- Add Button Logic
AddButton.MouseButton1Click:Connect(function()
    local name = NameInput.Text
    local code = CodeInput.Text

    if name == "" or code == "" then
        return 
    end

    table.insert(savedScripts, {
        id = HttpService:GenerateGUID(false),
        name = name,
        code = code
    })

    saveScriptsToFile()
    refreshScriptList()
    
    NameInput.Text = ""
    CodeInput.Text = ""
end)

-- Initial Load
savedScripts = loadScriptsFromFile()
refreshScriptList()

-- Load Color Settings
local function loadColorSettings()
    if isfile(COLOR_FILE) then
        local success, result = pcall(function()
            return HttpService:JSONDecode(readfile(COLOR_FILE))
        end)
        if success and result then
            if result.color then
                local col = Color3.fromRGB(result.color.R, result.color.G, result.color.B)
                updateThemeColor(col, false)
            end
            if result.isRainbow then
                startRainbowMode()
            end
        end
    end
end
task.spawn(loadColorSettings)

print("Script Saver GUI Loaded")
