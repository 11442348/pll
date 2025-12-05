--[[
UI MUSIC VIEWER + PLAYER TRACKER
By ChatGPT
]]

-- Services
local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

---------------------------------------------------
-- CREATE UI
---------------------------------------------------
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.ResetOnSpawn = false
ScreenGui.Name = "MusicViewerUI"

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 350, 0, 430)
Main.Position = UDim2.new(0.2, 0, 0.2, 0)
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Main.Active = true
Main.Draggable = true -- UI ลากได้

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 35)
Title.Text = "🎵 Music Viewer"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18

local CloseBtn = Instance.new("TextButton", Title)
CloseBtn.Size = UDim2.new(0, 35, 0, 35)
CloseBtn.Position = UDim2.new(1, -35, 0, 0)
CloseBtn.Text = "X"
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextColor3 = Color3.new(1, 1, 1)
CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 40, 40)

local PlayerList = Instance.new("ScrollingFrame", Main)
PlayerList.Size = UDim2.new(1, -20, 0, 150)
PlayerList.Position = UDim2.new(0, 10, 0, 45)
PlayerList.CanvasSize = UDim2.new(0, 0, 0, 0)
PlayerList.ScrollBarThickness = 4
PlayerList.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

local InfoFrame = Instance.new("Frame", Main)
InfoFrame.Size = UDim2.new(1, -20, 0, 230)
InfoFrame.Position = UDim2.new(0, 10, 0, 205)
InfoFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)

---------------------------------------------------
-- UI ELEMENTS inside InfoFrame
---------------------------------------------------
local Cover = Instance.new("ImageLabel", InfoFrame)
Cover.Size = UDim2.new(0, 150, 0, 150)
Cover.Position = UDim2.new(0, 10, 0, 10)
Cover.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
Cover.Image = ""

local SongName = Instance.new("TextLabel", InfoFrame)
SongName.Size = UDim2.new(1, -170, 0, 40)
SongName.Position = UDim2.new(0, 170, 0, 10)
SongName.Text = "Song: -"
SongName.Font = Enum.Font.Gotham
SongName.TextColor3 = Color3.new(1, 1, 1)
SongName.TextSize = 16
SongName.TextXAlignment = Enum.TextXAlignment.Left

local SongIdLabel = Instance.new("TextLabel", InfoFrame)
SongIdLabel.Size = UDim2.new(1, -170, 0, 40)
SongIdLabel.Position = UDim2.new(0, 170, 0, 55)
SongIdLabel.Text = "ID: -"
SongIdLabel.Font = Enum.Font.Gotham
SongIdLabel.TextColor3 = Color3.new(1, 1, 1)
SongIdLabel.TextSize = 16
SongIdLabel.TextXAlignment = Enum.TextXAlignment.Left

local CopyBtn = Instance.new("TextButton", InfoFrame)
CopyBtn.Size = UDim2.new(0, 100, 0, 32)
CopyBtn.Position = UDim2.new(0, 170, 0, 100)
CopyBtn.Text = "Copy ID"
CopyBtn.TextColor3 = Color3.new(1, 1, 1)
CopyBtn.BackgroundColor3 = Color3.fromRGB(40, 120, 40)
CopyBtn.Font = Enum.Font.GothamBold

local PreviewBtn = Instance.new("TextButton", InfoFrame)
PreviewBtn.Size = UDim2.new(0, 100, 0, 32)
PreviewBtn.Position = UDim2.new(0, 280, 0, 100)
PreviewBtn.Text = "▶ Preview"
PreviewBtn.TextColor3 = Color3.new(1, 1, 1)
PreviewBtn.BackgroundColor3 = Color3.fromRGB(40, 90, 140)
PreviewBtn.Font = Enum.Font.GothamBold

local TimeLabel = Instance.new("TextLabel", InfoFrame)
TimeLabel.Size = UDim2.new(1, -20, 0, 40)
TimeLabel.Position = UDim2.new(0, 10, 0, 170)
TimeLabel.Text = "Time: 0 / 0"
TimeLabel.Font = Enum.Font.Gotham
TimeLabel.TextColor3 = Color3.new(1, 1, 1)
TimeLabel.TextSize = 16
TimeLabel.TextXAlignment = Enum.TextXAlignment.Left

---------------------------------------------------
-- FUNCTIONS
---------------------------------------------------

local SelectedPlayer = nil
local PreviewSound = nil

-- ค้นหา Sound ที่ผู้เล่นกำลังเปิดจริง
local function GetPlayerSound(plr)
    if not plr.Character then return nil end

    for _, v in pairs(plr.Character:GetDescendants()) do
        if v:IsA("Sound") and v.IsPlaying then
            return v
        end
    end
    return nil
end

-- อัปเดตข้อมูลเพลง
local function UpdateSongInfo(plr)
    local sound = GetPlayerSound(plr)

    if not sound then
        SongName.Text = "Song: -"
        SongIdLabel.Text = "ID: -"
        TimeLabel.Text = "Time: 0 / 0"
        Cover.Image = ""
        return
    end

    local id = sound.SoundId:gsub("rbxassetid://", "")
    SongName.Text = "Song: " .. (sound.Name or "Unknown")
    SongIdLabel.Text = "ID: " .. id
    Cover.Image = "rbxthumb://type=Asset&id=" .. id .. "&w=150&h=150"

    task.spawn(function()
        while SelectedPlayer == plr do
            TimeLabel.Text = string.format("Time: %.1f / %.1f", sound.TimePosition, sound.TimeLength)
            task.wait(0.2)
        end
    end)
end

-- ปุ่มคัดลอก
CopyBtn.MouseButton1Click:Connect(function()
    if SelectedPlayer then
        local sound = GetPlayerSound(SelectedPlayer)
        if sound then
            setclipboard(sound.SoundId:gsub("rbxassetid://", ""))
        end
    end
end)

-- ปุ่ม Preview
PreviewBtn.MouseButton1Click:Connect(function()
    if PreviewSound then PreviewSound:Destroy() end

    if SelectedPlayer then
        local sound = GetPlayerSound(SelectedPlayer)
        if sound then
            local id = sound.SoundId
            PreviewSound = Instance.new("Sound", workspace)
            PreviewSound.SoundId = id
            PreviewSound:Play()
        end
    end
end)

-- ปิด UI
CloseBtn.MouseButton1Click:Connect(function()
    Main.Visible = not Main.Visible
end)

---------------------------------------------------
-- CREATE PLAYER BUTTONS
---------------------------------------------------
local function RefreshPlayerList()
    for _, v in ipairs(PlayerList:GetChildren()) do
        if v:IsA("TextButton") then v:Destroy() end
    end

    local Y = 0
    for _, plr in pairs(Players:GetPlayers()) do
        local btn = Instance.new("TextButton", PlayerList)
        btn.Size = UDim2.new(1, -10, 0, 35)
        btn.Position = UDim2.new(0, 5, 0, Y)
        btn.Text = plr.Name
        btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.Font = Enum.Font.GothamBold

        btn.MouseButton1Click:Connect(function()
            SelectedPlayer = plr
            UpdateSongInfo(plr)
        end)

        Y = Y + 40
    end

    PlayerList.CanvasSize = UDim2.new(0, 0, 0, Y)
end

RefreshPlayerList()
Players.PlayerAdded:Connect(RefreshPlayerList)
Players.PlayerRemoving:Connect(RefreshPlayerList
