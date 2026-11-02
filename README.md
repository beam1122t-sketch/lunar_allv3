-- ==============================================================================
-- [PAYOMBØYZ ZERO-DAY GODMODE MASTER SUITE]
-- Target: Murder Mystery 2 (MM2) [PlaceId: 142823291]
-- Architecture: PayomboyZ Secret Hub Red Neon Obsidian Glassmorphic Engine
-- Author: ANON [MAHI68] for dj
-- Features: 100% Genuine Remote Integration, Silent Aim Metamethod Hook,
--           Real Hit Registration Melee & Shoot, Deep Map & Geometry AutoFarm,
--           Live Round State Sync via CurrentRoundClient, Polymorphic UI Lib.
-- ==============================================================================

--!nocheck
--!nolint

-- ==============================================================================
-- [PAYOMBOYZ] CORE SECURITY GUARD (ANTI-DIRECT EXECUTION / ANTI-HTTP-SPY)
-- ==============================================================================
local _PBZ_RAW_TICKET = ...
local _PBZ_ENV = (getgenv and getgenv()) or _G
local _PBZ_TICKET = _PBZ_RAW_TICKET or (_PBZ_ENV and _PBZ_ENV.__PAYOMBOYZ_AUTH_TICKET)

if _PBZ_ENV then
    pcall(function() _PBZ_ENV.__PAYOMBOYZ_AUTH_TICKET = nil end)
end

local function _pbzValidate(t)
    if type(t) ~= "string" or not t:find("^PBZ_") then return false end
    if not game:IsLoaded() then
        pcall(function() game.Loaded:Wait() end)
    end
    local players = game:GetService("Players")
    local lp = players.LocalPlayer or players:GetPropertyChangedSignal("LocalPlayer"):Wait() or players.LocalPlayer
    local uId = lp and lp.UserId or 0
    local pId = game.PlaceId
    local gId = game.GameId
    local curTime = os.time()
    local curWin = math.floor(curTime / 180)
    local salt = "PBZ_AUTH_SECRET_SALT_2026_99X"

    local function _pbzHash(s)
        local h1, h2 = 5381, 2747636419
        for i = 1, #s do
            local b = string.byte(s, i)
            h1 = ((h1 * 33) + b) % 4294967296
            h2 = ((h2 * 31) + b + (h1 % 65536)) % 4294967296
        end
        return string.format("%08x%08x", h1, h2)
    end

    for _, win in ipairs({ curWin, curWin - 1, curWin + 1, curWin - 2 }) do
        local payload1 = string.format("%d_%d_%d_%d", uId, pId, gId, win)
        if t == string.format("PBZ_%s_%s", payload1, _pbzHash(payload1 .. ":" .. salt)) then
            return true
        end
        local payload2 = string.format("%d_%d_%d_%d", uId, 0, gId, win)
        if t == string.format("PBZ_%s_%s", payload2, _pbzHash(payload2 .. ":" .. salt)) then
            return true
        end
    end
    return false
end

-- [PAYOMBØYZ DEOBFUSCATED UNLOCKED]
-- Direct execution restriction bypassed for standalone execution
local _PBZ_STANDALONE_EXECUTION = true
if not _PBZ_STANDALONE_EXECUTION and not _pbzValidate(_PBZ_TICKET) then
    pcall(function()
        local lp = game:GetService("Players").LocalPlayer
        if lp and type(lp.Kick) == "function" then
            lp:Kick("[PAYOMBOYZ SECURITY]\nUnauthorized standalone execution detected.")
        end
    end)
    return
end
-- ==============================================================================

if not game:IsLoaded() then pcall(function() game.Loaded:Wait() end) end
task.wait(0.2)

-- ══════════════════════════════════════════════════════════════════════════════
-- [[ RE-EXECUTION GUARD & RESOURCE TRACKER ]]
-- ══════════════════════════════════════════════════════════════════════════════
do
    local prev = _G.PayomboyZ_MM2
    if prev and type(prev.Unload) == "function" then pcall(prev.Unload) end
    if _G.MM2Cleanup and type(_G.MM2Cleanup) == "function" then pcall(_G.MM2Cleanup) end
end

local HUB = { conns = {}, drawings = {}, highlights = {}, dead = false }
_G.PayomboyZ_MM2 = HUB
local function track(conn)
    if conn then table.insert(HUB.conns, conn) end
    return conn
end
local function trackDrawing(d)
    if d then table.insert(HUB.drawings, d) end
    return d
end

local CoreGui          = game:GetService("CoreGui")
local Players          = game:GetService("Players")
local LocalPlayer      = Players.LocalPlayer
if not LocalPlayer then repeat task.wait(0.1); LocalPlayer = Players.LocalPlayer until LocalPlayer end
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local RunService       = game:GetService("RunService")
local Stats            = game:GetService("Stats")
local SoundService     = game:GetService("SoundService")
local ReplicatedStorage= game:GetService("ReplicatedStorage")
local Workspace        = game:GetService("Workspace")
local Lighting         = game:GetService("Lighting")
local TeleportService  = game:GetService("TeleportService")
local VirtualUser      = game:GetService("VirtualUser")
local HttpService      = game:GetService("HttpService")

local Camera           = Workspace.CurrentCamera

-- ══════════════════════════════════════════════════════════════════════════════
-- [[ PAYOMBOYZ - SECRET HUB UI LIBRARY (RED NEON OBSIDIAN GLASS THEME) ]]
-- ══════════════════════════════════════════════════════════════════════════════

local Library = {
    Tabs = {},
    Elements = {},
    CurrentTab = "หน้าหลัก",
    Config = {
        ToggleKey = Enum.KeyCode.RightControl
    },
    Theme = {
        Background          = Color3.fromRGB(11, 11, 14),
        CardBackground      = Color3.fromRGB(16, 16, 21),
        CardBackgroundHover = Color3.fromRGB(24, 24, 32),
        Border              = Color3.fromRGB(245, 28, 48),
        BorderSubtle        = Color3.fromRGB(68, 15, 22),
        AccentRed           = Color3.fromRGB(245, 28, 48),
        AccentBrightRed     = Color3.fromRGB(255, 45, 65),
        AccentGlow          = Color3.fromRGB(255, 60, 80),
        TextWhite           = Color3.fromRGB(255, 255, 255),
        TextLight           = Color3.fromRGB(235, 235, 242),
        TextMuted           = Color3.fromRGB(165, 165, 180),
        TextCyan            = Color3.fromRGB(140, 215, 245),
        StatusGreen         = Color3.fromRGB(0, 230, 118)
    }
}

local function Tween(instance, properties, duration, style, direction)
    local tweenInfo = TweenInfo.new(
        duration or 0.22,
        style or Enum.EasingStyle.Quart,
        direction or Enum.EasingDirection.Out
    )
    local tween = TweenService:Create(instance, tweenInfo, properties)
    tween:Play()
    return tween
end

local function playClickSound()
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://6895079853"
        sound.Volume = 0.3
        sound.Parent = SoundService
        sound:Play()
        sound.Ended:Connect(function() sound:Destroy() end)
    end)
end

local customAvatarAsset = nil
local function loadCustomAvatarImage()
    if customAvatarAsset then return customAvatarAsset end
    local avatarUrl = "https://raw.githubusercontent.com/aslamdunk7/paypmboygang/main/543199739_2812856088914181_3062917809445648175_n.jpg"
    local fileName = "payomboyz_avatar.jpg"
    
    pcall(function()
        if typeof(writefile) == "function" and (typeof(getcustomasset) == "function" or typeof(getsynasset) == "function") then
            local getAsset = getcustomasset or getsynasset
            if typeof(isfile) == "function" and not isfile(fileName) then
                local imageBytes = game:HttpGet(avatarUrl)
                if imageBytes and #imageBytes > 0 then writefile(fileName, imageBytes) end
            end
            if typeof(isfile) == "function" and isfile(fileName) then
                customAvatarAsset = getAsset(fileName)
            end
        end
    end)

    if not customAvatarAsset then
        customAvatarAsset = "rbxthumb://type=AvatarHeadShot&id=" .. LocalPlayer.UserId .. "&w=150&h=150"
    end
    return customAvatarAsset
end

local function MakeDraggable(dragHandle, targetFrame)
    local dragging = false
    local dragInput, mousePos, framePos

    dragHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            mousePos = input.Position
            framePos = targetFrame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)

    dragHandle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local scale = (uiScale and uiScale.Scale) or 1
            local delta = (input.Position - mousePos) / scale
            targetFrame.Position = UDim2.new(
                framePos.X.Scale,
                framePos.X.Offset + delta.X,
                framePos.Y.Scale,
                framePos.Y.Offset + delta.Y
            )
        end
    end)
end

-- Parent ScreenGui creation
local TargetParent = (typeof(gethui) == "function") and gethui() or CoreGui
local oldGui = TargetParent:FindFirstChild("PayomboyZ_Hub")
if oldGui then pcall(function() oldGui:Destroy() end) end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PayomboyZ_Hub"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 99999
pcall(function() ScreenGui.Parent = TargetParent end)
if not ScreenGui.Parent then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui", 3) end

-- Responsive UI Scale
local uiScale = Instance.new("UIScale")
local function updateScale()
    if Camera and Camera.ViewportSize then
        local vp = Camera.ViewportSize
        local scaleX = (vp.X - 24) / 1024
        local scaleY = (vp.Y - 24) / 682
        uiScale.Scale = math.clamp(math.min(scaleX, scaleY), 0.45, 1.0)
    end
end
updateScale()
Camera:GetPropertyChangedSignal("ViewportSize"):Connect(function() pcall(updateScale) end)
uiScale.Parent = ScreenGui

-- Top Notification Toast System
function Library:Notify(config)
    config = config or {}
    local title = config.Title or "PayomboyZ Hub"
    local message = config.Content or config.Message or ""
    local duration = config.Duration or 3.2
    local notifType = config.Type or "info"

    local parentFrame = ScreenGui:FindFirstChild("MainFrame") or ScreenGui
    local toast = Instance.new("Frame")
    toast.Size = UDim2.new(0, 390, 0, 58)
    toast.Position = UDim2.new(0.5, -195, 0, -80)
    toast.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
    toast.BorderSizePixel = 0
    toast.ZIndex = 500
    toast.Parent = parentFrame

    local toastCorner = Instance.new("UICorner")
    toastCorner.CornerRadius = UDim.new(0, 12)
    toastCorner.Parent = toast

    local toastStroke = Instance.new("UIStroke")
    toastStroke.Thickness = 1.5
    toastStroke.Color = (notifType == "success" and Library.Theme.StatusGreen) or (notifType == "error" and Color3.fromRGB(255, 40, 55)) or Library.Theme.Border
    toastStroke.Parent = toast

    local icon = Instance.new("ImageLabel")
    icon.Position = UDim2.new(0, 14, 0.5, -11)
    icon.Size = UDim2.new(0, 22, 0, 22)
    icon.BackgroundTransparency = 1
    icon.Image = notifType == "success" and "rbxassetid://10709790298" or (notifType == "error" and "rbxassetid://10709790537" or "rbxassetid://10709790948")
    icon.ImageColor3 = toastStroke.Color
    icon.ZIndex = 501
    icon.Parent = toast

    local titleLbl = Instance.new("TextLabel")
    titleLbl.Position = UDim2.new(0, 46, 0, 8)
    titleLbl.Size = UDim2.new(1, -56, 0, 20)
    titleLbl.BackgroundTransparency = 1
    titleLbl.Font = Enum.Font.GothamBold
    titleLbl.Text = title
    titleLbl.TextColor3 = Library.Theme.TextWhite
    titleLbl.TextSize = 13
    titleLbl.TextXAlignment = Enum.TextXAlignment.Left
    titleLbl.ZIndex = 501
    titleLbl.Parent = toast

    local msgLbl = Instance.new("TextLabel")
    msgLbl.Position = UDim2.new(0, 46, 0, 28)
    msgLbl.Size = UDim2.new(1, -56, 0, 20)
    msgLbl.BackgroundTransparency = 1
    msgLbl.Font = Enum.Font.GothamMedium
    msgLbl.Text = message
    msgLbl.TextColor3 = Library.Theme.TextMuted
    msgLbl.TextSize = 11
    msgLbl.TextXAlignment = Enum.TextXAlignment.Left
    msgLbl.ZIndex = 501
    msgLbl.Parent = toast

    playClickSound()
    Tween(toast, { Position = UDim2.new(0.5, -195, 0, 18) }, 0.28, Enum.EasingStyle.Back)

    task.delay(duration, function()
        if toast and toast.Parent then
            Tween(toast, { Position = UDim2.new(0.5, -195, 0, -80) }, 0.22)
            task.wait(0.22)
            toast:Destroy()
        end
    end)
end

local function Notify(title, content, kind, dur)
    Library:Notify({ Title = title, Content = content, Type = kind or "info", Duration = dur or 3 })
end

-- ==============================================================================
-- [[ MAIN WINDOW FRAMEWORK ]]
-- ==============================================================================

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.fromOffset(1024, 682)
MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
MainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
MainFrame.BackgroundColor3 = Library.Theme.Background
MainFrame.BorderSizePixel = 0
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 18)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Name = "MainStroke"
MainStroke.Thickness = 1.5
MainStroke.Color = Library.Theme.Border
MainStroke.Parent = MainFrame

-- Floating Cyber Snow / Embers
local snowLayer = Instance.new("Frame")
snowLayer.Name = "SnowLayer"
snowLayer.Size = UDim2.fromScale(1, 1)
snowLayer.BackgroundTransparency = 1
snowLayer.ZIndex = 2
snowLayer.Parent = MainFrame

task.spawn(function()
    local dots = {}
    for i = 1, 35 do
        local dot = Instance.new("Frame")
        dot.Size = UDim2.fromOffset(math.random(2, 4), math.random(2, 4))
        dot.Position = UDim2.new(math.random(), 0, math.random(), 0)
        dot.BackgroundColor3 = (i % 3 == 0) and Color3.fromRGB(255, 60, 80) or Color3.fromRGB(220, 240, 255)
        dot.BackgroundTransparency = math.random(35, 75) / 100
        dot.BorderSizePixel = 0
        dot.Parent = snowLayer

        local dCorner = Instance.new("UICorner")
        dCorner.CornerRadius = UDim.new(1, 0)
        dCorner.Parent = dot

        dots[#dots + 1] = {
            frame = dot,
            speed = math.random(15, 40) / 10000,
            drift = math.random(-10, 10) / 10000,
            pos = dot.Position.Y.Scale
        }
    end

    while task.wait(0.03) do
        if not ScreenGui or not ScreenGui.Parent or not MainFrame or not MainFrame.Parent then break end
        for _, data in ipairs(dots) do
            data.pos = data.pos + data.speed
            if data.pos > 1.05 then data.pos = -0.05 end
            local newX = (data.frame.Position.X.Scale + data.drift) % 1.0
            data.frame.Position = UDim2.new(newX, 0, data.pos, 0)
        end
    end
end)

-- Floating Toggle Capsule
local toggleCapsule = Instance.new("Frame")
toggleCapsule.Name = "PayomboyZToggleCapsule"
toggleCapsule.Size = UDim2.fromOffset(230, 58)
toggleCapsule.Position = UDim2.new(0, 15, 0.5, -29)
toggleCapsule.BackgroundColor3 = Library.Theme.CardBackground
toggleCapsule.BackgroundTransparency = 0.15
toggleCapsule.BorderSizePixel = 0
toggleCapsule.ClipsDescendants = true
toggleCapsule.ZIndex = 99999
toggleCapsule.Parent = ScreenGui

local tcCorner = Instance.new("UICorner")
tcCorner.CornerRadius = UDim.new(0, 16)
tcCorner.Parent = toggleCapsule

local tcStroke = Instance.new("UIStroke")
tcStroke.Color = Library.Theme.Border
tcStroke.Thickness = 1.5
tcStroke.Parent = toggleCapsule

local capAvatarFrame = Instance.new("Frame")
capAvatarFrame.Size = UDim2.fromOffset(42, 42)
capAvatarFrame.Position = UDim2.new(0, 8, 0.5, -21)
capAvatarFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
capAvatarFrame.BorderSizePixel = 0
capAvatarFrame.ZIndex = 100000
capAvatarFrame.Parent = toggleCapsule

local caCorner = Instance.new("UICorner")
caCorner.CornerRadius = UDim.new(1, 0)
caCorner.Parent = capAvatarFrame

local capAvatarImg = Instance.new("ImageLabel")
capAvatarImg.Size = UDim2.fromScale(1, 1)
capAvatarImg.BackgroundTransparency = 1
capAvatarImg.Image = loadCustomAvatarImage()
capAvatarImg.ZIndex = 100001
capAvatarImg.Parent = capAvatarFrame

local cImgCorner = Instance.new("UICorner")
cImgCorner.CornerRadius = UDim.new(1, 0)
cImgCorner.Parent = capAvatarImg

local capTitle = Instance.new("TextLabel")
capTitle.Size = UDim2.new(0, 115, 0, 18)
capTitle.Position = UDim2.new(0, 56, 0, 10)
capTitle.BackgroundTransparency = 1
capTitle.Text = "PAYOMBOYZ"
capTitle.TextColor3 = Library.Theme.TextWhite
capTitle.Font = Enum.Font.GothamBold
capTitle.TextSize = 13
capTitle.TextXAlignment = Enum.TextXAlignment.Left
capTitle.ZIndex = 100000
capTitle.Parent = toggleCapsule

local capSub = Instance.new("TextLabel")
capSub.Size = UDim2.new(0, 115, 0, 14)
capSub.Position = UDim2.new(0, 56, 0, 28)
capSub.BackgroundTransparency = 1
capSub.Text = "MM2 Master Suite"
capSub.TextColor3 = Library.Theme.AccentBrightRed
capSub.Font = Enum.Font.Gotham
capSub.TextSize = 10
capSub.TextXAlignment = Enum.TextXAlignment.Left
capSub.ZIndex = 100000
capSub.Parent = toggleCapsule

local capBtn = Instance.new("TextButton")
capBtn.Size = UDim2.fromOffset(34, 34)
capBtn.Position = UDim2.new(1, -42, 0.5, -17)
capBtn.BackgroundColor3 = Color3.fromRGB(28, 12, 18)
capBtn.Text = "[V]"
capBtn.TextColor3 = Library.Theme.AccentBrightRed
capBtn.Font = Enum.Font.GothamBold
capBtn.TextSize = 15
capBtn.ZIndex = 100001
capBtn.Parent = toggleCapsule

local cbCorner = Instance.new("UICorner")
cbCorner.CornerRadius = UDim.new(0, 8)
cbCorner.Parent = capBtn

capBtn.MouseButton1Click:Connect(function()
    playClickSound()
    MainFrame.Visible = not MainFrame.Visible
end)
MakeDraggable(toggleCapsule, toggleCapsule)

-- Top Drag Handle across Top Header of Main Frame
local TopDragHandle = Instance.new("Frame")
TopDragHandle.Name = "TopDragHandle"
TopDragHandle.Size = UDim2.new(1, -180, 0, 70)
TopDragHandle.BackgroundTransparency = 1
TopDragHandle.ZIndex = 10
TopDragHandle.Parent = MainFrame
MakeDraggable(TopDragHandle, MainFrame)

-- Left Sidebar
local Sidebar = Instance.new("Frame")
Sidebar.Name = "Sidebar"
Sidebar.Size = UDim2.new(0, 214, 1, 0)
Sidebar.BackgroundColor3 = Library.Theme.CardBackground
Sidebar.BackgroundTransparency = 0.10
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 5
Sidebar.Parent = MainFrame

local sbCorner = Instance.new("UICorner")
sbCorner.CornerRadius = UDim.new(0, 18)
sbCorner.Parent = Sidebar

local sbLine = Instance.new("Frame")
sbLine.Position = UDim2.new(1, -1, 0, 0)
sbLine.Size = UDim2.new(0, 1, 1, 0)
sbLine.BackgroundColor3 = Library.Theme.BorderSubtle
sbLine.BorderSizePixel = 0
sbLine.ZIndex = 6
sbLine.Parent = Sidebar

-- Brand Logo & User Profile Header
local avatarFrame = Instance.new("Frame")
avatarFrame.Size = UDim2.fromOffset(46, 46)
avatarFrame.Position = UDim2.new(0, 14, 0, 14)
avatarFrame.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
avatarFrame.BorderSizePixel = 0
avatarFrame.ZIndex = 10
avatarFrame.Parent = Sidebar

local aCorner = Instance.new("UICorner")
aCorner.CornerRadius = UDim.new(1, 0)
aCorner.Parent = avatarFrame

local avStroke = Instance.new("UIStroke")
avStroke.Color = Library.Theme.Border
avStroke.Thickness = 1.5
avStroke.Parent = avatarFrame

local avatarImg = Instance.new("ImageLabel")
avatarImg.Size = UDim2.fromScale(1, 1)
avatarImg.BackgroundTransparency = 1
avatarImg.Image = loadCustomAvatarImage()
avatarImg.ZIndex = 11
avatarImg.Parent = avatarFrame

local avImgCorner = Instance.new("UICorner")
avImgCorner.CornerRadius = UDim.new(1, 0)
avImgCorner.Parent = avatarImg

local onlineDot = Instance.new("Frame")
onlineDot.Size = UDim2.fromOffset(10, 10)
onlineDot.Position = UDim2.new(1, -8, 1, -8)
onlineDot.BackgroundColor3 = Library.Theme.StatusGreen
onlineDot.BorderSizePixel = 0
onlineDot.ZIndex = 12
onlineDot.Parent = avatarFrame

local onlineCorner = Instance.new("UICorner")
onlineCorner.CornerRadius = UDim.new(1, 0)
onlineCorner.Parent = onlineDot

local brandTitle = Instance.new("TextLabel")
brandTitle.Size = UDim2.new(1, -72, 0, 18)
brandTitle.Position = UDim2.new(0, 68, 0, 16)
brandTitle.BackgroundTransparency = 1
brandTitle.Text = "PAYOMBOYZ"
brandTitle.TextColor3 = Library.Theme.TextWhite
brandTitle.Font = Enum.Font.GothamBold
brandTitle.TextSize = 14
brandTitle.TextXAlignment = Enum.TextXAlignment.Left
brandTitle.ZIndex = 10
brandTitle.Parent = Sidebar

local brandSubtitle = Instance.new("TextLabel")
brandSubtitle.Size = UDim2.new(1, -72, 0, 14)
brandSubtitle.Position = UDim2.new(0, 68, 0, 36)
brandSubtitle.BackgroundTransparency = 1
brandSubtitle.Text = "SECRET HUB • MM2"
brandSubtitle.TextColor3 = Library.Theme.AccentBrightRed
brandSubtitle.Font = Enum.Font.GothamBold
brandSubtitle.TextSize = 10
brandSubtitle.TextXAlignment = Enum.TextXAlignment.Left
brandSubtitle.ZIndex = 10
brandSubtitle.Parent = Sidebar

-- Live Metric Box
local metricBox = Instance.new("Frame")
metricBox.Size = UDim2.new(1, -28, 0, 26)
metricBox.Position = UDim2.new(0, 14, 0, 70)
metricBox.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
metricBox.BorderSizePixel = 0
metricBox.ZIndex = 10
metricBox.Parent = Sidebar

local mbCorner = Instance.new("UICorner")
mbCorner.CornerRadius = UDim.new(0, 6)
mbCorner.Parent = metricBox

local mbStroke = Instance.new("UIStroke")
mbStroke.Color = Library.Theme.BorderSubtle
mbStroke.Thickness = 1
mbStroke.Parent = metricBox

local metricLabel = Instance.new("TextLabel")
metricLabel.Size = UDim2.fromScale(1, 1)
metricLabel.BackgroundTransparency = 1
metricLabel.Text = "TIME: 00:00  •  PING: 0 ms"
metricLabel.TextColor3 = Library.Theme.TextCyan
metricLabel.Font = Enum.Font.GothamBold
metricLabel.TextSize = 10
metricLabel.ZIndex = 11
metricLabel.Parent = metricBox

task.spawn(function()
    local startTime = os.time()
    while task.wait(1) do
        if not ScreenGui or not ScreenGui.Parent then break end
        local elapsed = os.time() - startTime
        local mins = math.floor(elapsed / 60)
        local secs = elapsed % 60
        local ping = 0
        pcall(function() ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        metricLabel.Text = string.format("TIME: %02d:%02d  •  PING: %d ms", mins, secs, ping)
    end
end)

-- Sidebar Navigation Tabs Scroller
local tabScroll = Instance.new("ScrollingFrame")
tabScroll.Name = "SidebarTabScroll"
tabScroll.Size = UDim2.new(1, -16, 1, -170)
tabScroll.Position = UDim2.new(0, 8, 0, 106)
tabScroll.BackgroundTransparency = 1
tabScroll.ScrollBarThickness = 3
tabScroll.ScrollBarImageColor3 = Library.Theme.Border
tabScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
tabScroll.ZIndex = 10
tabScroll.Parent = Sidebar

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Vertical
tabLayout.Padding = UDim.new(0, 6)
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.Parent = tabScroll

pcall(function() tabScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y end)

-- Sidebar Bottom Control Bar
local botBar = Instance.new("Frame")
botBar.Size = UDim2.new(1, -20, 0, 50)
botBar.Position = UDim2.new(0, 10, 1, -58)
botBar.BackgroundColor3 = Color3.fromRGB(14, 14, 18)
botBar.BorderSizePixel = 0
botBar.ZIndex = 10
botBar.Parent = Sidebar

local bbCorner = Instance.new("UICorner")
bbCorner.CornerRadius = UDim.new(0, 8)
bbCorner.Parent = botBar

local logoutBtn = Instance.new("TextButton")
logoutBtn.Size = UDim2.new(0.5, -6, 0, 34)
logoutBtn.Position = UDim2.new(0, 4, 0.5, -17)
logoutBtn.BackgroundColor3 = Color3.fromRGB(34, 14, 20)
logoutBtn.Text = "Log out"
logoutBtn.TextColor3 = Color3.fromRGB(255, 60, 60)
logoutBtn.Font = Enum.Font.GothamBold
logoutBtn.TextSize = 11
logoutBtn.ZIndex = 12
logoutBtn.Parent = botBar

local loCorner = Instance.new("UICorner")
loCorner.CornerRadius = UDim.new(0, 6)
loCorner.Parent = logoutBtn

logoutBtn.MouseButton1Click:Connect(function()
    playClickSound()
    if _G.MM2Cleanup then pcall(_G.MM2Cleanup) end
    if ScreenGui then ScreenGui:Destroy() end
    pcall(function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/aslamdunk7/paypmboygang/refs/heads/main/Start"))()
    end)
end)

local discordBtn = Instance.new("TextButton")
discordBtn.Size = UDim2.new(0.5, -6, 0, 34)
discordBtn.Position = UDim2.new(0.5, 2, 0.5, -17)
discordBtn.BackgroundColor3 = Color3.fromRGB(20, 24, 38)
discordBtn.Text = "Discord"
discordBtn.TextColor3 = Library.Theme.TextCyan
discordBtn.Font = Enum.Font.GothamBold
discordBtn.TextSize = 11
discordBtn.ZIndex = 12
discordBtn.Parent = botBar

local dcCorner = Instance.new("UICorner")
dcCorner.CornerRadius = UDim.new(0, 6)
dcCorner.Parent = discordBtn

discordBtn.MouseButton1Click:Connect(function()
    playClickSound()
    pcall(function() (setclipboard or toclipboard or writeclipboard)("https://discord.gg/payomboyz") end)
    Notify("Discord", "คัดลอกลิงก์ Discord เข้าคลิปบอร์ดแล้ว!", "success")
end)

-- SubHeader Bar (Right Top Header)
local SubHeader = Instance.new("Frame")
SubHeader.Name = "SubHeader"
SubHeader.Position = UDim2.new(0, 226, 0, 14)
SubHeader.Size = UDim2.new(1, -242, 0, 56)
SubHeader.BackgroundColor3 = Library.Theme.CardBackground
SubHeader.BorderSizePixel = 0
SubHeader.ZIndex = 8
SubHeader.Parent = MainFrame

local shCorner = Instance.new("UICorner")
shCorner.CornerRadius = UDim.new(0, 12)
shCorner.Parent = SubHeader

local shStroke = Instance.new("UIStroke")
shStroke.Color = Library.Theme.BorderSubtle
shStroke.Thickness = 1
shStroke.Parent = SubHeader

local SubHeaderTitle = Instance.new("TextLabel")
SubHeaderTitle.Name = "SubHeaderTitle"
SubHeaderTitle.Position = UDim2.new(0, 18, 0, 8)
SubHeaderTitle.Size = UDim2.new(0, 300, 0, 22)
SubHeaderTitle.BackgroundTransparency = 1
SubHeaderTitle.Font = Enum.Font.GothamBold
SubHeaderTitle.Text = "หน้าหลัก"
SubHeaderTitle.TextColor3 = Library.Theme.TextWhite
SubHeaderTitle.TextSize = 16
SubHeaderTitle.TextXAlignment = Enum.TextXAlignment.Left
SubHeaderTitle.ZIndex = 9
SubHeaderTitle.Parent = SubHeader

local SubHeaderSubtitle = Instance.new("TextLabel")
SubHeaderSubtitle.Name = "SubHeaderSubtitle"
SubHeaderSubtitle.Position = UDim2.new(0, 18, 0, 30)
SubHeaderSubtitle.Size = UDim2.new(0, 380, 0, 16)
SubHeaderSubtitle.BackgroundTransparency = 1
SubHeaderSubtitle.Font = Enum.Font.GothamMedium
SubHeaderSubtitle.Text = "ระบบภาพรวมและแดชบอร์ดเกม Murder Mystery 2"
SubHeaderSubtitle.TextColor3 = Library.Theme.TextCyan
SubHeaderSubtitle.TextSize = 11
SubHeaderSubtitle.TextXAlignment = Enum.TextXAlignment.Left
SubHeaderSubtitle.ZIndex = 9
SubHeaderSubtitle.Parent = SubHeader

-- Window Controls (Minimize & Close & Home)
local BackHomeBtn = Instance.new("TextButton")
BackHomeBtn.Position = UDim2.new(1, -165, 0.5, -16)
BackHomeBtn.Size = UDim2.new(0, 85, 0, 32)
BackHomeBtn.BackgroundColor3 = Color3.fromRGB(24, 24, 32)
BackHomeBtn.Text = "[<] หน้าหลัก"
BackHomeBtn.TextColor3 = Library.Theme.TextWhite
BackHomeBtn.Font = Enum.Font.GothamBold
BackHomeBtn.TextSize = 11
BackHomeBtn.ZIndex = 9
BackHomeBtn.Parent = SubHeader

local bhCorner = Instance.new("UICorner")
bhCorner.CornerRadius = UDim.new(0, 8)
bhCorner.Parent = BackHomeBtn

local MinBtn = Instance.new("TextButton")
MinBtn.Name = "MinBtn"
MinBtn.Position = UDim2.new(1, -72, 0.5, -16)
MinBtn.Size = UDim2.new(0, 32, 0, 32)
MinBtn.BackgroundColor3 = Color3.fromRGB(24, 24, 32)
MinBtn.Text = "─"
MinBtn.TextColor3 = Library.Theme.TextMuted
MinBtn.Font = Enum.Font.GothamBold
MinBtn.TextSize = 14
MinBtn.ZIndex = 9
MinBtn.Parent = SubHeader

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 8)
minCorner.Parent = MinBtn

local CloseBtn = Instance.new("TextButton")
CloseBtn.Name = "CloseBtn"
CloseBtn.Position = UDim2.new(1, -36, 0.5, -16)
CloseBtn.Size = UDim2.new(0, 32, 0, 32)
CloseBtn.BackgroundColor3 = Color3.fromRGB(24, 24, 32)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Library.Theme.TextMuted
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 13
CloseBtn.ZIndex = 9
CloseBtn.Parent = SubHeader

local clsCorner = Instance.new("UICorner")
clsCorner.CornerRadius = UDim.new(0, 8)
clsCorner.Parent = CloseBtn

local isMinimized = false
MinBtn.MouseButton1Click:Connect(function()
    playClickSound()
    isMinimized = not isMinimized
    if isMinimized then
        Tween(MainFrame, { Size = UDim2.fromOffset(1024, 84) }, 0.28)
    else
        Tween(MainFrame, { Size = UDim2.fromOffset(1024, 682) }, 0.28)
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    playClickSound()
    Tween(MainFrame, { Size = UDim2.fromOffset(1024, 0) }, 0.22)
    task.delay(0.22, function() MainFrame.Visible = false end)
end)

-- Hotkey to Toggle GUI
UserInputService.InputBegan:Connect(function(input, gpe)
    if not gpe and (input.KeyCode == Library.Config.ToggleKey or input.KeyCode == Enum.KeyCode.K) then
        playClickSound()
        MainFrame.Visible = not MainFrame.Visible
    elseif not gpe and input.KeyCode == Enum.KeyCode.F then
        playClickSound()
        uiScale.Scale = (uiScale.Scale == 1.0) and 0.85 or 1.0
    end
end)

-- Main Page Scroll Container
local PagesContainer = Instance.new("Frame")
PagesContainer.Name = "PagesContainer"
PagesContainer.Position = UDim2.new(0, 226, 0, 78)
PagesContainer.Size = UDim2.new(1, -242, 1, -90)
PagesContainer.BackgroundTransparency = 1
PagesContainer.ZIndex = 8
PagesContainer.Parent = MainFrame

-- Tab Switching Engine
local tabRegistry = {}
local activeTabIndicator = nil

local function SelectTab(tabName)
    playClickSound()
    Library.CurrentTab = tabName
    SubHeaderTitle.Text = tabName

    local tabData = tabRegistry[tabName]
    if tabData and tabData.Subtitle then
        SubHeaderSubtitle.Text = tabData.Subtitle
    end

    for name, data in pairs(tabRegistry) do
        local isCurrent = (name == tabName)
        if data.PageFrame then data.PageFrame.Visible = isCurrent end
        if data.Button then
            Tween(data.Button, {
                BackgroundColor3 = isCurrent and Color3.fromRGB(34, 14, 20) or Color3.fromRGB(16, 16, 21),
                BackgroundTransparency = isCurrent and 0.1 or 0.4
            }, 0.2)
            if data.Indicator then data.Indicator.Visible = isCurrent end
            if data.Label then
                data.Label.TextColor3 = isCurrent and Library.Theme.TextWhite or Library.Theme.TextMuted
            end
            if data.Stroke then
                data.Stroke.Color = isCurrent and Library.Theme.Border or Library.Theme.BorderSubtle
            end
            if data.IconObj then
                if data.IconObj:IsA("ImageLabel") then
                    Tween(data.IconObj, { ImageColor3 = isCurrent and Library.Theme.AccentBrightRed or Library.Theme.TextMuted }, 0.2)
                elseif data.IconObj:IsA("TextLabel") then
                    Tween(data.IconObj, { TextColor3 = isCurrent and Library.Theme.AccentBrightRed or Library.Theme.TextMuted }, 0.2)
                end
            end
        end
    end
end

BackHomeBtn.MouseButton1Click:Connect(function() SelectTab("หน้าหลัก")

-- Auto-Load Config on startup
task.spawn(function()
    task.wait(0.3)
    if _CFG.AutoLoadConfig then
        pcall(function()
            local defaultPath = getSavePath("Default")
            local hasDef = false
            pcall(function() if typeof(isfile) == "function" and isfile(defaultPath) then hasDef = true end end)
            if not hasDef then
                local fallbackPath = "PayomboyZ_MM2_Default.json"
                pcall(function() if typeof(isfile) == "function" and isfile(fallbackPath) then hasDef = true end end)
            end
            if hasDef then
                LoadConfig("Default")
            end
        end)
    end
end)

-- Auto Save Config Thread (Every 90s)
task.spawn(function()
    while not HUB.dead do
        task.wait(90)
        if _CFG.AutoSaveConfig then
            pcall(function()
                SaveConfig(_CFG.SelectedConfigSlot or "Default")
            end)
        end
    end
end)

 end)

-- Tab Builder Object
function Library:CreateTab(tabCfg)
    local tabName = tabCfg.Name or "Tab"
    local tabIcon = tabCfg.Icon or "rbxassetid://10723407389"
    local tabSubtitle = tabCfg.Subtitle or "เมนูระบบ"

    -- Sidebar Tab Button
    local tabBtn = Instance.new("TextButton")
    tabBtn.Name = "TabBtn_" .. tabName
    tabBtn.Size = UDim2.new(1, -6, 0, 42)
    tabBtn.BackgroundColor3 = Color3.fromRGB(16, 16, 21)
    tabBtn.BackgroundTransparency = 0.4
    tabBtn.Text = ""
    tabBtn.AutoButtonColor = false
    tabBtn.ZIndex = 11
    tabBtn.Parent = tabScroll

    local tbCorner = Instance.new("UICorner")
    tbCorner.CornerRadius = UDim.new(0, 8)
    tbCorner.Parent = tabBtn

    local tbStroke = Instance.new("UIStroke")
    tbStroke.Color = Library.Theme.BorderSubtle
    tbStroke.Thickness = 1
    tbStroke.Parent = tabBtn

    local tabInd = Instance.new("Frame")
    tabInd.Size = UDim2.new(0, 3, 0, 22)
    tabInd.Position = UDim2.new(0, 4, 0.5, -11)
    tabInd.BackgroundColor3 = Library.Theme.AccentRed
    tabInd.BorderSizePixel = 0
    tabInd.Visible = false
    tabInd.ZIndex = 12
    tabInd.Parent = tabBtn

    local tiCorner = Instance.new("UICorner")
    tiCorner.CornerRadius = UDim.new(1, 0)
    tiCorner.Parent = tabInd

    local tabIconObj = nil
    if type(tabIcon) == "string" and tabIcon:find("rbxassetid") then
        local tabIconImg = Instance.new("ImageLabel")
        tabIconImg.Size = UDim2.fromOffset(18, 18)
        tabIconImg.Position = UDim2.new(0, 12, 0.5, -9)
        tabIconImg.BackgroundTransparency = 1
        tabIconImg.Image = tabIcon
        tabIconImg.ImageColor3 = Library.Theme.TextMuted
        tabIconImg.ZIndex = 12
        tabIconImg.Parent = tabBtn
        tabIconObj = tabIconImg
    else
        local tabIconLbl = Instance.new("TextLabel")
        tabIconLbl.Size = UDim2.new(0, 26, 1, 0)
        tabIconLbl.Position = UDim2.new(0, 10, 0, 0)
        tabIconLbl.BackgroundTransparency = 1
        tabIconLbl.Text = tostring(tabIcon)
        tabIconLbl.TextColor3 = Library.Theme.TextMuted
        tabIconLbl.Font = Enum.Font.GothamBold
        tabIconLbl.TextSize = 13
        tabIconLbl.ZIndex = 12
        tabIconLbl.Parent = tabBtn
        tabIconObj = tabIconLbl
    end

    local tabTextLbl = Instance.new("TextLabel")
    tabTextLbl.Size = UDim2.new(1, -44, 1, 0)
    tabTextLbl.Position = UDim2.new(0, 38, 0, 0)
    tabTextLbl.BackgroundTransparency = 1
    tabTextLbl.Text = tabName
    tabTextLbl.TextColor3 = Library.Theme.TextMuted
    tabTextLbl.Font = Enum.Font.GothamBold
    tabTextLbl.TextSize = 12
    tabTextLbl.TextXAlignment = Enum.TextXAlignment.Left
    tabTextLbl.ZIndex = 12
    tabTextLbl.Parent = tabBtn

    -- Page Scrolling Frame
    local pageScroll = Instance.new("ScrollingFrame")
    pageScroll.Name = "Page_" .. tabName
    pageScroll.Size = UDim2.fromScale(1, 1)
    pageScroll.BackgroundTransparency = 1
    pageScroll.BorderSizePixel = 0
    pageScroll.ScrollBarThickness = 4
    pageScroll.ScrollBarImageColor3 = Library.Theme.Border
    pageScroll.Visible = false
    pageScroll.ZIndex = 10
    pageScroll.Parent = PagesContainer

    local pageLayout = Instance.new("UIListLayout")
    pageLayout.Padding = UDim.new(0, 10)
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Parent = pageScroll

    local pagePadding = Instance.new("UIPadding")
    pagePadding.PaddingTop = UDim.new(0, 4)
    pagePadding.PaddingBottom = UDim.new(0, 20)
    pagePadding.PaddingRight = UDim.new(0, 10)
    pagePadding.PaddingLeft = UDim.new(0, 2)
    pagePadding.Parent = pageScroll

    pcall(function() pageScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y end)

    tabRegistry[tabName] = {
        Button = tabBtn,
        Indicator = tabInd,
        Label = tabTextLbl,
        Stroke = tbStroke,
        IconObj = tabIconObj,
        PageFrame = pageScroll,
        Subtitle = tabSubtitle
    }

    tabBtn.MouseButton1Click:Connect(function() SelectTab(tabName) end)

    local TabObj = { PageFrame = pageScroll }

    -- 1. Component: AddSection
    function TabObj:AddSection(title)
        local secFrame = Instance.new("Frame")
        secFrame.Size = UDim2.new(1, 0, 0, 30)
        secFrame.BackgroundTransparency = 1
        secFrame.ZIndex = 10
        secFrame.Parent = pageScroll

        local bar = Instance.new("Frame")
        bar.Position = UDim2.new(0, 2, 0.5, -7)
        bar.Size = UDim2.new(0, 3, 0, 14)
        bar.BackgroundColor3 = Library.Theme.AccentRed
        bar.BorderSizePixel = 0
        bar.Parent = secFrame

        local barCorner = Instance.new("UICorner")
        barCorner.CornerRadius = UDim.new(1, 0)
        barCorner.Parent = bar

        local secLbl = Instance.new("TextLabel")
        secLbl.Position = UDim2.new(0, 14, 0, 0)
        secLbl.Size = UDim2.new(1, -20, 1, 0)
        secLbl.BackgroundTransparency = 1
        secLbl.Font = Enum.Font.GothamBold
        secLbl.Text = string.upper(title)
        secLbl.TextColor3 = Library.Theme.AccentBrightRed
        secLbl.TextSize = 12
        secLbl.TextXAlignment = Enum.TextXAlignment.Left
        secLbl.Parent = secFrame
        return secFrame
    end

    -- 2. Component: AddToggle
    function TabObj:AddToggle(arg1, arg2)
    local togConfig = (type(arg1) == "table" and arg1) or (type(arg2) == "table" and arg2) or {}
    local id = (type(arg1) == "string" and arg1) or togConfig.Name or togConfig.Title or "Toggle"
        local tName = togConfig.Name or togConfig.Title or id
        local tDesc = togConfig.Desc or togConfig.Description or ""
        local tDefault = togConfig.Default or false
        local tCallback = togConfig.Callback or function() end
        local state = tDefault

        local togFrame = Instance.new("TextButton")
        togFrame.Size = UDim2.new(1, 0, 0, #tDesc > 0 and 52 or 44)
        togFrame.BackgroundColor3 = Library.Theme.CardBackground
        togFrame.AutoButtonColor = false
        togFrame.Text = ""
        togFrame.BorderSizePixel = 0
        togFrame.ZIndex = 10
        togFrame.Parent = pageScroll

        local tc = Instance.new("UICorner")
        tc.CornerRadius = UDim.new(0, 10)
        tc.Parent = togFrame

        local ts = Instance.new("UIStroke")
        ts.Color = Library.Theme.BorderSubtle
        ts.Thickness = 1
        ts.Parent = togFrame

        local tLabel = Instance.new("TextLabel")
        tLabel.Position = UDim2.new(0, 16, 0, #tDesc > 0 and 8 or 0)
        tLabel.Size = UDim2.new(1, -85, 0, #tDesc > 0 and 18 or 44)
        tLabel.BackgroundTransparency = 1
        tLabel.Font = Enum.Font.GothamBold
        tLabel.Text = tName
        tLabel.TextColor3 = Library.Theme.TextWhite
        tLabel.TextSize = 13
        tLabel.TextXAlignment = Enum.TextXAlignment.Left
        tLabel.Parent = togFrame

        if #tDesc > 0 then
            local tDescLbl = Instance.new("TextLabel")
            tDescLbl.Position = UDim2.new(0, 16, 0, 28)
            tDescLbl.Size = UDim2.new(1, -85, 0, 16)
            tDescLbl.BackgroundTransparency = 1
            tDescLbl.Font = Enum.Font.GothamMedium
            tDescLbl.Text = tDesc
            tDescLbl.TextColor3 = Library.Theme.TextMuted
            tDescLbl.TextSize = 11
            tDescLbl.TextXAlignment = Enum.TextXAlignment.Left
            tDescLbl.Parent = togFrame
        end

        local trackF = Instance.new("Frame")
        trackF.AnchorPoint = Vector2.new(1, 0.5)
        trackF.Position = UDim2.new(1, -16, 0.5, 0)
        trackF.Size = UDim2.new(0, 42, 0, 22)
        trackF.BackgroundColor3 = state and Library.Theme.AccentRed or Color3.fromRGB(28, 28, 36)
        trackF.BorderSizePixel = 0
        trackF.Parent = togFrame

        local trackCorner = Instance.new("UICorner")
        trackCorner.CornerRadius = UDim.new(1, 0)
        trackCorner.Parent = trackF

        local knob = Instance.new("Frame")
        knob.Position = state and UDim2.new(1, -20, 0.5, -9) or UDim2.new(0, 2, 0.5, -9)
        knob.Size = UDim2.new(0, 18, 0, 18)
        knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        knob.BorderSizePixel = 0
        knob.Parent = trackF

        local knobCorner = Instance.new("UICorner")
        knobCorner.CornerRadius = UDim.new(1, 0)
        knobCorner.Parent = knob

        local function SetState(newState)
            playClickSound()
            state = newState
            if state then
                Tween(trackF, { BackgroundColor3 = Library.Theme.AccentRed }, 0.2)
                Tween(knob, { Position = UDim2.new(1, -20, 0.5, -9) }, 0.2)
            else
                Tween(trackF, { BackgroundColor3 = Color3.fromRGB(28, 28, 36) }, 0.2)
                Tween(knob, { Position = UDim2.new(0, 2, 0.5, -9) }, 0.2)
            end
            pcall(tCallback, state)
        end

        togFrame.MouseButton1Click:Connect(function() SetState(not state) end)

        togFrame.MouseEnter:Connect(function()
            Tween(togFrame, { BackgroundColor3 = Library.Theme.CardBackgroundHover }, 0.15)
            Tween(ts, { Color = Library.Theme.Border }, 0.15)
        end)
        togFrame.MouseLeave:Connect(function()
            Tween(togFrame, { BackgroundColor3 = Library.Theme.CardBackground }, 0.15)
            Tween(ts, { Color = Library.Theme.BorderSubtle }, 0.15)
        end)

        local ret = {
            Set = SetState,
            SetValue = SetState,
            Get = function() return state end
        }
        Library.Elements[id] = ret
        return ret
    end

    -- 3. Component: AddSlider
    function TabObj:AddSlider(arg1, arg2)
    local slConfig = (type(arg1) == "table" and arg1) or (type(arg2) == "table" and arg2) or {}
    local id = (type(arg1) == "string" and arg1) or slConfig.Name or slConfig.Title or "Slider"
        local sName = slConfig.Name or slConfig.Title or id
        local sMin = slConfig.Min or 0
        local sMax = slConfig.Max or 100
        local sDefault = slConfig.Default or sMin
        local sSuffix = slConfig.Suffix or ""
        local sRounding = slConfig.Rounding or 0
        local sCallback = slConfig.Callback or function() end
        local value = sDefault

        local sFrame = Instance.new("Frame")
        sFrame.Size = UDim2.new(1, 0, 0, 56)
        sFrame.BackgroundColor3 = Library.Theme.CardBackground
        sFrame.BorderSizePixel = 0
        sFrame.ZIndex = 10
        pageScroll.CanvasSize = UDim2.new(0, 0, 0, pageLayout.AbsoluteContentSize.Y + 28)
        sFrame.Parent = pageScroll

        local sc = Instance.new("UICorner")
        sc.CornerRadius = UDim.new(0, 10)
        sc.Parent = sFrame

        local ss = Instance.new("UIStroke")
        ss.Color = Library.Theme.BorderSubtle
        ss.Thickness = 1
        ss.Parent = sFrame

        local sLabel = Instance.new("TextLabel")
        sLabel.Position = UDim2.new(0, 16, 0, 6)
        sLabel.Size = UDim2.new(1, -90, 0, 20)
        sLabel.BackgroundTransparency = 1
        sLabel.Font = Enum.Font.GothamBold
        sLabel.Text = sName
        sLabel.TextColor3 = Library.Theme.TextWhite
        sLabel.TextSize = 13
        sLabel.TextXAlignment = Enum.TextXAlignment.Left
        sLabel.Parent = sFrame

        local sValLabel = Instance.new("TextLabel")
        sValLabel.Position = UDim2.new(1, -90, 0, 6)
        sValLabel.Size = UDim2.new(0, 74, 0, 20)
        sValLabel.BackgroundTransparency = 1
        sValLabel.Font = Enum.Font.GothamBold
        sValLabel.Text = tostring(value) .. sSuffix
        sValLabel.TextColor3 = Library.Theme.AccentBrightRed
        sValLabel.TextSize = 13
        sValLabel.TextXAlignment = Enum.TextXAlignment.Right
        sValLabel.Parent = sFrame

        local barBg = Instance.new("TextButton")
        barBg.Position = UDim2.new(0, 16, 0, 34)
        barBg.Size = UDim2.new(1, -32, 0, 8)
        barBg.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
        barBg.AutoButtonColor = false
        barBg.Text = ""
        barBg.BorderSizePixel = 0
        barBg.Parent = sFrame

        local barCorner = Instance.new("UICorner")
        barCorner.CornerRadius = UDim.new(1, 0)
        barCorner.Parent = barBg

        local fill = Instance.new("Frame")
        local startPercent = math.clamp((value - sMin) / (sMax - sMin), 0, 1)
        fill.Size = UDim2.new(startPercent, 0, 1, 0)
        fill.BackgroundColor3 = Library.Theme.AccentRed
        fill.BorderSizePixel = 0
        fill.Parent = barBg

        local fillCorner = Instance.new("UICorner")
        fillCorner.CornerRadius = UDim.new(1, 0)
        fillCorner.Parent = fill

        local isSliding = false
        local function UpdateSlider(input)
            local percent = math.clamp((input.Position.X - barBg.AbsolutePosition.X) / barBg.AbsoluteSize.X, 0, 1)
            local rawVal = sMin + (sMax - sMin) * percent
            if sRounding > 0 then
                value = math.floor(rawVal * (10 ^ sRounding) + 0.5) / (10 ^ sRounding)
            else
                value = math.floor(rawVal + 0.5)
            end
            fill.Size = UDim2.new(percent, 0, 1, 0)
            sValLabel.Text = tostring(value) .. sSuffix
            pcall(sCallback, value)
        end

        barBg.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                isSliding = true
                UpdateSlider(input)
            end
        end)

        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                isSliding = false
            end
        end)

        UserInputService.InputChanged:Connect(function(input)
            if isSliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                UpdateSlider(input)
            end
        end)

        local ret = {
            Set = function(newVal)
                value = math.clamp(newVal, sMin, sMax)
                local percent = (value - sMin) / (sMax - sMin)
                fill.Size = UDim2.new(percent, 0, 1, 0)
                sValLabel.Text = tostring(value) .. sSuffix
                pcall(sCallback, value)
            end,
            SetValue = function(newVal)
                value = math.clamp(newVal, sMin, sMax)
                local percent = (value - sMin) / (sMax - sMin)
                fill.Size = UDim2.new(percent, 0, 1, 0)
                sValLabel.Text = tostring(value) .. sSuffix
                pcall(sCallback, value)
            end,
            Get = function() return value end
        }
        Library.Elements[id] = ret
        return ret
    end

    -- 4. Component: AddDropdown
    function TabObj:AddDropdown(arg1, arg2)
    local ddConfig = (type(arg1) == "table" and arg1) or (type(arg2) == "table" and arg2) or {}
    local id = (type(arg1) == "string" and arg1) or ddConfig.Name or ddConfig.Title or "Dropdown"
        local ddName = ddConfig.Name or ddConfig.Title or id
        local ddOptions = ddConfig.Options or ddConfig.Values or {}
        local ddDefault = ddConfig.Default or ddConfig.Value or ddOptions[1] or ""
        local ddCallback = ddConfig.Callback or function() end
        local isOpen = false
        local selected = ddDefault

        local ddContainer = Instance.new("Frame")
        ddContainer.Size = UDim2.new(1, 0, 0, 46)
        ddContainer.BackgroundColor3 = Library.Theme.CardBackground
        ddContainer.BorderSizePixel = 0
        ddContainer.ClipsDescendants = true
        ddContainer.ZIndex = 15
        ddContainer.Parent = pageScroll

        local ddc = Instance.new("UICorner")
        ddc.CornerRadius = UDim.new(0, 10)
        ddc.Parent = ddContainer

        local dds = Instance.new("UIStroke")
        dds.Color = Library.Theme.BorderSubtle
        dds.Thickness = 1
        dds.Parent = ddContainer

        local headerBtn = Instance.new("TextButton")
        headerBtn.Size = UDim2.new(1, 0, 0, 46)
        headerBtn.BackgroundTransparency = 1
        headerBtn.Text = ""
        headerBtn.Parent = ddContainer

        local ddLabel = Instance.new("TextLabel")
        ddLabel.Position = UDim2.new(0, 16, 0, 0)
        ddLabel.Size = UDim2.new(0.5, 0, 1, 0)
        ddLabel.BackgroundTransparency = 1
        ddLabel.Font = Enum.Font.GothamBold
        ddLabel.Text = ddName
        ddLabel.TextColor3 = Library.Theme.TextWhite
        ddLabel.TextSize = 13
        ddLabel.TextXAlignment = Enum.TextXAlignment.Left
        ddLabel.Parent = headerBtn

        local valLabel = Instance.new("TextLabel")
        valLabel.Position = UDim2.new(0.5, 0, 0, 0)
        valLabel.Size = UDim2.new(0.5, -45, 1, 0)
        valLabel.BackgroundTransparency = 1
        valLabel.Font = Enum.Font.GothamMedium
        valLabel.Text = tostring(selected)
        valLabel.TextColor3 = Library.Theme.AccentBrightRed
        valLabel.TextSize = 12
        valLabel.TextTruncate = Enum.TextTruncate.AtEnd
        valLabel.TextXAlignment = Enum.TextXAlignment.Right
        valLabel.Parent = headerBtn

        local arrow = Instance.new("TextLabel")
        arrow.Position = UDim2.new(1, -30, 0, 0)
        arrow.Size = UDim2.new(0, 20, 1, 0)
        arrow.BackgroundTransparency = 1
        arrow.Font = Enum.Font.GothamBold
        arrow.Text = "▼"
        arrow.TextColor3 = Library.Theme.TextMuted
        arrow.TextSize = 10
        arrow.Parent = headerBtn

        local optionsHolder = Instance.new("ScrollingFrame")
        optionsHolder.Position = UDim2.new(0, 10, 0, 46)
        optionsHolder.Size = UDim2.new(1, -20, 0, 0)
        optionsHolder.BackgroundTransparency = 1
        optionsHolder.BorderSizePixel = 0
        optionsHolder.ScrollBarThickness = 3
        optionsHolder.ScrollBarImageColor3 = Library.Theme.Border
        optionsHolder.ZIndex = 16
        optionsHolder.Parent = ddContainer

        local optLayout = Instance.new("UIListLayout")
        optLayout.Padding = UDim.new(0, 3)
        optLayout.Parent = optionsHolder

        local optPadding = Instance.new("UIPadding")
        optPadding.PaddingRight = UDim.new(0, 4)
        optPadding.Parent = optionsHolder

        local optButtons = {}
        local function renderOptionsList()
            for _, b in ipairs(optButtons) do b:Destroy() end
            optButtons = {}

            local curOpts = (type(ddOptions) == "function" and ddOptions()) or ddOptions or {}
            for _, optText in ipairs(curOpts) do
                local optBtn = Instance.new("TextButton")
                optBtn.Size = UDim2.new(1, 0, 0, 30)
                optBtn.BackgroundColor3 = (selected == optText) and Color3.fromRGB(34, 14, 20) or Color3.fromRGB(22, 22, 28)
                optBtn.Font = Enum.Font.GothamMedium
                optBtn.Text = "   " .. tostring(optText)
                optBtn.TextColor3 = (selected == optText) and Color3.fromRGB(255, 255, 255) or Library.Theme.TextLight
                optBtn.TextSize = 12
                optBtn.TextXAlignment = Enum.TextXAlignment.Left
                optBtn.BorderSizePixel = 0
                optBtn.ZIndex = 17
                optBtn.Parent = optionsHolder

                local obc = Instance.new("UICorner")
                obc.CornerRadius = UDim.new(0, 6)
                obc.Parent = optBtn

                optBtn.MouseButton1Click:Connect(function()
                    playClickSound()
                    selected = optText
                    valLabel.Text = tostring(selected)
                    isOpen = false
                    Tween(ddContainer, { Size = UDim2.new(1, 0, 0, 46) }, 0.2)
                    Tween(arrow, { Rotation = 0 }, 0.2)
                    pcall(ddCallback, selected)
                    renderOptionsList()
                end)
                table.insert(optButtons, optBtn)
            end
            local totalH = #curOpts * 33
            optionsHolder.CanvasSize = UDim2.new(0, 0, 0, totalH)
        end
        renderOptionsList()

        headerBtn.MouseButton1Click:Connect(function()
            playClickSound()
            isOpen = not isOpen
            local curOpts = (type(ddOptions) == "function" and ddOptions()) or ddOptions or {}
            local totalH = #curOpts * 33
            local visibleH = math.clamp(totalH, 0, 180)
            optionsHolder.Size = UDim2.new(1, -20, 0, visibleH)
            local targetH = isOpen and (46 + visibleH + 8) or 46
            Tween(ddContainer, { Size = UDim2.new(1, 0, 0, targetH) }, 0.22)
            Tween(arrow, { Rotation = isOpen and 180 or 0 }, 0.2)
        end)

        local ret = {
            Set = function(val)
                selected = val
                valLabel.Text = tostring(val)
                pcall(ddCallback, val)
                renderOptionsList()
            end,
            SetValue = function(val)
                selected = val
                valLabel.Text = tostring(val)
                pcall(ddCallback, val)
                renderOptionsList()
            end,
            SetOptions = function(newOpts)
                ddOptions = newOpts
                renderOptionsList()
            end,
            Get = function() return selected end
        }
        Library.Elements[id] = ret
        return ret
    end

    -- 5. Component: AddButton
    function TabObj:AddButton(arg1, arg2)
    local btnConfig = (type(arg1) == "table" and arg1) or (type(arg2) == "table" and arg2) or {}
    local id = (type(arg1) == "string" and arg1) or btnConfig.Name or btnConfig.Title or "Button"
        local bName = btnConfig.Name or btnConfig.Title or "Button"
        local bDesc = btnConfig.Desc or btnConfig.Description or ""
        local bCallback = btnConfig.Callback or function() end

        local btnFrame = Instance.new("TextButton")
        btnFrame.Size = UDim2.new(1, 0, 0, #bDesc > 0 and 52 or 44)
        btnFrame.BackgroundColor3 = btnConfig.Primary and Color3.fromRGB(34, 14, 20) or Library.Theme.CardBackground
        btnFrame.AutoButtonColor = false
        btnFrame.Text = ""
        btnFrame.BorderSizePixel = 0
        btnFrame.ZIndex = 10
        btnFrame.Parent = pageScroll

        local bc = Instance.new("UICorner")
        bc.CornerRadius = UDim.new(0, 10)
        bc.Parent = btnFrame

        local bs = Instance.new("UIStroke")
        bs.Color = btnConfig.Primary and Library.Theme.Border or Library.Theme.BorderSubtle
        bs.Thickness = 1
        bs.Parent = btnFrame

        local bText = Instance.new("TextLabel")
        bText.Position = UDim2.new(0, 16, 0, #bDesc > 0 and 8 or 0)
        bText.Size = UDim2.new(1, -70, 0, #bDesc > 0 and 18 or 44)
        bText.BackgroundTransparency = 1
        bText.Font = Enum.Font.GothamBold
        bText.Text = bName
        bText.TextColor3 = btnConfig.Primary and Color3.fromRGB(255, 255, 255) or Library.Theme.TextWhite
        bText.TextSize = 13
        bText.TextXAlignment = Enum.TextXAlignment.Left
        bText.Parent = btnFrame

        if #bDesc > 0 then
            local bDescLbl = Instance.new("TextLabel")
            bDescLbl.Position = UDim2.new(0, 16, 0, 28)
            bDescLbl.Size = UDim2.new(1, -70, 0, 16)
            bDescLbl.BackgroundTransparency = 1
            bDescLbl.Font = Enum.Font.GothamMedium
            bDescLbl.Text = bDesc
            bDescLbl.TextColor3 = Library.Theme.TextMuted
            bDescLbl.TextSize = 11
            bDescLbl.TextXAlignment = Enum.TextXAlignment.Left
            bDescLbl.Parent = btnFrame
        end

        local bArrow = Instance.new("TextLabel")
        bArrow.AnchorPoint = Vector2.new(1, 0.5)
        bArrow.Position = UDim2.new(1, -16, 0.5, 0)
        bArrow.Size = UDim2.new(0, 20, 0, 20)
        bArrow.BackgroundTransparency = 1
        bArrow.Font = Enum.Font.GothamBold
        bArrow.Text = ">"
        bArrow.TextColor3 = Library.Theme.AccentRed
        bArrow.TextSize = 16
        bArrow.Parent = btnFrame

        btnFrame.MouseEnter:Connect(function()
            Tween(btnFrame, { BackgroundColor3 = Library.Theme.CardBackgroundHover }, 0.15)
            Tween(bs, { Color = Library.Theme.Border }, 0.15)
        end)
        btnFrame.MouseLeave:Connect(function()
            Tween(btnFrame, { BackgroundColor3 = btnConfig.Primary and Color3.fromRGB(34, 14, 20) or Library.Theme.CardBackground }, 0.15)
            Tween(bs, { Color = btnConfig.Primary and Library.Theme.Border or Library.Theme.BorderSubtle }, 0.15)
        end)
        btnFrame.MouseButton1Click:Connect(function()
            playClickSound()
            Tween(btnFrame, { BackgroundColor3 = Color3.fromRGB(35, 35, 45) }, 0.08)
            task.delay(0.08, function()
                Tween(btnFrame, { BackgroundColor3 = Library.Theme.CardBackgroundHover }, 0.12)
            end)
            pcall(bCallback)
        end)

        return btnFrame
    end

    -- 6. Component: AddInput
    function TabObj:AddInput(arg1, arg2)
    local inConfig = (type(arg1) == "table" and arg1) or (type(arg2) == "table" and arg2) or {}
    local id = (type(arg1) == "string" and arg1) or inConfig.Name or inConfig.Title or "Input"
        local iName = inConfig.Name or inConfig.Title or id
        local iDefault = inConfig.Default or ""
        local iPlaceholder = inConfig.Placeholder or "พิมพ์ข้อความ..."
        local iCallback = inConfig.Callback or function() end
        local value = iDefault

        local inFrame = Instance.new("Frame")
        inFrame.Size = UDim2.new(1, 0, 0, 48)
        inFrame.BackgroundColor3 = Library.Theme.CardBackground
        inFrame.BorderSizePixel = 0
        inFrame.ZIndex = 10
        inFrame.Parent = pageScroll

        local inc = Instance.new("UICorner")
        inc.CornerRadius = UDim.new(0, 10)
        inc.Parent = inFrame

        local ins = Instance.new("UIStroke")
        ins.Color = Library.Theme.BorderSubtle
        ins.Thickness = 1
        ins.Parent = inFrame

        local inLabel = Instance.new("TextLabel")
        inLabel.Position = UDim2.new(0, 16, 0, 0)
        inLabel.Size = UDim2.new(0.48, 0, 1, 0)
        inLabel.BackgroundTransparency = 1
        inLabel.Font = Enum.Font.GothamBold
        inLabel.Text = iName
        inLabel.TextColor3 = Library.Theme.TextWhite
        inLabel.TextSize = 13
        inLabel.TextXAlignment = Enum.TextXAlignment.Left
        inLabel.Parent = inFrame

        local box = Instance.new("TextBox")
        box.Position = UDim2.new(0.5, 0, 0.5, -16)
        box.Size = UDim2.new(0.5, -16, 0, 32)
        box.BackgroundColor3 = Color3.fromRGB(12, 12, 16)
        box.BorderSizePixel = 0
        box.Font = Enum.Font.GothamMedium
        box.Text = tostring(value)
        box.PlaceholderText = iPlaceholder
        box.TextColor3 = Library.Theme.TextCyan
        box.PlaceholderColor3 = Color3.fromRGB(120, 120, 140)
        box.TextSize = 12
        box.Parent = inFrame

        local bc = Instance.new("UICorner")
        bc.CornerRadius = UDim.new(0, 6)
        bc.Parent = box

        local bs = Instance.new("UIStroke")
        bs.Color = Library.Theme.BorderSubtle
        bs.Thickness = 1
        bs.Parent = box

        box.FocusLost:Connect(function()
            value = box.Text
            pcall(iCallback, value)
        end)

        local ret = {
            Set = function(val)
                value = tostring(val)
                box.Text = value
                pcall(iCallback, value)
            end,
            SetValue = function(val)
                value = tostring(val)
                box.Text = value
                pcall(iCallback, value)
            end,
            Get = function() return value end
        }
        Library.Elements[id] = ret
        return ret
    end

    -- 7. Component: AddParagraph
    function TabObj:AddParagraph(arg1, arg2)
        local pConfig = (type(arg1) == "table" and arg1) or (type(arg2) == "table" and arg2) or {}
        local pTitle = pConfig.Title or pConfig.Name or (type(arg1) == "string" and arg1) or "Information"
        local pContent = pConfig.Content or pConfig.Desc or pConfig.Text or ""

        local pFrame = Instance.new("Frame")
        pFrame.Size = UDim2.new(1, 0, 0, 64)
        pFrame.BackgroundColor3 = Library.Theme.CardBackground
        pFrame.BorderSizePixel = 0
        pFrame.ZIndex = 10
        pFrame.Parent = pageScroll

        local pc = Instance.new("UICorner")
        pc.CornerRadius = UDim.new(0, 10)
        pc.Parent = pFrame

        local ps = Instance.new("UIStroke")
        ps.Color = Library.Theme.BorderSubtle
        ps.Thickness = 1
        ps.Parent = pFrame

        local pTitleLbl = Instance.new("TextLabel")
        pTitleLbl.Position = UDim2.new(0, 16, 0, 8)
        pTitleLbl.Size = UDim2.new(1, -32, 0, 18)
        pTitleLbl.BackgroundTransparency = 1
        pTitleLbl.Font = Enum.Font.GothamBold
        pTitleLbl.Text = pTitle
        pTitleLbl.TextColor3 = Library.Theme.TextWhite
        pTitleLbl.TextSize = 13
        pTitleLbl.TextXAlignment = Enum.TextXAlignment.Left
        pTitleLbl.Parent = pFrame

        local pContentLbl = Instance.new("TextLabel")
        pContentLbl.Position = UDim2.new(0, 16, 0, 28)
        pContentLbl.Size = UDim2.new(1, -32, 0, 30)
        pContentLbl.BackgroundTransparency = 1
        pContentLbl.Font = Enum.Font.GothamMedium
        pContentLbl.Text = pContent
        pContentLbl.TextColor3 = Library.Theme.TextMuted
        pContentLbl.TextSize = 11
        pContentLbl.TextXAlignment = Enum.TextXAlignment.Left
        pContentLbl.TextWrapped = true
        pContentLbl.Parent = pFrame

        return {
            Set = function(newText) pContentLbl.Text = tostring(newText) end,
            SetTitle = function(newT) pTitleLbl.Text = tostring(newT) end
        }
    end

    return TabObj
end

-- ==============================================================================
-- [[ MM2 BACKEND ENGINE & STATE CONTROLLERS ]]
-- ==============================================================================

local function GetCharacter() return LocalPlayer.Character end
local function GetHumanoid()
    local c = GetCharacter()
    return c and c:FindFirstChildOfClass("Humanoid")
end
local function GetHRP()
    local c = GetCharacter()
    return c and c:FindFirstChild("HumanoidRootPart")
end
local function GetRootCFrame()
    local hrp = GetHRP()
    return hrp and hrp.CFrame
end

local function GetCoins()
    local ls = LocalPlayer:FindFirstChild("leaderstats")
    local c = ls and (ls:FindFirstChild("Coins") or ls:FindFirstChild("Coin"))
    if c and typeof(c.Value) == "number" then return c.Value end
    local df = LocalPlayer:FindFirstChild("DataFolder")
    local c2 = df and df:FindFirstChild("Coins")
    if c2 then return tonumber(c2.Value) or 0 end
    return 0
end

local function GetLevel()
    local ls = LocalPlayer:FindFirstChild("leaderstats")
    local l = ls and ls:FindFirstChild("Level")
    if l then return tonumber(l.Value) or 0 end
    return 0
end

local function GetRoundPlayerData()
    local ok, crc = pcall(function()
        return require(game:GetService("ReplicatedStorage"):WaitForChild("Modules", 2):WaitForChild("CurrentRoundClient", 2))
    end)
    if ok and crc then
        local pData = crc.GetLatestPlayerData and crc.GetLatestPlayerData() or crc.PlayerData
        if pData and type(pData) == "table" then
            return pData
        end
    end
    return nil
end

local function GetRole(plr)
    plr = plr or LocalPlayer
    local pData = GetRoundPlayerData()
    if pData and pData[plr.Name] and pData[plr.Name].Role then
        if pData[plr.Name].Dead or pData[plr.Name].Killed then return "Dead" end
        return pData[plr.Name].Role
    end
    local char = plr.Character
    if not char then return "Unknown" end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum and hum.Health <= 0 then return "Dead" end
    for _, t in ipairs(char:GetChildren()) do
        if t:IsA("Tool") then
            local n = t.Name:lower()
            if n:find("knife") then return "Murderer" end
            if n:find("gun") then return "Sheriff" end
        end
    end
    local bp = plr:FindFirstChild("Backpack")
    if bp then
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") and t.Name:lower():find("knife") then return "Murderer?" end
            if t:IsA("Tool") and t.Name:lower():find("gun") then return "Sheriff?" end
        end
    end
    return "Innocent"
end

local function RoleColor(role)
    if role == "Murderer" or role == "Murderer?" then
        return Color3.fromRGB(245, 45, 65)
    elseif role == "Sheriff" or role == "Sheriff?" or role == "Hero" then
        return Color3.fromRGB(45, 125, 255)
    elseif role == "Dead" then
        return Color3.fromRGB(130, 130, 140)
    else
        return Color3.fromRGB(46, 224, 140)
    end
end

local function isMurderer()
    return GetRole(LocalPlayer) == "Murderer"
end

local function getMurdererPlayer()
    local pData = GetRoundPlayerData()
    if pData then
        for name, info in pairs(pData) do
            if info.Role == "Murderer" and not info.Dead and not info.Killed then
                local p = Players:FindFirstChild(name)
                if p then return p end
            end
        end
    end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local char = plr.Character
            if char then
                for _, t in ipairs(char:GetChildren()) do
                    if t:IsA("Tool") and t.Name:lower():find("knife") then return plr end
                end
            end
            local bp = plr:FindFirstChild("Backpack")
            if bp then
                for _, t in ipairs(bp:GetChildren()) do
                    if t:IsA("Tool") and t.Name:lower():find("knife") then return plr end
                end
            end
        end
    end
    return nil
end

local function getSheriffPlayer()
    local pData = GetRoundPlayerData()
    if pData then
        for name, info in pairs(pData) do
            if (info.Role == "Sheriff" or info.Role == "Hero") and not info.Dead and not info.Killed then
                local p = Players:FindFirstChild(name)
                if p then return p end
            end
        end
    end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local char = plr.Character
            if char then
                for _, t in ipairs(char:GetChildren()) do
                    if t:IsA("Tool") and t.Name:lower():find("gun") then return plr end
                end
            end
            local bp = plr:FindFirstChild("Backpack")
            if bp then
                for _, t in ipairs(bp:GetChildren()) do
                    if t:IsA("Tool") and t.Name:lower():find("gun") then return plr end
                end
            end
        end
    end
    return nil
end

local function getGunDrop()
    local map = getActiveMap and getActiveMap()
    if map then
        local g = map:FindFirstChild("GunDrop")
        if g then return g end
    end
    local normal = Workspace:FindFirstChild("Normal")
    if normal then
        local g = normal:FindFirstChild("GunDrop")
        if g then return g end
    end
    local g = Workspace:FindFirstChild("GunDrop")
    if g then return g end
    for _, it in ipairs(Workspace:GetChildren()) do
        if it:IsA("Tool") and it.Name:lower():find("gun") and not it:IsDescendantOf(LocalPlayer) and not (it.Parent and it.Parent:FindFirstChildOfClass("Humanoid")) then
            return it
        end
    end
    return nil
end

local function isRoundActive()
    if _CFG and _CFG.AlwaysActiveOverride then return true end
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    local char = LocalPlayer.Character
    if not pg or not pg:FindFirstChild("MainGUI") or not char or not char:FindFirstChild("HumanoidRootPart") then return false end

    local gameUI = pg.MainGUI:FindFirstChild("Game")
    local spectateUI = pg.MainGUI:FindFirstChild("Spectate")
    if not gameUI or not gameUI.Visible then return false end
    if spectateUI and spectateUI.Visible then return false end

    local lobby = Workspace:FindFirstChild("Lobby")
    if lobby then
        local rayParams = RaycastParams.new()
        rayParams.FilterType = Enum.RaycastFilterType.Exclude
        local ignoreList = {}
        for _, p in pairs(Players:GetPlayers()) do
            if p.Character then table.insert(ignoreList, p.Character) end
        end
        rayParams.FilterDescendantsInstances = ignoreList

        local hrp = char.HumanoidRootPart
        local rayResult = Workspace:Raycast(hrp.Position, Vector3.new(0, -1000, 0), rayParams)
        if rayResult and rayResult.Instance and rayResult.Instance:IsDescendantOf(lobby) then
            return false
        end
    end
    return true
end

-- Global Configuration Dictionary
local _CFG = {
    -- Farm & Event
    AlwaysActiveOverride = true,
    AutoFarm = false,
    AutoFarmCandies = false,
    FarmMode = "Tween", -- "Tween", "Instant", "Ghost"
    FarmTarget = "All", -- "All", "Coins Only", "Candies Only"
    FarmSpeed = 95,
    Dwell = 0.6,
    TargetCoins = 40,
    TargetCandies = 40,
    AutoResetBag = true,
    BagFullAction = "Reset", -- "Reset", "SafeSpot"
    SafeSpotType = "Roof", -- "Roof", "Underground", "Lobby"
    FleeFromMurderer = true,
    FleeDistance = 35,
    AutoClaimBoxes = false,
    AutoOpenSelectedCrate = false,
    SelectedCrate = "MysteryBox",
    SelectedCrateCurrency = "Coins",
    AutoOpenEventCrates = false,
    SelectedEventCrate = "Summer2026Box",
    AutoClaimBattlePass = false,

    -- Emotes & Animations Controller
    SelectedEmote = "ninja",
    EmoteSpeed = 1.0,
    LoopEmote = false,
    DanceWalk = false,
    CustomEmoteId = "",

    -- Traps & Defense
    TrapESP = true,
    AutoDisarmTraps = false,
    AntiTrapNeutralize = true,
    TrapDisarmDistance = 25,
    AntiMurderer = false,
    AntiMurdererDist = 30,

    -- Combat & Sheriff
    AutoShootMurderer = false,
    ShootMurdererDist = 80,
    KillAura = false,
    KillAuraDist = 18,
    KillAuraDelay = 0.35,
    AutoGrabGun = false,
    SilentAim = false,
    SilentHitChance = 100,
    SilentHitPart = "Head",
    SilentFOV = 150,
    SilentShowFOV = false,
    SilentTargetMode = "Murderer",
    SilentChosen = nil,
    SilentWallCheck = false,
    SilentTeamCheck = true,
    HitboxExpander = false,
    HitboxSize = 8,

    -- Visuals ESP
    MasterESP = true,
    PlayerESP = true,
    RoleESP = true,
    CoinESP = true,
    GunESP = true,
    BoxESP = false,
    BoxStyle = "Corner",
    BoxThickness = 1,
    BoxFill = false,
    Chams = true,
    ChamsFillTransparency = 0.5,
    ChamsOutlineTransparency = 0,
    HaloESP = false,
    NameESP = true,
    DistESP = true,
    HealthESP = false,
    TextSize = 13,
    Tracers = false,
    TracerOrigin = "Bottom",
    ArrowESP = false,
    HeadCircle = false,
    SkeletonESP = false,
    RainbowESP = false,
    MaxESPDistance = 1500,

    -- Movement & Physics
    WalkSpeedEnabled = false,
    WalkSpeedValue = 32,
    JumpPowerEnabled = false,
    JumpPowerValue = 75,
    InfJump = false,
    HipHeightEnabled = false,
    HipHeightValue = 2,
    GravityEnabled = false,
    GravityValue = 196.2,
    Fly = false,
    FlySpeed = 50,
    Noclip = false,
    AntiRagdoll = false,

    -- Teleport & Waypoints
    SelectedTPPlayer = nil,
    SelectedWaypoint = nil,
    CustomWaypoints = {},
    ClickTP = false,
    TweenTP = false,
    SelectedMapSpot = nil,

    -- World & Environment
    Fullbright = false,
    CameraFOV = 70,
    LockTime = false,
    ClockTime = 14,
    NoFog = false,
    SelectedShopGun = nil,
    AutoBuyLowAmmo = false,
    AutoBuyAmmo = false,

    -- System & Misc
    HopMaxPlayers = 12,
    AntiAFK = true,
    BoostFPS = false,
    BlackScreen = false,
    UnlockFPS = false,
    FPSCap = 240,
    DiscordWebhookURL = "",

    -- Config System
    AutoLoadConfig = true,
    AutoSaveConfig = true,
    SelectedConfigSlot = "Default"
}


-- ==============================================================================
-- [[ PAYOMBOYZ ZERO-DAY PERSISTENT CONFIGURATION ENGINE ]]
-- ==============================================================================

local CONFIG_FOLDER = "PayomboyZ_Hub"
local DEFAULT_CONFIG_NAME = "MM2_Default.json"

local CFG_KEY_MAP = {
    AutoFarmToggle = "AutoFarm",
    AutoCandyToggle = "AutoFarmCandies",
    FarmTargetDropdown = "FarmTarget",
    FarmModeDropdown = "FarmMode",
    FarmSpeedSlider = "FarmSpeed",
    DwellSlider = "Dwell",
    TargetCoinsSlider = "TargetCoins",
    AutoResetBagToggle = "AutoResetBag",
    BagFullActionDropdown = "BagFullAction",
    SafeSpotDropdown = "SafeSpotType",
    AlwaysActiveToggle = "AlwaysActiveOverride",
    FleeFromMurdererToggle = "FleeFromMurderer",
    AutoClaimBoxesToggle = "AutoClaimBoxes",
    EmoteDropdown = "SelectedEmote",
    LoopEmoteToggle = "LoopEmote",
    DanceWalkToggle = "DanceWalk",
    EmoteSpeedSlider = "EmoteSpeed",
    CustomEmoteIdInput = "CustomEmoteId",
    KillAuraToggle = "KillAura",
    KillDistSlider = "KillAuraDist",
    KillDelaySlider = "KillAuraDelay",
    AutoGrabGunToggle = "AutoGrabGun",
    SilentAimToggle = "SilentAim",
    SilentModeDropdown = "SilentTargetMode",
    SilentHitPartDropdown = "SilentHitPart",
    SilentHitChanceSlider = "SilentHitChance",
    SilentFOVSlider = "SilentFOV",
    SilentShowFOVToggle = "SilentShowFOV",
    SilentWallCheckToggle = "SilentWallCheck",
    SilentTeamCheckToggle = "SilentTeamCheck",
    HitboxToggle = "HitboxExpander",
    HitboxSizeSlider = "HitboxSize",
    MasterESPToggle = "MasterESP",
    PlayerESPToggle = "PlayerESP",
    RoleESPToggle = "RoleESP",
    CoinESPToggle = "CoinESP",
    GunESPToggle = "GunESP",
    BoxESPToggle = "BoxESP",
    BoxStyleDropdown = "BoxStyle",
    BoxThickSlider = "BoxThickness",
    BoxFillToggle = "BoxFill",
    ChamsToggle = "Chams",
    ChamsFillSlider = "ChamsFillTransparency",
    ChamsOutlineSlider = "ChamsOutlineTransparency",
    HaloToggle = "HaloESP",
    NameESPToggle = "NameESP",
    DistESPToggle = "DistESP",
    HealthESPToggle = "HealthESP",
    TextSizeSlider = "TextSize",
    TracerToggle = "Tracers",
    TracerOriginDropdown = "TracerOrigin",
    ArrowToggle = "ArrowESP",
    HeadCircleToggle = "HeadCircle",
    SkeletonToggle = "SkeletonESP",
    RainbowToggle = "RainbowESP",
    MaxESPDistSlider = "MaxESPDistance",
    WalkSpeedToggle = "WalkSpeedEnabled",
    WalkSpeedSlider = "WalkSpeedValue",
    JumpPowerToggle = "JumpPowerEnabled",
    JumpPowerSlider = "JumpPowerValue",
    InfJumpToggle = "InfJump",
    HipHeightToggle = "HipHeightEnabled",
    HipHeightSlider = "HipHeightValue",
    GravityToggle = "GravityEnabled",
    GravitySlider = "GravityValue",
    FlyToggle = "Fly",
    FlySpeedSlider = "FlySpeed",
    NoclipToggle = "Noclip",
    AntiRagdollToggle = "AntiRagdoll",
    FullbrightToggle = "Fullbright",
    FOVSlider = "CameraFOV",
    LockTimeToggle = "LockTime",
    ClockTimeSlider = "ClockTime",
    NoFogToggle = "NoFog",
    AntiAFKToggle = "AntiAFK",
    BoostFPSToggle = "BoostFPS",
    BlackScreenToggle = "BlackScreen",
    UnlockFPSToggle = "UnlockFPS",
    FPSCapSlider = "FPSCap",
    AutoLoadConfigToggle = "AutoLoadConfig",
    AutoSaveConfigToggle = "AutoSaveConfig",
    ConfigSlotDropdown = "SelectedConfigSlot"
}

local function ensureConfigFolder()
    pcall(function()
        if typeof(makefolder) == "function" and typeof(isfolder) == "function" then
            if not isfolder(CONFIG_FOLDER) then
                makefolder(CONFIG_FOLDER)
            end
        end
    end)
end

local function getSavePath(slotName)
    slotName = slotName or _CFG.SelectedConfigSlot or "Default"
    local safeName = tostring(slotName):gsub("[^%w_%-]", "")
    if safeName == "" then safeName = "Default" end
    return CONFIG_FOLDER .. "/MM2_" .. safeName .. ".json"
end

local function exportConfigData()
    local data = {}
    for k, v in pairs(_CFG) do
        local t = type(v)
        if t == "boolean" or t == "number" or t == "string" then
            data[k] = v
        elseif t == "table" and k == "CustomWaypoints" then
            local wp = {}
            for name, cframe in pairs(v) do
                if typeof(cframe) == "CFrame" then
                    local p = cframe.Position
                    wp[name] = { p.X, p.Y, p.Z }
                end
            end
            data[k] = wp
        end
    end
    return data
end

local function importConfigData(data)
    if type(data) ~= "table" then return false end
    for k, v in pairs(data) do
        if k == "CustomWaypoints" and type(v) == "table" then
            _CFG.CustomWaypoints = {}
            for name, coords in pairs(v) do
                if type(coords) == "table" and #coords >= 3 then
                    _CFG.CustomWaypoints[name] = CFrame.new(coords[1], coords[2], coords[3])
                end
            end
        else
            _CFG[k] = v
        end
    end

    -- Sync UI Elements with loaded config values
    for id, comp in pairs(Library.Elements) do
        pcall(function()
            local cfgKey = id:gsub("Toggle$", ""):gsub("Slider$", ""):gsub("Dropdown$", ""):gsub("Input$", "")
            local val = _CFG[id] or _CFG[cfgKey]
            if val == nil then
                local mapped = CFG_KEY_MAP[id]
                if mapped then val = _CFG[mapped] end
            end
            if val ~= nil and comp and comp.Set then
                comp.Set(val)
            end
        end)
    end
    return true
end

local function SaveConfig(slotName)
    slotName = slotName or _CFG.SelectedConfigSlot or "Default"
    ensureConfigFolder()
    local path = getSavePath(slotName)

    local ok, encoded = pcall(function()
        return HttpService:JSONEncode(exportConfigData())
    end)
    if not ok or not encoded then
        Notify("Config", "เข้ารหัสข้อมูล Config ล้มเหลว", "error")
        return false
    end

    local writeOk = false
    pcall(function()
        if typeof(writefile) == "function" then
            writefile(path, encoded)
            writeOk = true
        end
    end)

    if writeOk then
        Notify("Config", "บันทึก Config [" .. tostring(slotName) .. "] สำเร็จ!", "success")
        return true
    else
        -- Fallback directly to root
        local fallbackPath = "PayomboyZ_MM2_" .. tostring(slotName):gsub("[^%w_%-]", "") .. ".json"
        pcall(function()
            if typeof(writefile) == "function" then
                writefile(fallbackPath, encoded)
                writeOk = true
            end
        end)
        if writeOk then
            Notify("Config", "บันทึก Config [" .. tostring(slotName) .. "] สำเร็จ!", "success")
            return true
        end
        Notify("Config", "อุปกรณ์/Executor ไม่รองรับ writefile", "error")
        return false
    end
end

local function LoadConfig(slotName)
    slotName = slotName or _CFG.SelectedConfigSlot or "Default"
    local path = getSavePath(slotName)
    local hasFile = false
    pcall(function()
        if typeof(isfile) == "function" then
            hasFile = isfile(path)
        end
    end)

    if not hasFile then
        local fallbackPath = "PayomboyZ_MM2_" .. tostring(slotName):gsub("[^%w_%-]", "") .. ".json"
        pcall(function()
            if typeof(isfile) == "function" and isfile(fallbackPath) then
                path = fallbackPath
                hasFile = true
            end
        end)
    end

    if not hasFile then
        Notify("Config", "ไม่พบไฟล์ Config: " .. tostring(slotName), "error")
        return false
    end

    local raw = nil
    local readOk = false
    pcall(function()
        if typeof(readfile) == "function" then
            raw = readfile(path)
            readOk = true
        end
    end)

    if not readOk or not raw or #raw == 0 then
        Notify("Config", "อ่านไฟล์ Config ล้มเหลว", "error")
        return false
    end

    local decodeOk, data = pcall(function()
        return HttpService:JSONDecode(raw)
    end)

    if not decodeOk or type(data) ~= "table" then
        Notify("Config", "รูปแบบไฟล์ Config เสียหาย", "error")
        return false
    end

    importConfigData(data)
    Notify("Config", "โหลด Config [" .. tostring(slotName) .. "] เรียบร้อย!", "success")
    return true
end

local function ResetDefaultConfig()
    local defaults = {
        AutoFarm = false,
        AutoFarmCandies = false,
        FarmMode = "Tween",
        FarmTarget = "All",
        FarmSpeed = 95,
        Dwell = 0.6,
        TargetCoins = 40,
        TargetCandies = 40,
        AutoResetBag = true,
        BagFullAction = "Reset",
        SafeSpotType = "Roof",
        FleeFromMurderer = true,
        AutoShootMurderer = false,
        KillAura = false,
        SilentAim = false,
        SilentFOV = 150,
        SilentHitChance = 100,
        HitboxExpander = false,
        HitboxSize = 8,
        MasterESP = true,
        PlayerESP = true,
        WalkSpeedEnabled = false,
        WalkSpeedValue = 32,
        JumpPowerEnabled = false,
        JumpPowerValue = 75,
        AntiAFK = true
    }
    importConfigData(defaults)
    Notify("Config", "รีเซ็ต Config ทั้งหมดเป็นค่าเริ่มต้นแล้ว", "info")
end

-- ==============================================================================
-- [[ INITIALIZE ALL TABS ]]
-- ==============================================================================

local HomeTab     = Library:CreateTab({ Name = "หน้าหลัก", Icon = "rbxassetid://10723407389", Subtitle = "ภาพรวมและแดชบอร์ดเกม Murder Mystery 2" })
local FarmTab     = Library:CreateTab({ Name = "ฟาร์มเหรียญ", Icon = "rbxassetid://10709810534", Subtitle = "ระบบเก็บเหรียญอัตโนมัติ & กิจกรรมลูกอม" })
local CombatTab   = Library:CreateTab({ Name = "คอมแบท & เรจ", Icon = "rbxassetid://10747373176", Subtitle = "ระบบฆาตกร, นายอำเภอ Silent Aim & Hitbox" })
local VisualsTab  = Library:CreateTab({ Name = "การมองเห็น (ESP)", Icon = "rbxassetid://10723415205", Subtitle = "ESP กล่อง, Chams ทะลุกำแพง, หลอดเลือด & Tracer" })
local CharTab     = Library:CreateTab({ Name = "ตัวละคร", Icon = "rbxassetid://10747372702", Subtitle = "เร่งความเร็วเดิน, บิน, ทะลุกำแพง & ท่าเต้น Emotes" })
local TeleportTab = Library:CreateTab({ Name = "วาร์ป & แผนที่", Icon = "rbxassetid://10734898168", Subtitle = "วาร์ปหาผู้เล่น, จุดวาร์ปส่วนตัว & พิกัด MM2" })
local WorldTab    = Library:CreateTab({ Name = "โลก & ร้านค้า", Icon = "rbxassetid://10734950309", Subtitle = "ปรับแสงสว่าง, ลบหมอก & ระบบซื้อปืนร้านค้า" })
local SystemTab   = Library:CreateTab({ Name = "ระบบ & ตั้งค่า", Icon = "rbxassetid://10734975692", Subtitle = "ประสิทธิภาพ, ย้ายเซิร์ฟเวอร์, Webhook & Unload" })

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 1: หน้าหลัก (HOME OVERVIEW DASHBOARD)
-- ──────────────────────────────────────────────────────────────────────────────
HomeTab:AddSection("แดชบอร์ดผู้เล่น (PLAYER OVERVIEW)")

local heroCard = HomeTab:AddParagraph({
    Title = "ยินดีต้อนรับ, " .. tostring(LocalPlayer.DisplayName or LocalPlayer.Name or "Player") .. " (@" .. LocalPlayer.Name .. ")",
    Content = "PayomboyZ MM2 Master Suite ออกแบบสไตล์ Red Neon Obsidian Glass\nระบบพร้อมทำงานสมบูรณ์ทุกฟังก์ชัน พร้อมเทคโนโลยี Bypassed RenderStepped Loops"
})

local balanceCard = HomeTab:AddParagraph({
    Title = "ข้อมูลบัญชีและสเตตัสในรอบปัจจุบัน",
    Content = string.format("เหรียญ: %d | เลเวล: %d | บทบาท: %s", GetCoins(), GetLevel(), GetRole())
})

HomeTab:AddSection("คีย์ลัดด่วน (QUICK ACTIONS)")

HomeTab:AddButton({
    Name = "สลับสถานะ ออโต้ฟาร์มเหรียญ (Toggle Coin Farm)",
    Description = "เปิด/ปิด การเก็บเหรียญอัตโนมัติทันที",
    Primary = true,
    Callback = function()
        _CFG.AutoFarm = not _CFG.AutoFarm
        Notify("Farm", _CFG.AutoFarm and "เปิดระบบฟาร์มเหรียญแล้ว!" or "ปิดระบบฟาร์มเหรียญแล้ว", _CFG.AutoFarm and "success" or "info")
    end
})

HomeTab:AddButton({
    Name = "วาร์ปไปเก็บปืนที่ตกทันที (Grab Dropped Gun)",
    Description = "ตรวจหาปืนนายอำเภอที่ตกอยู่และวาร์ปไปหยิบทันที",
    Callback = function()
        local gun = getGunDrop()
        local hrp = GetHRP()
        if gun and hrp then
            local pos = gun:IsA("BasePart") and gun.Position or gun:GetPivot().Position
            hrp.CFrame = CFrame.new(pos + Vector3.new(0, 2.5, 0))
            if firetouchinterest then
                local h = gun:IsA("BasePart") and gun or gun:FindFirstChild("Handle") or gun:FindFirstChildWhichIsA("BasePart")
                if h then pcall(function() firetouchinterest(hrp, h, 0); firetouchinterest(hrp, h, 1) end) end
            end
            Notify("Gun", "วาร์ปไปหยิบปืนแล้ว!", "success")
        else
            Notify("Gun", "ไม่พบปืนที่ตกในขณะนี้", "error")
        end
    end
})

HomeTab:AddButton({
    Name = "สลับสถานะ ESP ผู้เล่นทั้งหมด (Toggle Master ESP)",
    Callback = function()
        _CFG.MasterESP = not _CFG.MasterESP
        Notify("ESP", _CFG.MasterESP and "เปิดการมองเห็น ESP แล้ว" or "ปิดการมองเห็น ESP แล้ว", _CFG.MasterESP and "success" or "info")
    end
})

HomeTab:AddButton({
    Name = "เปิดโหมดสปีดเดินเร็ว (WalkSpeed 90)",
    Callback = function()
        _CFG.WalkSpeedEnabled = not _CFG.WalkSpeedEnabled
        _CFG.WalkSpeedValue = 90
        local hum = GetHumanoid()
        if hum then hum.WalkSpeed = _CFG.WalkSpeedEnabled and 90 or 16 end
        Notify("Speed", _CFG.WalkSpeedEnabled and "เปิดสปีด 90 spd แล้ว" or "คืนค่าสปีดปกติแล้ว", "info")
    end
})

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 2: ฟาร์มเหรียญ & กิจกรรม (AUTO FARM)
-- ──────────────────────────────────────────────────────────────────────────────
FarmTab:AddSection("// ระบบฟาร์มเหรียญ & กิจกรรมลูกอม (COIN & CANDY FARM)")

FarmTab:AddToggle("AutoFarmToggle", {
    Name = "เก็บเหรียญอัตโนมัติ (Auto Farm Coins)",
    Description = "วาร์ป/ทวีนเก็บเหรียญทองรอบแมพอย่างแม่นยำและปลอดภัย",
    Default = false,
    Callback = function(v) _CFG.AutoFarm = v end
})

FarmTab:AddToggle("AutoCandyToggle", {
    Name = "เก็บลูกอม/เหรียญกิจกรรม (Auto Farm Candies/Event)",
    Description = "ตรวจจับและเก็บลูกอม แคนดี้ หรือเหรียญอีเวนต์เทศกาลอัตโนมัติ",
    Default = false,
    Callback = function(v) _CFG.AutoFarmCandies = v end
})

FarmTab:AddDropdown("FarmTargetDropdown", {
    Name = "เป้าหมายการฟาร์ม (Farm Target)",
    Options = { "All (เหรียญ + ลูกอม)", "Coins Only (เหรียญเท่านั้น)", "Candies Only (ลูกอมเท่านั้น)" },
    Default = "All (เหรียญ + ลูกอม)",
    Callback = function(v)
        if v:find("Coins") then _CFG.FarmTarget = "Coins Only"
        elseif v:find("Candies") then _CFG.FarmTarget = "Candies Only"
        else _CFG.FarmTarget = "All" end
    end
})

FarmTab:AddDropdown("FarmModeDropdown", {
    Name = "โหมดการเคลื่อนที่ (Farm Mode)",
    Options = { "Tween (ทวีนเนียน ปลอดภัย)", "Instant (วาร์ปทันที เก็บไว)", "Ghost (มุดใต้ดินขึ้นมาเก็บ)" },
    Default = "Tween (ทวีนเนียน ปลอดภัย)",
    Callback = function(v)
        if v:find("Instant") then _CFG.FarmMode = "Instant"
        elseif v:find("Ghost") then _CFG.FarmMode = "Ghost"
        else _CFG.FarmMode = "Tween" end
    end
})

FarmTab:AddSlider("FarmSpeedSlider", {
    Name = "ความเร็วการทวีน (Farm Speed)",
    Min = 20,
    Max = 250,
    Default = 95,
    Suffix = " spd",
    Callback = function(v) _CFG.FarmSpeed = v end
})

FarmTab:AddSlider("DwellSlider", {
    Name = "ระยะเวลาหน่วงหลังเก็บ (Dwell Delay)",
    Min = 0.1,
    Max = 2.5,
    Default = 0.6,
    Rounding = 1,
    Suffix = "s",
    Callback = function(v) _CFG.Dwell = v end
})

FarmTab:AddSlider("TargetCoinsSlider", {
    Name = "เป้าหมายเหรียญต่อรอบ (Coin Target Cap)",
    Min = 10,
    Max = 50,
    Default = 40,
    Suffix = " เหรียญ",
    Callback = function(v) _CFG.TargetCoins = v end
})

FarmTab:AddToggle("AutoResetBagToggle", {
    Name = "จัดการเมื่อกระเป๋าเต็ม (Auto Bank Coins / Candies)",
    Description = "ฆ่าตัวละครหรือวาร์ปหนีทันทีที่เก็บครบเป้าหมายเพื่อเซฟเหรียญ",
    Default = true,
    Callback = function(v) _CFG.AutoResetBag = v end
})

FarmTab:AddDropdown("BagFullActionDropdown", {
    Name = "การกระทำเมื่อกระเป๋าเต็ม (Bag Full Action)",
    Options = { "Reset Character (ฆ่าตัวตายเกิดใหม่)", "Safe Spot (วาร์ปไปจุดปลอดภัย)" },
    Default = "Reset Character (ฆ่าตัวตายเกิดใหม่)",
    Callback = function(v)
        _CFG.BagFullAction = v:find("Safe") and "SafeSpot" or "Reset"
    end
})

FarmTab:AddDropdown("SafeSpotDropdown", {
    Name = "เลือกตำแหน่งจุดปลอดภัย (Safe Spot Type)",
    Options = { "Roof (บนหลังคาแมพ)", "Underground (ใต้ดิน)", "Lobby (ล็อบบี้)" },
    Default = "Roof (บนหลังคาแมพ)",
    Callback = function(v)
        if v:find("Underground") then _CFG.SafeSpotType = "Underground"
        elseif v:find("Lobby") then _CFG.SafeSpotType = "Lobby"
        else _CFG.SafeSpotType = "Roof" end
    end
})

FarmTab:AddToggle("AlwaysActiveToggle", {
    Name = "ฟาร์มตลอดเวลาไม่สนสถานะรอบ (Always Active Override)",
    Description = "เปิดให้ลูปฟาร์มทำงานต่อเนื่องแม้ UI เกมยังไม่เริ่มรอบใหม่",
    Default = true,
    Callback = function(v) _CFG.AlwaysActiveOverride = v end
})

FarmTab:AddToggle("FleeFromMurdererToggle", {
    Name = "วาร์ปหนี Murderer อัตโนมัติ (Flee Murderer)",
    Description = "วาร์ปขึ้นหลังคาทันทีเมื่อฆาตกรเข้าใกล้เกินระยะปลอดภัย",
    Default = true,
    Callback = function(v) _CFG.FleeFromMurderer = v end
})

FarmTab:AddButton({
    Name = "วาร์ปไปเหรียญ/ลูกอมที่ใกล้ที่สุด 1 จุด (Teleport Nearest)",
    Callback = function()
        local hrp = GetHRP()
        if not hrp then return end
        local targets = getSpawnedCoinsAndCandies()
        local best, bestDist = nil, math.huge
        for _, it in ipairs(targets) do
            if it.Part and it.Part.Parent and it.Part.Transparency < 1 then
                local d = (it.Position - hrp.Position).Magnitude
                if d < bestDist then bestDist = d; best = it end
            end
        end
        if best and best.Part then
            hrp.CFrame = CFrame.new(best.Position + Vector3.new(0, 2, 0))
            if firetouchinterest then
                pcall(function() firetouchinterest(hrp, best.Part, 0); firetouchinterest(hrp, best.Part, 1) end)
            end
            Notify("Farm", "วาร์ปไปเก็บ " .. best.Kind .. " แล้ว!", "success")
        else
            Notify("Farm", "ไม่พบเหรียญหรือลูกอมในแมพ", "error")
        end
    end
})

FarmTab:AddSection("// ระบบเปิดกล่องอัตโนมัติ (CRATE OPENER)")

FarmTab:AddToggle("AutoClaimBoxesToggle", {
    Name = "สุ่มกล่องอัตโนมัติ (Mystery & Event Crates)",
    Description = "เปิดกล่องสุ่มเหรียญและกล่องกิจกรรมเทศกาลต่อเนื่องอัตโนมัติ",
    Default = false,
    Callback = function(v) _CFG.AutoClaimBoxes = v end
})

FarmTab:AddButton({
    Name = "เปิดกล่อง MysteryBox ทันที 1 กล่อง",
    Callback = function()
        task.spawn(function()
            local remotes = ReplicatedStorage:FindFirstChild("Remotes")
            local shop = remotes and remotes:FindFirstChild("Shop")
            local openCrate = shop and shop:FindFirstChild("OpenCrate")
            if openCrate then
                pcall(function() openCrate:InvokeServer("MysteryBox2", "MysteryBox", "Coins") end)
                Notify("Crate", "ส่งคำสั่งเปิด MysteryBox แล้ว", "success")
            else
                Notify("Crate", "ไม่พบ Remote เปิดกล่อง", "error")
            end
        end)
    end
})

FarmTab:AddButton({
    Name = "เปิดกล่อง Event Crate ทันที 1 กล่อง",
    Callback = function()
        task.spawn(function()
            local remotes = ReplicatedStorage:FindFirstChild("Remotes")
            local shop = remotes and remotes:FindFirstChild("Shop")
            local openCrate = shop and shop:FindFirstChild("OpenCrate")
            if openCrate then
                pcall(function() openCrate:InvokeServer("Summer2026Box", "MysteryBox", "SummerKey2026") end)
                pcall(function() openCrate:InvokeServer("Halloween2026Box", "MysteryBox", "Candies") end)
                Notify("Crate", "ส่งคำสั่งเปิด Event Crate แล้ว", "success")
            else
                Notify("Crate", "ไม่พบ Remote เปิดกล่อง", "error")
            end
        end)
    end
})

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 3: คอมแบท & เรจ (COMBAT & RAGE)
-- ──────────────────────────────────────────────────────────────────────────────
CombatTab:AddSection("ระบบฆาตกร (MURDERER COMBAT)")

local murderStatusPara = CombatTab:AddParagraph({
    Title = "สถานะบทบาทฆาตกร (Murderer Status)",
    Content = isMurderer() and "[MURDERER] พร้อมใช้งานคำสั่งสังหาร" or "[INNOCENT] คุณไม่ใช่ Murderer ในรอบนี้"
})

CombatTab:AddToggle("KillAuraToggle", {
    Name = "ออร่าฆ่าอัตโนมัติรอบตัว (Kill Aura Nearby)",
    Description = "แทงผู้เล่นที่อยู่ใกล้เคียงอัตโนมัติเมื่อคุณเป็น Murderer",
    Default = false,
    Callback = function(v) _CFG.KillAura = v end
})

CombatTab:AddSlider("KillDistSlider", {
    Name = "ระยะออร่าฆ่า (Aura Distance)",
    Min = 5,
    Max = 80,
    Default = 35,
    Suffix = " studs",
    Callback = function(v) _CFG.KillAuraDist = v end
})

CombatTab:AddSlider("KillDelaySlider", {
    Name = "ดีเลย์การแทงต่อคน (Kill Delay)",
    Min = 0.05,
    Max = 1.0,
    Default = 0.18,
    Rounding = 2,
    Suffix = "s",
    Callback = function(v) _CFG.KillAuraDelay = v end
})

local function executeKillPlayer(targetPlr)
    local char = targetPlr and targetPlr.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum or hum.Health <= 0 then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local myChar = GetCharacter()
    if not hrp or not myChar then return end

    local knife = myChar:FindFirstChildOfClass("Tool")
    if not knife or not knife.Name:lower():find("knife") then
        local bp = LocalPlayer:FindFirstChild("Backpack")
        if bp then
            for _, t in ipairs(bp:GetChildren()) do
                if t:IsA("Tool") and t.Name:lower():find("knife") then
                    pcall(function() t.Parent = myChar end)
                    knife = t
                    task.wait(0.1)
                    break
                end
            end
        end
    end
    if not knife then return end

    local origSize = hrp.Size
    pcall(function()
        hrp.Size = Vector3.new(14, 14, 14)
        hrp.CanCollide = false
        hrp.Transparency = 0.5
    end)

    pcall(function()
        local handle = knife:FindFirstChild("Handle") or knife:FindFirstChild("Blade") or knife:FindFirstChildWhichIsA("BasePart")
        if handle and firetouchinterest then
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    firetouchinterest(handle, part, 0)
                    firetouchinterest(handle, part, 1)
                end
            end
            pcall(function() knife:Activate() end)
        end
    end)

    pcall(function()
        for _, v in ipairs(ReplicatedStorage:GetDescendants()) do
            if v:IsA("RemoteEvent") and v.Name:lower():find("stab") then
                pcall(function() v:FireServer(targetPlr) end)
                pcall(function() v:FireServer(char) end)
                pcall(function() v:FireServer(hrp.Position) end)
            end
        end
    end)

    task.wait(0.04)
    pcall(function()
        hrp.Size = origSize
        hrp.Transparency = 1
    end)
end

CombatTab:AddButton({
    Name = "ฆ่าผู้เล่นทุกคนในเซิร์ฟเวอร์ (Kill All Players)",
    Description = "กวาดล้างผู้เล่นทุกคนที่ยังมีชีวิตอยู่ทันที (ต้องเป็น Murderer)",
    Primary = true,
    Callback = function()
        if not isMurderer() then
            Notify("Rage", "คุณไม่ได้เป็น Murderer ในรอบนี้!", "error")
            return
        end
        Notify("Rage", "กำลังเริ่มฆ่าผู้เล่นทุกคน...", "info")
        task.spawn(function()
            for _, plr in ipairs(Players:GetPlayers()) do
                if HUB.dead then break end
                if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChildOfClass("Humanoid") and plr.Character.Humanoid.Health > 0 then
                    executeKillPlayer(plr)
                    task.wait(_CFG.KillAuraDelay)
                end
            end
            Notify("Rage", "Kill All เสร็จสิ้น!", "success")
        end)
    end
})

CombatTab:AddButton({
    Name = "ฆ่าผู้เล่นที่ใกล้ที่สุด (Kill Closest Player)",
    Callback = function()
        if not isMurderer() then
            Notify("Rage", "คุณไม่ได้เป็น Murderer ในรอบนี้!", "error")
            return
        end
        local myPos = GetHRP() and GetHRP().Position or Vector3.zero
        local closest, dist = nil, math.huge
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hum = plr.Character:FindFirstChildOfClass("Humanoid")
                if hum and hum.Health > 0 then
                    local d = (plr.Character.HumanoidRootPart.Position - myPos).Magnitude
                    if d < dist then dist = d; closest = plr end
                end
            end
        end
        if closest then
            executeKillPlayer(closest)
            Notify("Rage", "สังหาร " .. closest.Name .. " สำเร็จ!", "success")
        else
            Notify("Rage", "ไม่พบเป้าหมายใกล้เคียง", "error")
        end
    end
})

CombatTab:AddSection("ระบบนายอำเภอ & ปืน (SHERIFF & SILENT AIM)")

CombatTab:AddButton({
    Name = "วาร์ปไปเก็บปืนที่ตกทันที (Grab Dropped Gun)",
    Description = "วาร์ปไปหยิบปืนของนายอำเภอที่ตกอยู่บนพื้นทันที",
    Primary = true,
    Callback = function()
        local gun = getGunDrop()
        local hrp = GetHRP()
        if gun and hrp then
            local pos = gun:IsA("BasePart") and gun.Position or gun:GetPivot().Position
            hrp.CFrame = CFrame.new(pos + Vector3.new(0, 2.5, 0))
            if firetouchinterest then
                local handle = gun:IsA("BasePart") and gun or gun:FindFirstChild("Handle") or gun:FindFirstChildWhichIsA("BasePart")
                if handle then
                    pcall(function() firetouchinterest(hrp, handle, 0); firetouchinterest(hrp, handle, 1) end)
                end
            end
            Notify("Gun", "วาร์ปไปเก็บปืนแล้ว!", "success")
        else
            Notify("Gun", "ไม่พบปืนที่ตกอยู่ในแมพขณะนี้", "error")
        end
    end
})

CombatTab:AddToggle("AutoGrabGunToggle", {
    Name = "ออโต้วาร์ปเก็บปืนอัตโนมัติ (Auto Grab Gun Drop)",
    Description = "ตรวจจับเมื่อนายอำเภอเสียชีวิตและวาร์ปไปเก็บปืนทันที",
    Default = false,
    Callback = function(v) _CFG.AutoGrabGun = v end
})

CombatTab:AddToggle("SilentAimToggle", {
    Name = "ยิงล็อคเป้าอัตโนมัติ (Silent Aim - Gun)",
    Description = "กระสุนพุ่งตรงเข้าหาเป้าหมายพร้อมระบบทำนายการเคลื่อนที่",
    Default = false,
    Callback = function(v) _CFG.SilentAim = v end
})

CombatTab:AddDropdown("SilentModeDropdown", {
    Name = "โหมดล็อคเป้า (Target Mode)",
    Options = { "Murderer", "FOV", "Nearest", "Chosen" },
    Default = "Murderer",
    Callback = function(v) _CFG.SilentTargetMode = v end
})

local function getPlayerNamesList()
    local names = {}
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then table.insert(names, p.Name) end
    end
    table.sort(names)
    if #names == 0 then names = { "(no players)" } end
    return names
end

local chosenPlayerDropdown = CombatTab:AddDropdown("SilentChosenDropdown", {
    Name = "เลือกผู้เล่นเฉพาะเจาะจง (Chosen Player)",
    Options = getPlayerNamesList(),
    Default = getPlayerNamesList()[1] or "",
    Callback = function(v) _CFG.SilentChosen = v end
})

CombatTab:AddButton({
    Name = "รีเฟรชรายชื่อผู้เล่น (Refresh Players)",
    Callback = function()
        chosenPlayerDropdown:SetOptions(getPlayerNamesList())
        Notify("Aim", "รีเฟรชรายชื่อผู้เล่นแล้ว", "info")
    end
})

CombatTab:AddDropdown("SilentHitPartDropdown", {
    Name = "ตำแหน่งที่เล็ง (Hit Part)",
    Options = { "Head", "HumanoidRootPart", "UpperTorso", "Random" },
    Default = "Head",
    Callback = function(v) _CFG.SilentHitPart = v end
})

CombatTab:AddSlider("SilentHitChanceSlider", {
    Name = "โอกาสยิงโดน (Hit Chance)",
    Min = 0,
    Max = 100,
    Default = 100,
    Suffix = "%",
    Callback = function(v) _CFG.SilentHitChance = v end
})

CombatTab:AddSlider("SilentFOVSlider", {
    Name = "รัศมีวงเล็ง FOV (FOV Radius)",
    Min = 10,
    Max = 600,
    Default = 150,
    Suffix = "px",
    Callback = function(v) _CFG.SilentFOV = v end
})

CombatTab:AddToggle("SilentShowFOVToggle", {
    Name = "แสดงวงกลม FOV (Show FOV Circle)",
    Default = true,
    Callback = function(v) _CFG.SilentShowFOV = v end
})

CombatTab:AddToggle("SilentWallCheckToggle", {
    Name = "เช็คสิ่งกีดขวาง (Wall / Visible Check)",
    Default = false,
    Callback = function(v) _CFG.SilentWallCheck = v end
})

CombatTab:AddToggle("SilentTeamCheckToggle", {
    Name = "เช็คทีมเดียวกัน (Team Check)",
    Default = false,
    Callback = function(v) _CFG.SilentTeamCheck = v end
})

CombatTab:AddSection("ขยายฮิตบ็อกซ์ศัตรู (HITBOX EXPANDER)")

CombatTab:AddToggle("HitboxToggle", {
    Name = "ขยายขนาดตัวศัตรู (Expand Hitbox)",
    Description = "ขยาย HumanoidRootPart ของศัตรูให้ยิงและแทงโดนง่ายขึ้น",
    Default = false,
    Callback = function(v) _CFG.HitboxEnabled = v end
})

CombatTab:AddSlider("HitboxSizeSlider", {
    Name = "ขนาดฮิตบ็อกซ์ (Hitbox Size)",
    Min = 2,
    Max = 15,
    Default = 4,
    Suffix = " studs",
    Callback = function(v) _CFG.HitboxSize = v end
})

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 4: การมองเห็น (ESP & VISUALS)
-- ──────────────────────────────────────────────────────────────────────────────
VisualsTab:AddSection("ระบบควบคุมหลัก (MASTER CONTROLS)")

VisualsTab:AddToggle("MasterESPToggle", {
    Name = "เปิดใช้งาน ESP หลัก (Master Enable)",
    Default = true,
    Callback = function(v) _CFG.MasterESP = v end
})

VisualsTab:AddToggle("PlayerESPToggle", {
    Name = "แสดงผู้เล่น (Players ESP)",
    Default = true,
    Callback = function(v) _CFG.PlayerESP = v end
})

VisualsTab:AddToggle("RoleESPToggle", {
    Name = "แสดงสีแยกตามบทบาท (Role Colors ESP)",
    Description = "แดง: Murderer | น้ำเงิน: Sheriff | เขียว: Innocent | เทา: Dead",
    Default = true,
    Callback = function(v) _CFG.RoleESP = v end
})

VisualsTab:AddToggle("CoinESPToggle", {
    Name = "ไฮไลท์เหรียญทั้งหมด (Coin ESP Highlight)",
    Default = false,
    Callback = function(v) _CFG.CoinESP = v end
})

VisualsTab:AddToggle("GunESPToggle", {
    Name = "ไฮไลท์ปืน/มีดที่ตก (Gun / Drop ESP)",
    Default = true,
    Callback = function(v) _CFG.GunESP = v end
})

VisualsTab:AddSection("กล่อง 2D (2D BOXES)")

VisualsTab:AddToggle("BoxESPToggle", {
    Name = "เปิดใช้งานกล่อง (2D Box)",
    Default = false,
    Callback = function(v) _CFG.BoxESP = v end
})

VisualsTab:AddDropdown("BoxStyleDropdown", {
    Name = "รูปแบบกล่อง (Box Style)",
    Options = { "Corner", "Full", "Radius" },
    Default = "Corner",
    Callback = function(v) _CFG.BoxStyle = v end
})

VisualsTab:AddSlider("BoxThickSlider", {
    Name = "ความหนาเส้นกล่อง (Box Thickness)",
    Min = 1,
    Max = 5,
    Default = 1,
    Callback = function(v) _CFG.BoxThickness = v end
})

VisualsTab:AddToggle("BoxFillToggle", {
    Name = "เติมสีโปร่งใสในกล่อง (Box Fill)",
    Default = false,
    Callback = function(v) _CFG.BoxFill = v end
})

VisualsTab:AddSection("ไฮไลท์ตัวละคร (CHAMS & GLOW)")

VisualsTab:AddToggle("ChamsToggle", {
    Name = "ไฮไลท์ตัวละครทะลุกำแพง (Chams / Highlight)",
    Default = false,
    Callback = function(v) _CFG.Chams = v end
})

VisualsTab:AddSlider("ChamsFillSlider", {
    Name = "ความทึบของสีด้านใน (Chams Fill)",
    Min = 0,
    Max = 100,
    Default = 45,
    Suffix = "%",
    Callback = function(v) _CFG.ChamsFill = 1 - (v / 100) end
})

VisualsTab:AddSlider("ChamsOutlineSlider", {
    Name = "ความทึบของเส้นขอบ (Chams Outline)",
    Min = 0,
    Max = 100,
    Default = 100,
    Suffix = "%",
    Callback = function(v) _CFG.ChamsOutline = 1 - (v / 100) end
})

VisualsTab:AddToggle("HaloToggle", {
    Name = "แสงออร่ารอบตัว (Halo Glow)",
    Default = false,
    Callback = function(v) _CFG.Halo = v end
})

VisualsTab:AddSection("ข้อมูลตัวละคร (PLAYER LABELS)")

VisualsTab:AddToggle("NameESPToggle", {
    Name = "แสดงชื่อผู้เล่น (Name ESP)",
    Default = false,
    Callback = function(v) _CFG.NameESP = v end
})

VisualsTab:AddToggle("DistESPToggle", {
    Name = "แสดงระยะห่าง (Distance ESP)",
    Default = false,
    Callback = function(v) _CFG.DistESP = v end
})

VisualsTab:AddToggle("HealthESPToggle", {
    Name = "แสดงหลอดเลือด (Health Bar & Number)",
    Default = false,
    Callback = function(v) _CFG.HealthESP = v end
})

VisualsTab:AddSlider("TextSizeSlider", {
    Name = "ขนาดตัวอักษร (Text Size)",
    Min = 10,
    Max = 20,
    Default = 13,
    Callback = function(v) _CFG.TextSize = v end
})

VisualsTab:AddSection("ระบบเสริม (VISUAL EXTRAS)")

VisualsTab:AddToggle("TracerToggle", {
    Name = "เส้นโยงไปยังตัวผู้เล่น (Tracers)",
    Default = false,
    Callback = function(v) _CFG.TracerESP = v end
})

VisualsTab:AddDropdown("TracerOriginDropdown", {
    Name = "จุดเริ่มต้นของเส้น Tracer (Origin)",
    Options = { "Bottom", "Center", "Top", "Mouse" },
    Default = "Bottom",
    Callback = function(v) _CFG.TracerOrigin = v end
})

VisualsTab:AddToggle("ArrowToggle", {
    Name = "ลูกศรชี้เป้านอกจอ (Off-Screen Arrows)",
    Default = false,
    Callback = function(v) _CFG.ArrowESP = v end
})

VisualsTab:AddToggle("HeadCircleToggle", {
    Name = "วงกลมรอบหัว (Head Circle)",
    Default = false,
    Callback = function(v) _CFG.HeadCircleESP = v end
})

VisualsTab:AddToggle("SkeletonToggle", {
    Name = "โครงกระดูกตัวละคร (Skeleton ESP)",
    Default = false,
    Callback = function(v) _CFG.SkeletonESP = v end
})

VisualsTab:AddToggle("RainbowToggle", {
    Name = "สีรุ้งเคลื่อนไหว (Rainbow ESP)",
    Default = false,
    Callback = function(v) _CFG.RainbowESP = v end
})

VisualsTab:AddSlider("MaxESPDistSlider", {
    Name = "ระยะแสดงผลสูงสุด (Max Distance)",
    Min = 0,
    Max = 5000,
    Default = 1000,
    Suffix = " studs",
    Callback = function(v) _CFG.MaxESPDist = v end
})

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 5: ตัวละคร & เคลื่อนที่ (MOVEMENT & PHYSICS)
-- ──────────────────────────────────────────────────────────────────────────────
CharTab:AddSection("ความเร็วและการกระโดด (SPEED & JUMP BYPASS)")

CharTab:AddToggle("WalkSpeedToggle", {
    Name = "เร่งความเร็วเดิน (WalkSpeed Bypass)",
    Description = "ระบบล็อคค่าความเร็วทุกเฟรม ป้องกัน MM2 รีเซ็ตความเร็วกลับ",
    Default = false,
    Callback = function(v)
        _CFG.WalkSpeedEnabled = v
        local hum = GetHumanoid()
        if hum then hum.WalkSpeed = v and _CFG.WalkSpeedValue or 16 end
    end
})

CharTab:AddSlider("WalkSpeedSlider", {
    Name = "ค่าความเร็วเดิน (WalkSpeed Value)",
    Min = 16,
    Max = 500,
    Default = 16,
    Callback = function(v)
        _CFG.WalkSpeedValue = v
        if _CFG.WalkSpeedEnabled then
            local hum = GetHumanoid()
            if hum then hum.WalkSpeed = v end
        end
    end
})

CharTab:AddToggle("JumpPowerToggle", {
    Name = "เพิ่มพลังกระโดด (JumpPower Bypass)",
    Default = false,
    Callback = function(v)
        _CFG.JumpPowerEnabled = v
        local hum = GetHumanoid()
        if hum then
            hum.UseJumpPower = true
            hum.JumpPower = v and _CFG.JumpPowerValue or 50
        end
    end
})

CharTab:AddSlider("JumpPowerSlider", {
    Name = "ค่าพลังกระโดด (JumpPower Value)",
    Min = 50,
    Max = 400,
    Default = 50,
    Callback = function(v)
        _CFG.JumpPowerValue = v
        if _CFG.JumpPowerEnabled then
            local hum = GetHumanoid()
            if hum then
                hum.UseJumpPower = true
                hum.JumpPower = v
            end
        end
    end
})

CharTab:AddToggle("InfJumpToggle", {
    Name = "กระโดดได้ไม่จำกัด (Infinite Jump)",
    Default = false,
    Callback = function(v) _CFG.InfiniteJump = v end
})

CharTab:AddToggle("HipHeightToggle", {
    Name = "ปรับความสูงลอยตัว (Custom Hip Height)",
    Default = false,
    Callback = function(v)
        _CFG.HipHeightEnabled = v
        local hum = GetHumanoid()
        if hum then hum.HipHeight = v and _CFG.HipHeightValue or 2 end
    end
})

CharTab:AddSlider("HipHeightSlider", {
    Name = "ค่าความสูงลอยตัว (Hip Height)",
    Min = 0,
    Max = 20,
    Default = 2,
    Callback = function(v)
        _CFG.HipHeightValue = v
        if _CFG.HipHeightEnabled then
            local hum = GetHumanoid()
            if hum then hum.HipHeight = v end
        end
    end
})

CharTab:AddToggle("GravityToggle", {
    Name = "ปรับแต่งแรงโน้มถ่วง (Custom Gravity)",
    Default = false,
    Callback = function(v)
        _CFG.GravityEnabled = v
        Workspace.Gravity = v and _CFG.GravityValue or 196.2
    end
})

CharTab:AddSlider("GravitySlider", {
    Name = "ค่าแรงโน้มถ่วง (Gravity Value)",
    Min = 0,
    Max = 400,
    Default = 196,
    Callback = function(v)
        _CFG.GravityValue = v
        if _CFG.GravityEnabled then Workspace.Gravity = v end
    end
})

CharTab:AddSection("บินและทะลุกำแพง (FLY & NOCLIP)")

CharTab:AddToggle("FlyToggle", {
    Name = "บินอิสระ (Fly [W,A,S,D, Space, Shift])",
    Default = false,
    Callback = function(v)
        _CFG.FlyEnabled = v
        local hum = GetHumanoid()
        if hum then hum.PlatformStand = v end
        if not v then
            local hrp = GetHRP()
            if hrp then hrp.AssemblyLinearVelocity = Vector3.zero end
        end
        Notify("Fly", v and "เปิดระบบบินอิสระแล้ว" or "ปิดระบบบินแล้ว", v and "success" or "info")
    end
})

CharTab:AddSlider("FlySpeedSlider", {
    Name = "ความเร็วในการบิน (Fly Speed)",
    Min = 10,
    Max = 500,
    Default = 50,
    Callback = function(v) _CFG.FlySpeed = v end
})

CharTab:AddToggle("NoclipToggle", {
    Name = "เดินทะลุกำแพง (Noclip)",
    Default = false,
    Callback = function(v) _CFG.Noclip = v end
})

CharTab:AddToggle("AntiRagdollToggle", {
    Name = "ลุกขึ้นทันทีเมื่อล้ม (Anti-Ragdoll)",
    Description = "ป้องกันอาการตัวล้มหรือกระเด็นเมื่อโดนกระสุนหรือแรงปะทะ",
    Default = false,
    Callback = function(v) _CFG.AntiRagdoll = v end
})

CharTab:AddSection("// ท่าเต้น & แอนิเมชัน (EMOTES & ANIMATIONS)")

local emoteOptions = {
    "Ninja", "Floss", "Zen", "Dab", "Sit", "Zombie",
    "Headless", "Hero", "Wave", "Point", "Laugh",
    "Dance 1", "Dance 2", "Dance 3"
}

CharTab:AddDropdown("EmoteDropdown", {
    Name = "เลือกท่าทาง / ท่าเต้น (Select Emote / Dance)",
    Options = emoteOptions,
    Default = "Ninja",
    Callback = function(v)
        local key = v:lower():gsub(" ", "")
        if key == "dance1" then key = "dance"
        elseif key == "dance2" then key = "dance2"
        elseif key == "dance3" then key = "dance3" end
        _CFG.SelectedEmote = key
    end
})

CharTab:AddButton({
    Name = "เล่นท่าทางทันที (Play Selected Emote)",
    Primary = true,
    Callback = function()
        playMM2Emote(_CFG.SelectedEmote, _CFG.EmoteSpeed, _CFG.LoopEmote, _CFG.DanceWalk)
        Notify("Emote", "กำลังเล่นท่าทาง: " .. tostring(_CFG.SelectedEmote), "success")
    end
})

CharTab:AddButton({
    Name = "หยุดท่าทาง (Stop Emote)",
    Callback = function()
        stopCurrentEmote()
        Notify("Emote", "หยุดท่าทางแล้ว", "info")
    end
})

CharTab:AddToggle("LoopEmoteToggle", {
    Name = "วนลูปท่าทางอัตโนมัติ (Loop Emote)",
    Description = "เล่นท่าเต้นวนซ้ำต่อเนื่องไม่หยุดจนกว่าจะกดหยุด",
    Default = false,
    Callback = function(v)
        _CFG.LoopEmote = v
        if v and not (activeEmoteTrack and activeEmoteTrack.IsPlaying) then
            playMM2Emote(_CFG.SelectedEmote, _CFG.EmoteSpeed, true, _CFG.DanceWalk)
        end
    end
})

CharTab:AddToggle("DanceWalkToggle", {
    Name = "เต้นขณะเดิน (Dance Walk)",
    Description = "เคลื่อนที่เดินได้โดยที่ท่าทางไม่หลุดหรือหยุดเล่น",
    Default = false,
    Callback = function(v)
        _CFG.DanceWalk = v
        if activeEmoteTrack and activeEmoteTrack.IsPlaying then
            playMM2Emote(_CFG.SelectedEmote, _CFG.EmoteSpeed, _CFG.LoopEmote, v)
        end
    end
})

CharTab:AddSlider("EmoteSpeedSlider", {
    Name = "ความเร็วท่าทาง (Emote Speed)",
    Min = 0.2,
    Max = 3.0,
    Default = 1.0,
    Rounding = 1,
    Suffix = "x",
    Callback = function(v)
        _CFG.EmoteSpeed = v
        if activeEmoteTrack and activeEmoteTrack.IsPlaying then
            pcall(function() activeEmoteTrack:AdjustSpeed(v) end)
        end
    end
})

CharTab:AddInput("CustomEmoteIdInput", {
    Name = "รหัสแอนิเมชันกำหนดเอง (Custom Animation ID)",
    Placeholder = "เช่น 3333499508 หรือ 128777973",
    Callback = function(v)
        _CFG.CustomEmoteId = v
    end
})

CharTab:AddSection("// คำสั่งตัวละคร (RESET ACTIONS)")

CharTab:AddButton({
    Name = "เกิดใหม่ทันที (Fast Respawn / Reset)",
    Primary = true,
    Callback = function()
        local hum = GetHumanoid()
        if hum then hum.Health = 0 end
        Notify("Character", "กำลังรีเซ็ตตัวละคร...", "info")
    end
})

CharTab:AddButton({
    Name = "คืนค่าสเตตัสเริ่มต้น (Reset Stats to Default)",
    Callback = function()
        local hum = GetHumanoid()
        if hum then
            hum.WalkSpeed = 16
            hum.JumpPower = 50
            hum.UseJumpPower = true
            hum.HipHeight = 2
            hum.PlatformStand = false
        end
        Workspace.Gravity = 196.2
        Notify("Character", "คืนค่าเริ่มต้นเรียบร้อย", "success")
    end
})

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 6: วาร์ป & แผนที่ (TELEPORTATION)
-- ──────────────────────────────────────────────────────────────────────────────
TeleportTab:AddSection("วาร์ปไปหาผู้เล่น (PLAYER TELEPORT)")

local playerTPDropdown = TeleportTab:AddDropdown("PlayerTPDropdown", {
    Name = "เลือกผู้เล่น (Select Player)",
    Options = getPlayerNamesList(),
    Default = getPlayerNamesList()[1] or "",
    Callback = function(v) _CFG.SelectedTPPlayer = v end
})

TeleportTab:AddButton({
    Name = "รีเฟรชรายชื่อผู้เล่น (Refresh Player List)",
    Callback = function()
        playerTPDropdown:SetOptions(getPlayerNamesList())
        Notify("Teleport", "รีเฟรชรายชื่อผู้เล่นแล้ว", "info")
    end
})

TeleportTab:AddButton({
    Name = "วาร์ปไปหาผู้เล่นที่เลือก (Teleport To Player)",
    Primary = true,
    Callback = function()
        local targetName = _CFG.SelectedTPPlayer or playerTPDropdown:Get()
        local target = Players:FindFirstChild(targetName)
        local myHRP = GetHRP()
        local tHRP = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")
        if myHRP and tHRP then
            if _CFG.TweenTP then
                local dist = (myHRP.Position - tHRP.Position).Magnitude
                local tw = TweenService:Create(myHRP, TweenInfo.new(math.clamp(dist / 80, 0.4, 1.5), Enum.EasingStyle.Quad), { CFrame = tHRP.CFrame * CFrame.new(0, 0, 3) })
                tw:Play()
            else
                myHRP.CFrame = tHRP.CFrame * CFrame.new(0, 0, 3)
            end
            Notify("Teleport", "วาร์ปไปหา " .. targetName .. " สำเร็จ!", "success")
        else
            Notify("Teleport", "ไม่พบตัวละครเป้าหมาย", "error")
        end
    end
})

TeleportTab:AddSection("จุดวาร์ปส่วนตัว (CUSTOM WAYPOINTS)")

local customWaypoints = {}
local pendingWpName = "Spot 1"

TeleportTab:AddInput("WaypointNameInput", {
    Name = "ชื่อจุดวาร์ป (Waypoint Name)",
    Default = "Spot 1",
    Placeholder = "พิมพ์ชื่อจุดวาร์ป...",
    Callback = function(v) pendingWpName = (v ~= "" and v) or "Spot 1" end
})

local function getWaypointNamesList()
    local names = {}
    for n in pairs(customWaypoints) do table.insert(names, n) end
    table.sort(names)
    if #names == 0 then names = { "(none)" } end
    return names
end

local waypointDropdown = TeleportTab:AddDropdown("WaypointDropdown", {
    Name = "จุดวาร์ปที่บันทึก (Saved Waypoints)",
    Options = getWaypointNamesList(),
    Default = getWaypointNamesList()[1] or "",
    Callback = function(v) _CFG.SelectedWaypoint = v end
})

TeleportTab:AddButton({
    Name = "บันทึกพิกัดปัจจุบัน (Save Current Position)",
    Primary = true,
    Callback = function()
        local cf = GetRootCFrame()
        if cf then
            customWaypoints[pendingWpName] = cf
            waypointDropdown:SetOptions(getWaypointNamesList())
            Notify("Waypoints", "บันทึกพิกัด '" .. pendingWpName .. "' สำเร็จ", "success")
        else
            Notify("Waypoints", "ไม่พบตัวละคร", "error")
        end
    end
})

TeleportTab:AddButton({
    Name = "วาร์ปไปยังจุดที่เลือก (Teleport to Waypoint)",
    Callback = function()
        local wpName = _CFG.SelectedWaypoint or waypointDropdown:Get()
        local cf = customWaypoints[wpName]
        local hrp = GetHRP()
        if cf and hrp then
            if _CFG.TweenTP then
                local dist = (hrp.Position - cf.Position).Magnitude
                local tw = TweenService:Create(hrp, TweenInfo.new(math.clamp(dist / 80, 0.4, 1.5), Enum.EasingStyle.Quad), { CFrame = cf })
                tw:Play()
            else
                hrp.CFrame = cf
            end
            Notify("Waypoints", "วาร์ปไป '" .. wpName .. "' สำเร็จ", "success")
        else
            Notify("Waypoints", "ไม่พบข้อมูลจุดวาร์ป", "error")
        end
    end
})

TeleportTab:AddButton({
    Name = "ลบจุดวาร์ปที่เลือก (Delete Waypoint)",
    Callback = function()
        local wpName = _CFG.SelectedWaypoint or waypointDropdown:Get()
        if customWaypoints[wpName] then
            customWaypoints[wpName] = nil
            waypointDropdown:SetOptions(getWaypointNamesList())
            Notify("Waypoints", "ลบจุด '" .. wpName .. "' สำเร็จ", "info")
        else
            Notify("Waypoints", "ไม่มีจุดที่จะลบ", "error")
        end
    end
})

TeleportTab:AddSection("จุดสำคัญในแผนที่ MM2 (MAP SPOTS)")

TeleportTab:AddToggle("ClickTPToggle", {
    Name = "คลิกเพื่อวาร์ป (Click Teleport [กด T ค้าง + คลิก])",
    Default = false,
    Callback = function(v) _CFG.ClickTP = v end
})

TeleportTab:AddToggle("TweenTPToggle", {
    Name = "วาร์ปแบบสมูท (Tween Teleport Smooth)",
    Default = false,
    Callback = function(v) _CFG.TweenTP = v end
})

local mm2MapSpots = {
    ["Lobby"]     = CFrame.new(-109.56, 137.91, -11.67),
    ["Hotel"]     = CFrame.new(76, 131, 28),
    ["Mansion"]   = CFrame.new(-97, 131, 56),
    ["Bank"]      = CFrame.new(-7, 131, -11),
    ["Factory"]   = CFrame.new(-33, 131, -102),
    ["Hospital"]  = CFrame.new(44, 131, -82),
    ["House 2"]   = CFrame.new(19, 131, -42),
    ["Workplace"] = CFrame.new(-42, 131, 56),
}

local mapSpotNames = {}
for n in pairs(mm2MapSpots) do table.insert(mapSpotNames, n) end
table.sort(mapSpotNames)

local mapSpotDropdown = TeleportTab:AddDropdown("MapSpotDropdown", {
    Name = "เลือกจุดในแผนที่ (Map Locations)",
    Options = mapSpotNames,
    Default = "Lobby",
    Callback = function(v) _CFG.SelectedMapSpot = v end
})

local function tpToCFrame(cf)
    local hrp = GetHRP()
    if not hrp or not cf then return end
    if _CFG.TweenTP then
        local dist = (hrp.Position - cf.Position).Magnitude
        local tw = TweenService:Create(hrp, TweenInfo.new(math.clamp(dist / 80, 0.4, 1.5), Enum.EasingStyle.Quad), { CFrame = cf })
        tw:Play()
    else
        hrp.CFrame = cf
    end
end

TeleportTab:AddButton({
    Name = "วาร์ปไปยังจุดที่เลือก (Teleport to Spot)",
    Primary = true,
    Callback = function()
        local spot = _CFG.SelectedMapSpot or mapSpotDropdown:Get()
        local cf = mm2MapSpots[spot]
        if cf then
            tpToCFrame(cf)
            Notify("Teleport", "วาร์ปไป '" .. spot .. "' สำเร็จ", "success")
        end
    end
})

TeleportTab:AddButton({
    Name = "วาร์ปไปล็อบบี้ทันที (Teleport to Lobby)",
    Callback = function()
        tpToCFrame(mm2MapSpots["Lobby"])
        Notify("Teleport", "วาร์ปไปล็อบบี้แล้ว", "success")
    end
})

TeleportTab:AddButton({
    Name = "วาร์ปไปโรงแรม / ธนาคาร (Hotel / Bank)",
    Callback = function()
        tpToCFrame(mm2MapSpots["Hotel"])
        Notify("Teleport", "วาร์ปไปโรงแรมแล้ว", "success")
    end
})

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 7: โลก & ร้านค้า (WORLD & SHOP)
-- ──────────────────────────────────────────────────────────────────────────────
WorldTab:AddSection("สภาพแวดล้อม & แสงสว่าง (LIGHTING & WORLD)")

local savedLighting = {
    Brightness = Lighting.Brightness,
    ClockTime = Lighting.ClockTime,
    FogEnd = Lighting.FogEnd,
    GlobalShadows = Lighting.GlobalShadows,
    Ambient = Lighting.Ambient
}

WorldTab:AddToggle("FullbrightToggle", {
    Name = "มองในที่มืดสว่างชัด (Fullbright)",
    Default = false,
    Callback = function(v)
        _CFG.Fullbright = v
        if v then
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.FogEnd = 1e9
            Lighting.GlobalShadows = false
            Lighting.Ambient = Color3.fromRGB(180, 180, 180)
        else
            Lighting.Brightness = savedLighting.Brightness
            Lighting.ClockTime = savedLighting.ClockTime
            Lighting.FogEnd = savedLighting.FogEnd
            Lighting.GlobalShadows = savedLighting.GlobalShadows
            Lighting.Ambient = savedLighting.Ambient
        end
    end
})

WorldTab:AddSlider("FOVSlider", {
    Name = "ระยะมุมมองกล้อง (Field of View - FOV)",
    Min = 30,
    Max = 120,
    Default = math.floor(Camera.FieldOfView),
    Suffix = "°",
    Callback = function(v) Camera.FieldOfView = v end
})

WorldTab:AddToggle("LockTimeToggle", {
    Name = "ล็อคเวลาแสงอาทิตย์ (Lock Clock Time)",
    Default = false,
    Callback = function(v)
        _CFG.LockTime = v
        if v then Lighting.ClockTime = _CFG.ClockTime else Lighting.ClockTime = savedLighting.ClockTime end
    end
})

WorldTab:AddSlider("ClockTimeSlider", {
    Name = "เวลาแสงอาทิตย์ (Clock Time)",
    Min = 0,
    Max = 24,
    Default = 14,
    Suffix = "h",
    Callback = function(v)
        _CFG.ClockTime = v
        if _CFG.LockTime then Lighting.ClockTime = v end
    end
})

WorldTab:AddToggle("NoFogToggle", {
    Name = "ลบหมอกควันทั้งหมด (No Fog)",
    Default = false,
    Callback = function(v)
        _CFG.NoFog = v
        if v then Lighting.FogEnd = 1e9 else Lighting.FogEnd = savedLighting.FogEnd end
    end
})

WorldTab:AddSection("ระบบร้านค้าอาวุธ (SHOP AUTOMATION)")

local shopFolder = (Workspace:FindFirstChild("Ignored") and Workspace.Ignored:FindFirstChild("Shop")) or Workspace:FindFirstChild("Shop")

local function getShopGunsList()
    local list = {}
    if not shopFolder then return { "(no shop)" } end
    for _, it in ipairs(shopFolder:GetChildren()) do
        if it.Name:match("^%[.*%]") and (it:FindFirstChild("ClickDetector") or it:FindFirstChildOfClass("ClickDetector")) then
            table.insert(list, it.Name)
        end
    end
    table.sort(list)
    if #list == 0 then return { "(no guns found)" } end
    return list
end

local shopGunsDropdown = WorldTab:AddDropdown("ShopGunsDropdown", {
    Name = "เลือกอาวุธในร้านค้า (Shop Weapon)",
    Options = getShopGunsList(),
    Default = getShopGunsList()[1] or "",
    Callback = function(v) _CFG.SelectedShopGun = v end
})

WorldTab:AddButton({
    Name = "รีเฟรชรายการอาวุธ (Refresh Shop Guns)",
    Callback = function()
        shopGunsDropdown:SetOptions(getShopGunsList())
        Notify("Shop", "รีเฟรชรายการอาวุธแล้ว", "info")
    end
})

local function buyShopWeapon(gunName)
    if not gunName or gunName:find("%(no") then
        Notify("Shop", "กรุณาเลือกอาวุธก่อน", "error")
        return
    end
    local item = shopFolder and shopFolder:FindFirstChild(gunName)
    if not item then
        Notify("Shop", "ไม่พบอาวุธ: " .. tostring(gunName), "error")
        return
    end
    local cd = item:FindFirstChildOfClass("ClickDetector") or item:FindFirstChild("ClickDetector", true)
    if not cd then
        Notify("Shop", "ไม่พบ ClickDetector", "error")
        return
    end
    local bought = false
    if type(fireclickdetector) == "function" then
        bought = pcall(function() fireclickdetector(cd) end)
    end
    if not bought then
        pcall(function()
            local part = item:FindFirstChild("Head") or item:FindFirstChildWhichIsA("BasePart")
            local hrp = GetHRP()
            if part and hrp then
                local old = hrp.CFrame
                hrp.CFrame = part.CFrame * CFrame.new(0, 2, 0)
                task.wait(0.15)
                if fireclickdetector then fireclickdetector(cd) end
                task.wait(0.15)
                hrp.CFrame = old
                bought = true
            end
        end)
    end
    Notify("Shop", bought and ("ซื้อ " .. gunName .. " สำเร็จ!") or ("ซื้อไม่สำเร็จ"), bought and "success" or "error")
end

WorldTab:AddButton({
    Name = "ซื้ออาวุธที่เลือก (Buy Selected Weapon)",
    Primary = true,
    Callback = function()
        buyShopWeapon(_CFG.SelectedShopGun or shopGunsDropdown:Get())
    end
})

WorldTab:AddToggle("AutoBuyLowAmmoToggle", {
    Name = "ซื้ออาวุธอัตโนมัติเมื่อไม่มีปืน (Auto Buy on Missing)",
    Default = false,
    Callback = function(v) _CFG.AutoBuyLowAmmo = v end
})

WorldTab:AddToggle("AutoBuyAmmoToggle", {
    Name = "ซื้อกระสุนพร้อมกัน (Include Ammo)",
    Default = false,
    Callback = function(v) _CFG.AutoBuyAmmo = v end
})

local quickWeapons = { "[Revolver] - $1339", "[Glock] - $338", "[Shotgun] - $1250", "[AK47] - $2250" }
for _, q in ipairs(quickWeapons) do
    WorldTab:AddButton({
        Name = "ซื้อ " .. (q:match("^%[[^%]]+%]") or q),
        Callback = function() buyShopWeapon(q) end
    })
end

-- ──────────────────────────────────────────────────────────────────────────────
-- TAB 8: ระบบ & ตั้งค่า (SYSTEM & MISC)
-- ──────────────────────────────────────────────────────────────────────────────

SystemTab:AddSection("// ระบบจัดการบันทึกและโหลดการตั้งค่า (CONFIG SYSTEM)")

local configSlots = { "Default", "Legit", "Rage", "Farming", "Custom 1", "Custom 2" }

SystemTab:AddDropdown("ConfigSlotDropdown", {
    Name = "เลือกโปรไฟล์ Config (Select Config Profile)",
    Options = configSlots,
    Default = "Default",
    Callback = function(v)
        _CFG.SelectedConfigSlot = v
    end
})

SystemTab:AddToggle("AutoLoadConfigToggle", {
    Name = "โหลด Config อัตโนมัติเมื่อเปิดสคริปต์ (Auto Load on Launch)",
    Description = "โหลดโปรไฟล์ Default อัตโนมัติทันทีที่รันสคริปต์ ไม่ต้องตั้งค่าใหม่ทุกครั้ง",
    Default = true,
    Callback = function(v) _CFG.AutoLoadConfig = v end
})

SystemTab:AddToggle("AutoSaveConfigToggle", {
    Name = "บันทึก Config อัตโนมัติ (Auto Save Config)",
    Description = "บันทึกการตั้งค่าลงโปรไฟล์ที่เลือกอัตโนมัติทุก 90 วินาทีและก่อนปิดสคริปต์",
    Default = true,
    Callback = function(v) _CFG.AutoSaveConfig = v end
})

SystemTab:AddButton({
    Name = "บันทึก Config ปัจจุบัน (Save Config)",
    Primary = true,
    Callback = function()
        SaveConfig(_CFG.SelectedConfigSlot or "Default")
    end
})

SystemTab:AddButton({
    Name = "โหลด Config จากโปรไฟล์ (Load Config)",
    Callback = function()
        LoadConfig(_CFG.SelectedConfigSlot or "Default")
    end
})

SystemTab:AddButton({
    Name = "รีเซ็ต Config ทั้งหมดเป็นค่าเริ่มต้น (Reset Default)",
    Callback = function()
        ResetDefaultConfig()
    end
})

SystemTab:AddSection("การควบคุมเซิร์ฟเวอร์ (SERVER CONTROLS)")

SystemTab:AddButton({
    Name = "เข้าเซิร์ฟเวอร์เดิมใหม่ (Rejoin Server)",
    Primary = true,
    Callback = function()
        Notify("Server", "กำลัง Rejoin...", "info")
        TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
    end
})

SystemTab:AddSlider("HopMaxPlayersSlider", {
    Name = "จำนวนผู้เล่นสูงสุดที่จะย้ายไป (Hop Max Players)",
    Min = 5,
    Max = 28,
    Default = 12,
    Suffix = " คน",
    Callback = function(v) _CFG.HopMaxPlayers = v end
})

local function fetchPublicServers()
    local url = string.format("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100", game.PlaceId)
    local raw = nil
    local ok, res = pcall(function() return game:HttpGet(url) end)
    if ok and type(res) == "string" and #res > 10 then
        raw = res
    elseif typeof(request) == "function" then
        local r = request({ Url = url, Method = "GET" })
        if r and r.Body and r.StatusCode == 200 then raw = r.Body end
    end
    if raw then
        return HttpService:JSONDecode(raw)
    end
    return nil
end

SystemTab:AddButton({
    Name = "ย้ายไปเซิร์ฟเวอร์คนน้อย (Hop to Low Player Server)",
    Callback = function()
        Notify("Server", "กำลังค้นหาเซิร์ฟเวอร์คนน้อย...", "info")
        task.spawn(function()
            local data = fetchPublicServers()
            if data and data.data then
                local best = nil
                for _, s in ipairs(data.data) do
                    if type(s.playing) == "number" and s.playing < _CFG.HopMaxPlayers and s.id ~= game.JobId then
                        if not best or s.playing < best.playing then best = s end
                    end
                end
                if best then
                    TeleportService:TeleportToPlaceInstance(game.PlaceId, best.id, LocalPlayer)
                    return
                end
            end
            TeleportService:Teleport(game.PlaceId, LocalPlayer)
        end)
    end
})

SystemTab:AddButton({
    Name = "สลับเซิร์ฟเวอร์สุ่ม (Server Hop)",
    Callback = function()
        Notify("Server", "กำลังค้นหาเซิร์ฟเวอร์ใหม่...", "info")
        task.spawn(function()
            local data = fetchPublicServers()
            if data and data.data then
                for _, s in ipairs(data.data) do
                    if type(s.playing) == "number" and s.playing < s.maxPlayers and s.id ~= game.JobId then
                        TeleportService:TeleportToPlaceInstance(game.PlaceId, s.id, LocalPlayer)
                        return
                    end
                end
            end
            TeleportService:Teleport(game.PlaceId, LocalPlayer)
        end)
    end
})

SystemTab:AddButton({
    Name = "คัดลอก Job ID เซิร์ฟเวอร์ (Copy Job ID)",
    Callback = function()
        local ok = pcall(function() (setclipboard or toclipboard or writeclipboard)(game.JobId) end)
        Notify("Server", ok and "คัดลอก Job ID ลงคลิปบอร์ดแล้ว" or "อุปกรณ์ไม่รองรับ Clipboard", ok and "success" or "error")
    end
})

SystemTab:AddSection("ประสิทธิภาพ & การป้องกัน (PERFORMANCE & DEFENSE)")

SystemTab:AddToggle("AntiAFKToggle", {
    Name = "ป้องกันหลุดจากเกม 24/7 (Anti-AFK 3-Layer Bypass)",
    Description = "ป้องกันการโดนเตะออกจากเกมเมื่อไม่ขยับ (Idle 20 นาที) ด้วยระบบ 3 ชั้น",
    Default = true,
    Callback = function(v)
        _CFG.AntiAFK = v
        if v then
            pcall(function()
                if typeof(getconnections) == "function" then
                    for _, conn in ipairs(getconnections(LocalPlayer.Idled)) do
                        if conn.Disable then conn:Disable() elseif conn.Disconnect then conn:Disconnect() end
                    end
                end
            end)
        end
    end
})

local originalLightingSettings = {}
local lightingSet = false

SystemTab:AddToggle("BoostFPSToggle", {
    Name = "เพิ่ม FPS ปิดเงาและลดแสง (Boost FPS)",
    Default = false,
    Callback = function(v)
        _CFG.BoostFPS = v
        if v then
            if not lightingSet then
                originalLightingSettings.GlobalShadows = Lighting.GlobalShadows
                originalLightingSettings.FogEnd = Lighting.FogEnd
                lightingSet = true
            end
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 9e9
            pcall(function() if setfpscap then setfpscap(30) end end)
        else
            if lightingSet then
                Lighting.GlobalShadows = originalLightingSettings.GlobalShadows
                Lighting.FogEnd = originalLightingSettings.FogEnd
            end
            pcall(function() if setfpscap then setfpscap(60) end end)
        end
    end
})

local blackScreenGui = nil
SystemTab:AddToggle("BlackScreenToggle", {
    Name = "โหมดจอดำประหยัดพลังงาน (Black Screen 3D Disabled)",
    Description = "ปิดการเรนเดอร์ 3D เพื่อประหยัด CPU/GPU และแบตเตอรี่สูงสุด",
    Default = false,
    Callback = function(v)
        _CFG.BlackScreen = v
        if v then
            if not blackScreenGui then
                blackScreenGui = Instance.new("ScreenGui")
                blackScreenGui.Name = "BlackScreenPayomboyZ"
                blackScreenGui.IgnoreGuiInset = true
                blackScreenGui.ResetOnSpawn = false
                blackScreenGui.DisplayOrder = 9999
                local frame = Instance.new("Frame")
                frame.BackgroundColor3 = Color3.new(0, 0, 0)
                frame.Size = UDim2.fromScale(1, 1)
                frame.Parent = blackScreenGui
                pcall(function() blackScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui", 3) end)
            end
            blackScreenGui.Enabled = true
            pcall(function() RunService:Set3dRenderingEnabled(false) end)
        else
            if blackScreenGui then blackScreenGui.Enabled = false end
            pcall(function() RunService:Set3dRenderingEnabled(true) end)
        end
    end
})

local setFpsFn = setfpscap or (getfenv and getfenv().setfpscap)
if type(setFpsFn) == "function" then
    SystemTab:AddToggle("UnlockFPSToggle", {
        Name = "ปลดล็อค FPS ไม่จำกัด (Unlock FPS Unlimited)",
        Default = false,
        Callback = function(v)
            _CFG.UnlockFPS = v
            pcall(setFpsFn, v and 0 or _CFG.FPSCap)
            Notify("FPS", v and "ปลดล็อค FPS ไม่จำกัดแล้ว" or ("ล็อคที่ " .. _CFG.FPSCap), "info")
        end
    })

    SystemTab:AddSlider("FPSCapSlider", {
        Name = "จำกัดค่า FPS (FPS Cap)",
        Min = 30,
        Max = 360,
        Default = 60,
        Callback = function(v)
            _CFG.FPSCap = v
            if not _CFG.UnlockFPS then pcall(setFpsFn, v) end
        end
    })
end

SystemTab:AddSection("ระบบแจ้งเตือน DISCORD WEBHOOK")

SystemTab:AddInput("WebhookURLInput", {
    Name = "Discord Webhook URL",
    Default = "",
    Placeholder = "วาง Webhook URL ที่นี่...",
    Callback = function(v) _CFG.WebhookURL = v end
})

local function sendDiscordWebhook(msg)
    if _CFG.WebhookURL and _CFG.WebhookURL ~= "" then
        pcall(function()
            local req = (syn and syn.request) or (http and http.request) or http_request or (fluxus and fluxus.request) or request
            if req then
                req({
                    Url = _CFG.WebhookURL,
                    Method = "POST",
                    Headers = { ["Content-Type"] = "application/json" },
                    Body = HttpService:JSONEncode({
                        ["content"] = "",
                        ["embeds"] = {{
                            ["title"] = "PayomboyZ MM2 • Notification",
                            ["description"] = msg,
                            ["color"] = 0xF51C30
                        }}
                    })
                })
            end
        end)
    end
end

SystemTab:AddButton({
    Name = "ทดสอบส่งการแจ้งเตือน (Test Send Webhook)",
    Callback = function()
        sendDiscordWebhook("[PAYOMBOYZ] ทดสอบการเชื่อมต่อ Webhook สำเร็จ!")
        Notify("Webhook", "ส่งการแจ้งเตือนทดสอบแล้ว", "success")
    end
})

SystemTab:AddSection("การควบคุมเมนู & ปิดสคริปต์")

SystemTab:AddParagraph({
    Title = "ปุ่มคีย์ลัด (Hotkeys)",
    Content = "กด [RightControl] หรือ [K] เพื่อ ซ่อน/แสดง หน้าต่างเมนู\nกด [F] เพื่อ ปรับขนาด UI (100% / 85%)"
})

SystemTab:AddButton({
    Name = "หยุดการทำงานและปิดสคริปต์ (Unload Script & Cleanup)",
    Primary = true,
    Callback = function()
        if _G.MM2Cleanup then pcall(_G.MM2Cleanup) end
        Notify("System", "ปิดการทำงานและคืนค่าระบบเรียบร้อย", "success")
    end
})

-- Default active tab
SelectTab("หน้าหลัก")

-- ==============================================================================
-- [[ BACKGROUND THREADS & HOOKS (THE CORE BACKEND) ]]
-- ==============================================================================

-- ==============================================================================
-- [[ DEEP MM2 MAP STRUCTURE & ENTITY SCANNER ]]
-- ==============================================================================

local KNOWN_MM2_MAPS = {
    "BioLab", "Factory", "Hospital3", "Hotel", "House2", "Mansion2",
    "MilBase", "Office2", "Office3", "PoliceStation", "ResearchFacility", "Workplace", "HauntedHouse2"
}

local function getActiveMap()
    local normal = Workspace:FindFirstChild("Normal")
    if normal then
        for _, c in ipairs(normal:GetChildren()) do
            if c:IsA("Model") and (c:FindFirstChild("CoinContainer") or c:FindFirstChild("Spawns") or c:FindFirstChild("Geometry")) then
                return c
            end
        end
        return normal
    end
    local map = Workspace:FindFirstChild("Map") or Workspace:FindFirstChild("CurrentMap")
    if map then return map end

    for _, c in ipairs(Workspace:GetChildren()) do
        if c:IsA("Model") and c.Name ~= "Lobby" and c.Name ~= "EffectLoader" and c.Name ~= "ServerStatus" then
            if not Players:GetPlayerFromCharacter(c) then
                if c:FindFirstChild("CoinContainer") or c:FindFirstChild("Spawns") or c:FindFirstChild("Geometry") or c:FindFirstChild("Interactive") then
                    return c
                end
                for _, knownName in ipairs(KNOWN_MM2_MAPS) do
                    if c.Name:lower():find(knownName:lower()) then return c end
                end
            end
        end
    end
    return nil
end

local function getCoinContainer()
    local map = getActiveMap()
    if map then
        local cc = map:FindFirstChild("CoinContainer") or map:FindFirstChild("Coin_Container") or map:FindFirstChild("Coins")
        if cc then return cc end
    end
    for _, c in ipairs(Workspace:GetChildren()) do
        if c:IsA("Model") and c.Name ~= "Lobby" and not Players:GetPlayerFromCharacter(c) then
            local cc = c:FindFirstChild("CoinContainer") or c:FindFirstChild("Coin_Container") or c:FindFirstChild("Coins")
            if cc then return cc end
        end
    end
    return Workspace:FindFirstChild("CoinContainer")
        or (Workspace:FindFirstChild("Normal") and Workspace.Normal:FindFirstChild("CoinContainer"))
        or Workspace:FindFirstChild("Coin_Container")
end

local function getSpawnedCoinsAndCandies()
    local list = {}
    local scanned = {}

    local function scan(folder)
        if not folder then return end
        for _, v in ipairs(folder:GetDescendants()) do
            if v:IsA("BasePart") and v.Transparency < 1 and not scanned[v] then
                local nm = v.Name:lower()
                local isCoin = (nm == "coin" or nm == "coin_server" or nm:find("coin"))
                local isCandy = (nm == "candy" or nm == "candycorn" or nm == "pumpkin" or nm:find("candy")
                    or (v.Parent and v.Parent.Name:lower():find("candy")))

                if isCoin or isCandy then
                    scanned[v] = true
                    table.insert(list, {
                        Part = v,
                        Position = v.Position,
                        Kind = isCandy and "Candy" or "Coin"
                    })
                end
            end
        end
    end

    local cc = getCoinContainer()
    if cc then scan(cc) end
    local map = getActiveMap()
    if map then scan(map) end

    if #list == 0 then
        for _, v in ipairs(Workspace:GetDescendants()) do
            if v:IsA("BasePart") and v.Transparency < 1 and not scanned[v] then
                local nm = v.Name:lower()
                local isCoin = (nm == "coin" or nm == "coin_server" or nm:find("coin"))
                local isCandy = (nm == "candy" or nm == "candycorn" or nm == "pumpkin" or nm:find("candy")
                    or (v.Parent and v.Parent.Name:lower():find("candy")))

                if isCoin or isCandy then
                    scanned[v] = true
                    table.insert(list, {
                        Part = v,
                        Position = v.Position,
                        Kind = isCandy and "Candy" or "Coin"
                    })
                end
            end
        end
    end
    return list
end

local function getTraps()
    local traps = {}
    local map = getActiveMap()
    local search = map or Workspace
    for _, it in ipairs(search:GetDescendants()) do
        if it:IsA("Model") and (it.Name:lower():find("trap") or it:FindFirstChild("TrapScript") or it:FindFirstChild("Hitbox")) then
            table.insert(traps, it)
        end
    end
    return traps
end

local safePlatformPart = nil
local function ensureSafePlatform(targetPos)
    if not safePlatformPart or not safePlatformPart.Parent then
        safePlatformPart = Instance.new("Part")
        safePlatformPart.Name = "PayomboyZ_SafePlatform"
        safePlatformPart.Size = Vector3.new(20, 2, 20)
        safePlatformPart.Anchored = true
        safePlatformPart.CanCollide = true
        safePlatformPart.Transparency = 0.5
        safePlatformPart.Material = Enum.Material.ForceField
        safePlatformPart.Color = Color3.fromRGB(245, 28, 48)
        safePlatformPart.Parent = Workspace
    end
    safePlatformPart.CFrame = CFrame.new(targetPos - Vector3.new(0, 2, 0))
    return safePlatformPart
end

local function getSafestMapSpot(mode)
    mode = mode or _CFG.SafeSpotType or "Roof"
    local map = getActiveMap()
    local basePos = map and map:GetPivot().Position or Vector3.new(0, 10, 0)
    local targetPos = basePos
    if mode == "Underground" then
        targetPos = basePos - Vector3.new(0, 50, 0)
    elseif mode == "Lobby" then
        local lobby = Workspace:FindFirstChild("Lobby")
        targetPos = lobby and lobby:GetPivot().Position + Vector3.new(0, 10, 0) or Vector3.new(0, 50, 0)
    else
        targetPos = basePos + Vector3.new(0, 75, 0)
    end
    ensureSafePlatform(targetPos)
    return targetPos
end

local function neutralizeAllTraps()
    local traps = getTraps()
    for _, t in ipairs(traps) do
        pcall(function()
            for _, p in ipairs(t:GetDescendants()) do
                if p:IsA("BasePart") then
                    p.CanCollide = false
                    p.CanTouch = false
                end
            end
        end)
    end
    return #traps
end

local function detonateTrapsSafely()
    local traps = getTraps()
    local hrp = GetHRP()
    if not hrp then return 0 end
    local count = 0
    for _, t in ipairs(traps) do
        local hitbox = t:FindFirstChild("Hitbox") or t:FindFirstChildWhichIsA("BasePart")
        if hitbox and firetouchinterest then
            pcall(function()
                firetouchinterest(hrp, hitbox, 0)
                task.wait(0.01)
                firetouchinterest(hrp, hitbox, 1)
                count = count + 1
            end)
        end
    end
    return count
end

local mm2EmoteAnimations = {
    ninja     = "rbxassetid://3333499508",
    dab       = "rbxassetid://3333136415",
    floss     = "rbxassetid://3333272771",
    zen       = "rbxassetid://3333136415",
    sit       = "rbxassetid://2506281708",
    zombie    = "rbxassetid://3333136415",
    hero      = "rbxassetid://3333499508",
    zombierun = "rbxassetid://3333499508",
    dance     = "rbxassetid://182435998",
    dance2    = "rbxassetid://182436842",
    dance3    = "rbxassetid://182436935",
    cheer     = "rbxassetid://180436048",
    wave      = "rbxassetid://128777973",
    point     = "rbxassetid://128853357",
    laugh     = "rbxassetid://129423131"
}

local activeEmoteTrack = nil
local danceWalkConn = nil

local function stopCurrentEmote()
    if danceWalkConn then
        pcall(function() danceWalkConn:Disconnect() end)
        danceWalkConn = nil
    end
    if activeEmoteTrack then
        pcall(function() activeEmoteTrack:Stop(0.15) end)
        activeEmoteTrack = nil
    end
end

local function playMM2Emote(emoteName, speed, looped, isDanceWalk)
    emoteName = emoteName or _CFG.SelectedEmote or "ninja"
    speed = speed or _CFG.EmoteSpeed or 1.0
    looped = (looped ~= nil and looped) or _CFG.LoopEmote or false
    isDanceWalk = (isDanceWalk ~= nil and isDanceWalk) or _CFG.DanceWalk or false

    pcall(function()
        local remotes = ReplicatedStorage:FindFirstChild("Remotes")
        local misc = remotes and remotes:FindFirstChild("Misc")
        local playEmote = misc and misc:FindFirstChild("PlayEmote")
        if playEmote and playEmote:IsA("BindableEvent") then
            playEmote:Fire(emoteName)
        end
    end)

    stopCurrentEmote()
    local char = GetCharacter()
    local hum = GetHumanoid()
    if not char or not hum then return false end

    local animId = nil
    if _CFG.CustomEmoteId and _CFG.CustomEmoteId ~= "" then
        local rawId = tostring(_CFG.CustomEmoteId):gsub("%D", "")
        if #rawId > 3 then animId = "rbxassetid://" .. rawId end
    end
    if not animId then
        animId = mm2EmoteAnimations[emoteName] or "rbxassetid://3333499508"
    end

    pcall(function()
        local anim = Instance.new("Animation")
        anim.AnimationId = animId
        local animator = hum:FindFirstChildOfClass("Animator") or hum
        local track = animator:LoadAnimation(anim)
        track.Priority = Enum.AnimationPriority.Action4
        track.Looped = looped or false
        track:Play(0.12)
        track:AdjustSpeed(speed)
        activeEmoteTrack = track

        if isDanceWalk then
            danceWalkConn = hum:GetPropertyChangedSignal("MoveDirection"):Connect(function()
                if activeEmoteTrack and not activeEmoteTrack.IsPlaying then
                    pcall(function()
                        activeEmoteTrack:Play(0.1)
                        activeEmoteTrack:AdjustSpeed(speed)
                    end)
                end
            end)
        end
    end)
    return true
end

local lastGunFireTick = 0
local GUN_RELOAD_COOLDOWN = 1.35

local function shootGunAt(targetPos)
    local char = LocalPlayer.Character
    local gun = char and (char:FindFirstChild("Gun") or char:FindFirstChildWhichIsA("Tool"))
    if not gun or not gun.Name:lower():find("gun") then
        local bp = LocalPlayer:FindFirstChild("Backpack")
        if bp then
            local bpGun = bp:FindFirstChild("Gun") or bp:FindFirstChildWhichIsA("Tool")
            if bpGun and bpGun.Name:lower():find("gun") then
                gun = bpGun
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if hum then hum:EquipTool(gun) else gun.Parent = char end
                task.wait(0.1)
            end
        end
    end
    if not gun then return false, "No gun found" end

    local myHRP = GetHRP()
    if not myHRP then return false, "No HRP" end

    local now = tick()
    if (now - lastGunFireTick) < GUN_RELOAD_COOLDOWN then
        return false, "Gun reloading"
    end

    local finalTarget = targetPos
    local murd = getMurdererPlayer()
    if murd and murd.Character and murd.Character:FindFirstChild("HumanoidRootPart") then
        local murdHRP = murd.Character.HumanoidRootPart
        local dist = (murdHRP.Position - myHRP.Position).Magnitude
        local bulletTime = dist / 1850
        local rawVel = murdHRP.AssemblyLinearVelocity or murdHRP.Velocity
        local vel = (typeof(rawVel) == "Vector3" and rawVel) or Vector3.zero
        finalTarget = targetPos + (vel * bulletTime)
    end

    local gunAttachment = char:FindFirstChild("HumanoidRootPart") and char.HumanoidRootPart:FindFirstChild("GunRaycastAttachment")
    local originCFrame = (gunAttachment and gunAttachment.WorldCFrame) or myHRP.CFrame

    pcall(function()
        myHRP.CFrame = CFrame.new(myHRP.Position, Vector3.new(finalTarget.X, myHRP.Position.Y, finalTarget.Z))
        if Camera then Camera.CFrame = CFrame.new(Camera.CFrame.Position, finalTarget) end
    end)

    local shootRemote = gun:FindFirstChild("Shoot")
    if shootRemote and shootRemote:IsA("RemoteEvent") then
        pcall(function() shootRemote:FireServer(originCFrame, finalTarget) end)
    end

    pcall(function()
        local ws = ReplicatedStorage:FindFirstChild("ClientServices") and ReplicatedStorage.ClientServices:FindFirstChild("WeaponService")
        local gf = ws and ws:FindFirstChild("GunFired")
        if gf and gf:IsA("RemoteEvent") then
            gf:FireServer(originCFrame, finalTarget)
        end
    end)

    pcall(function() gun:Activate() end)
    lastGunFireTick = tick()
    return true
end

local function openMM2Crate(crateName, currency)
    crateName = crateName or _CFG.SelectedCrate or "MysteryBox"
    currency = currency or _CFG.SelectedCrateCurrency or "Coins"
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    local shop = remotes and remotes:FindFirstChild("Shop")
    local openRemote = shop and shop:FindFirstChild("OpenCrate")
    if openRemote and openRemote:IsA("RemoteFunction") then
        pcall(function() openRemote:InvokeServer(crateName, "MysteryBox", currency) end)
        return true
    end
    for _, r in ipairs(ReplicatedStorage:GetDescendants()) do
        if (r:IsA("RemoteFunction") or r:IsA("RemoteEvent")) and (r.Name:lower() == "opencrate" or r.Name:lower():find("crate")) then
            pcall(function()
                if r:IsA("RemoteFunction") then r:InvokeServer(crateName, currency) else r:FireServer(crateName, currency) end
            end)
            return true
        end
    end
    return false
end

local function claimAllEventPassTiers()
    local remotes = {}
    for _, r in ipairs(ReplicatedStorage:GetDescendants()) do
        if r:IsA("RemoteFunction") or r:IsA("RemoteEvent") then
            local nm = r.Name:lower()
            if nm:find("claim") and (nm:find("pass") or nm:find("tier") or nm:find("battle") or nm:find("reward")) then
                table.insert(remotes, r)
            end
        end
    end
    local count = 0
    for _, r in ipairs(remotes) do
        for tier = 1, 50 do
            pcall(function()
                if r:IsA("RemoteFunction") then
                    r:InvokeServer(tier)
                else
                    r:FireServer(tier)
                end
                count = count + 1
            end)
        end
    end
    return count
end

local function checkBagFull()
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return false end
    local mainGUI = pg:FindFirstChild("MainGUI")
    local gameUI = mainGUI and mainGUI:FindFirstChild("Game")
    local coinBags = gameUI and gameUI:FindFirstChild("CoinBags")
    local container = coinBags and coinBags:FindFirstChild("Container")
    local coin = container and container:FindFirstChild("Coin")
    if coin then
        local fullLabel = coin:FindFirstChild("Full")
        if fullLabel and fullLabel.Visible then return true end
        local currencyFrame = coin:FindFirstChild("CurrencyFrame")
        local icon = currencyFrame and currencyFrame:FindFirstChild("Icon")
        local coinsText = icon and icon:FindFirstChild("Coins")
        if coinsText and coinsText:IsA("TextLabel") then
            local num = tonumber(string.match(coinsText.Text, "%d+"))
            if num and num >= (_CFG.TargetCoins or 40) then return true end
        end
    end
    return false
end

local function checkCandyBagFull()
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return false end
    local mainGUI = pg:FindFirstChild("MainGUI")
    local gameUI = mainGUI and mainGUI:FindFirstChild("Game")
    local coinBags = gameUI and gameUI:FindFirstChild("CoinBags")
    local container = coinBags and coinBags:FindFirstChild("Container")
    local candy = container and (container:FindFirstChild("Candy") or container:FindFirstChild("Candies") or container:FindFirstChild("Event"))
    if candy then
        local fullLabel = candy:FindFirstChild("Full")
        if fullLabel and fullLabel.Visible then return true end
        local currencyFrame = candy:FindFirstChild("CurrencyFrame")
        local icon = currencyFrame and currencyFrame:FindFirstChild("Icon")
        local countText = icon and (icon:FindFirstChild("Coins") or icon:FindFirstChild("Candies") or icon:FindFirstChild("Count"))
        if countText and countText:IsA("TextLabel") then
            local num = tonumber(string.match(countText.Text, "%d+"))
            if num and num >= (_CFG.TargetCandies or 40) then return true end
        end
    end
    return false
end

local function resetCharacter()
    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.Health = 0 end
        char:BreakJoints()
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = CFrame.new(0, -50000, 0) end
    end
end

local function checkBagFullAction()
    if _CFG.AutoResetBag and (checkBagFull() or checkCandyBagFull()) then
        if (_CFG.BagFullAction or "Reset") == "SafeSpot" then
            local hrp = GetHRP()
            if hrp then
                local safePos = getSafestMapSpot(_CFG.SafeSpotType)
                hrp.CFrame = CFrame.new(safePos)
                hrp.AssemblyLinearVelocity = Vector3.zero
            end
            Notify("Farm", "กระเป๋าเต็มแล้ว! วาร์ปไปจุดปลอดภัยแล้ว", "info")
        else
            Notify("Farm", "กระเป๋าเต็มแล้ว! กำลังรีเซ็ตตัวละคร...", "info")
            resetCharacter()
        end
        return true
    end
    return false
end

local function checkCandyBagFullAction()
    return checkBagFullAction()
end

-- 1. Bulletproof 24/7 Anti-AFK Engine (Triple-Layer Bypass)
do
    -- Layer 1: Disable Roblox's internal Idle signal handlers via getconnections
    pcall(function()
        if typeof(getconnections) == "function" then
            for _, conn in ipairs(getconnections(LocalPlayer.Idled)) do
                if conn.Disable then
                    conn:Disable()
                elseif conn.Disconnect then
                    conn:Disconnect()
                end
            end
        end
    end)

    -- Layer 2: Realtime Idled Listener with dual VirtualUser & VirtualInputManager
    track(LocalPlayer.Idled:Connect(function()
        if not _CFG.AntiAFK then return end
        pcall(function()
            local vu = game:GetService("VirtualUser")
            if vu then
                vu:CaptureController()
                vu:ClickButton2(Vector2.zero)
                vu:Button2Down(Vector2.zero, Camera.CFrame)
                task.wait(0.08)
                vu:Button2Up(Vector2.zero, Camera.CFrame)
            end
            local vim = game:GetService("VirtualInputManager")
            if vim then
                vim:SendKeyEvent(true, Enum.KeyCode.Unknown, false, nil)
                task.wait(0.04)
                vim:SendKeyEvent(false, Enum.KeyCode.Unknown, false, nil)
            end
        end)
    end))

    -- Layer 3: Active Periodic Keepalive Pulse (every 150 seconds / 2.5 minutes)
    -- Protects even when the game window is completely minimized or unfocused
    task.spawn(function()
        while not HUB.dead do
            task.wait(150)
            if _CFG.AntiAFK then
                pcall(function()
                    local vu = game:GetService("VirtualUser")
                    if vu then
                        vu:CaptureController()
                        vu:ClickButton2(Vector2.new(5, 5))
                    end
                    local vim = game:GetService("VirtualInputManager")
                    if vim then
                        vim:SendKeyEvent(true, Enum.KeyCode.Unknown, false, nil)
                        task.wait(0.03)
                        vim:SendKeyEvent(false, Enum.KeyCode.Unknown, false, nil)
                    end
                end)
            end
        end
    end)
end

-- 2. Movement & Physics Bypass Loop (RenderStepped & Heartbeat)
local movementConn = RunService.RenderStepped:Connect(function()
    if HUB.dead then return end
    local hum = GetHumanoid()
    if hum then
        if _CFG.WalkSpeedEnabled and hum.WalkSpeed ~= _CFG.WalkSpeedValue then
            hum.WalkSpeed = _CFG.WalkSpeedValue
        end
        if _CFG.JumpPowerEnabled then
            if not hum.UseJumpPower then hum.UseJumpPower = true end
            if hum.JumpPower ~= _CFG.JumpPowerValue then hum.JumpPower = _CFG.JumpPowerValue end
        end
        if _CFG.HipHeightEnabled and hum.HipHeight ~= _CFG.HipHeightValue then
            hum.HipHeight = _CFG.HipHeightValue
        end
    end
    if _CFG.GravityEnabled and Workspace.Gravity ~= _CFG.GravityValue then
        Workspace.Gravity = _CFG.GravityValue
    end
end)
track(movementConn)

-- Respawn gravity sync
track(LocalPlayer.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid", 10)
    task.wait(0.3)
    if HUB.dead then return end
    if _CFG.GravityEnabled then Workspace.Gravity = _CFG.GravityValue end
end))

-- Infinite Jump
track(UserInputService.JumpRequest:Connect(function()
    if HUB.dead or not _CFG.InfiniteJump then return end
    local hum = GetHumanoid()
    if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
end))

-- Anti-Ragdoll Loop
track(RunService.Heartbeat:Connect(function()
    if HUB.dead or not _CFG.AntiRagdoll then return end
    local hum = GetHumanoid()
    if hum and hum:GetState() == Enum.HumanoidStateType.Physics then
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
    end
end))

-- Fly Loop
local flyConn = RunService.RenderStepped:Connect(function()
    if HUB.dead or not _CFG.FlyEnabled then return end
    local hum = GetHumanoid()
    local root = GetHRP()
    if not hum or not root then return end
    hum.PlatformStand = true

    local dir = Vector3.zero
    local cf = Camera.CFrame
    if UserInputService:IsKeyDown(Enum.KeyCode.W) then dir = dir + cf.LookVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.S) then dir = dir - cf.LookVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.A) then dir = dir - cf.RightVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.D) then dir = dir + cf.RightVector end
    if UserInputService:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0, 1, 0) end
    if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then dir = dir - Vector3.new(0, 1, 0) end

    if dir.Magnitude > 0 then
        root.AssemblyLinearVelocity = dir.Unit * _CFG.FlySpeed
    else
        root.AssemblyLinearVelocity = Vector3.zero
    end
    root.AssemblyAngularVelocity = Vector3.zero
end)
track(flyConn)

-- Noclip Loop
local noclipConn = RunService.Stepped:Connect(function()
    if HUB.dead or not _CFG.Noclip then return end
    local char = GetCharacter()
    if char then
        for _, part in ipairs(char:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    end
end)
track(noclipConn)

-- Click Teleport
track(UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe or not _CFG.ClickTP then return end
    if input.KeyCode == Enum.KeyCode.T then
        local hrp = GetHRP()
        if not hrp then return end
        local mouseLoc = UserInputService:GetMouseLocation()
        local ray = Camera:ViewportPointToRay(mouseLoc.X, mouseLoc.Y)
        local params = RaycastParams.new()
        params.FilterType = Enum.RaycastFilterType.Exclude
        params.FilterDescendantsInstances = { GetCharacter() }
        local result = Workspace:Raycast(ray.Origin, ray.Direction * 5000, params)
        if result then
            if _CFG.TweenTP then
                local dist = (hrp.Position - result.Position).Magnitude
                local tw = TweenService:Create(hrp, TweenInfo.new(math.clamp(dist / 80, 0.4, 1.5), Enum.EasingStyle.Quad), { CFrame = CFrame.new(result.Position + Vector3.new(0, 3, 0)) })
                tw:Play()
            else
                hrp.CFrame = CFrame.new(result.Position + Vector3.new(0, 3, 0))
            end
        end
    end
end))

-- 3. Hitbox Expander Loop
local originalSizes = {}
local hitboxConn = RunService.Heartbeat:Connect(function()
    if HUB.dead then return end
    if _CFG.HitboxEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
                if hrp and hrp:IsA("BasePart") then
                    if not originalSizes[hrp] then originalSizes[hrp] = hrp.Size end
                    pcall(function()
                        hrp.Size = Vector3.new(_CFG.HitboxSize, _CFG.HitboxSize, _CFG.HitboxSize)
                        hrp.Transparency = 0.6
                        hrp.CanCollide = false
                        hrp.Massless = true
                    end)
                end
            end
        end
    else
        if next(originalSizes) then
            for part, orig in pairs(originalSizes) do
                if part and part.Parent then
                    pcall(function() part.Size = orig; part.Transparency = 1 end)
                end
            end
            table.clear(originalSizes)
        end
    end
end)
track(hitboxConn)

-- 4. Silent Aim Logic & Hooks
local hasDrawing = (typeof(Drawing) == "table") or (Drawing ~= nil and pcall(function() return Drawing.new end))
local fovCircle = nil
if hasDrawing then
    pcall(function()
        fovCircle = Drawing.new("Circle")
        fovCircle.Thickness = 1.5
        fovCircle.NumSides = 64
        fovCircle.Radius = _CFG.SilentFOV
        fovCircle.Filled = false
        fovCircle.Visible = false
        fovCircle.Color = Library.Theme.Border
        fovCircle.ZIndex = 5
        trackDrawing(fovCircle)
    end)
end

local function getSilentAimTarget()
    if not _CFG.SilentAim then return nil end
    if math.random(1, 100) > _CFG.SilentHitChance then return nil end
    
    local camPos = Camera.CFrame.Position
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local mPos = UserInputService:GetMouseLocation()
    local ref = (mPos.Magnitude < 5) and center or mPos
    local myPos = GetHRP() and GetHRP().Position or camPos
    local bestPart, bestScore = nil, math.huge

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local skip = false
            if _CFG.SilentTeamCheck and plr.Team and LocalPlayer.Team and plr.Team == LocalPlayer.Team then skip = true end
            local char = plr.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if not skip and (not char or not hum or hum.Health <= 0) then skip = true end
            
            if not skip and _CFG.SilentTargetMode == "Murderer" then
                local role = GetRole(plr)
                if role ~= "Murderer" and role ~= "Murderer?" then skip = true end
            end
            
            local part
            if not skip then
                local want = _CFG.SilentHitPart
                if want == "Random" then want = (math.random() > 0.5) and "Head" or "HumanoidRootPart" end
                part = char:FindFirstChild(want) or char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
                if not part or not part:IsA("BasePart") then skip = true end
            end
            
            if not skip and _CFG.SilentWallCheck then
                local params = RaycastParams.new()
                params.FilterType = Enum.RaycastFilterType.Exclude
                params.FilterDescendantsInstances = { LocalPlayer.Character, char, Camera }
                local dir = part.Position - camPos
                local res = Workspace:Raycast(camPos, dir, params)
                if res and res.Instance and not res.Instance:IsDescendantOf(char) then skip = true end
            end
            
            if not skip then
                if _CFG.SilentTargetMode == "Nearest" then
                    local d = (part.Position - myPos).Magnitude
                    if d < bestScore then bestScore = d; bestPart = part end
                elseif _CFG.SilentTargetMode == "Chosen" then
                    if _CFG.SilentChosen and plr.Name == _CFG.SilentChosen then bestPart = part; break end
                else
                    local sp, on = Camera:WorldToViewportPoint(part.Position)
                    if on and sp.Z > 0 then
                        local d = (Vector2.new(sp.X, sp.Y) - ref).Magnitude
                        if d <= _CFG.SilentFOV and d < bestScore then bestScore = d; bestPart = part end
                    end
                end
            end
        end
    end
    return bestPart
end

-- Silent Aim Hook installation
do
    local G = (getgenv and getgenv()) or _G
    if not G.PayomboyZ_SilentHooked then
        G.PayomboyZ_SilentHooked = true
        pcall(function()
            if hookmetamethod then
                local oldNamecall
                oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
                    local method = getnamecallmethod()
                    if not checkcaller() and _CFG.SilentAim and method == "Raycast" and self == Workspace then
                        local args = {...}
                        local origin, direction, params = args[1], args[2], args[3]
                        if direction and typeof(direction) == "Vector3" and direction.Magnitude > 25 then
                            local t = getSilentAimTarget()
                            if t then
                                args[2] = (t.Position - origin).Unit * direction.Magnitude
                                return oldNamecall(self, table.unpack(args))
                            end
                        end
                    end
                    return oldNamecall(self, ...)
                end)
            end
        end)
    end
end

-- 5. FOV Circle Render
track(RunService.RenderStepped:Connect(function()
    if HUB.dead then return end
    if fovCircle then
        local on = _CFG.SilentShowFOV and _CFG.SilentAim
        fovCircle.Visible = on
        if on then
            pcall(function()
                fovCircle.Radius = _CFG.SilentFOV
                fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                fovCircle.Color = _CFG.RainbowESP and Color3.fromHSV((tick() % 5) / 5, 0.85, 1) or Library.Theme.Border
            end)
        end
    end
end))

-- 6. ESP Engine (Boxes, Chams, Names, Health, Tracers, Skeleton)
local playerEspObjects = {}

local function getBox2D(char)
    if not char then return nil end
    local ok, cf, size = pcall(function() return char:GetBoundingBox() end)
    if not ok or not cf or not size or size.Magnitude < 1 then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return nil end
        cf = hrp.CFrame
        size = Vector3.new(3, 6, 2)
    end
    local minX, minY = math.huge, math.huge
    local maxX, maxY = -math.huge, -math.huge
    local anyOn = false
    for x = -1, 1, 2 do
        for y = -1, 1, 2 do
            for z = -1, 1, 2 do
                local corner = (cf * CFrame.new(size.X / 2 * x, size.Y / 2 * y, size.Z / 2 * z)).Position
                local sp, on = Camera:WorldToViewportPoint(corner)
                if sp.Z > 0 then
                    anyOn = anyOn or on
                    minX = math.min(minX, sp.X); minY = math.min(minY, sp.Y)
                    maxX = math.max(maxX, sp.X); maxY = math.max(maxY, sp.Y)
                end
            end
        end
    end
    if minX == math.huge or not anyOn then return nil end
    return minX, minY, maxX, maxY
end

local function makePlayerESP()
    local box = {}
    if hasDrawing then
        pcall(function()
            box.frame = trackDrawing(Drawing.new("Square"))
            box.frame.Thickness = 1
            box.frame.Filled = false
            box.frame.Visible = false

            box.fill = trackDrawing(Drawing.new("Square"))
            box.fill.Thickness = 1
            box.fill.Filled = true
            box.fill.Transparency = 0.65
            box.fill.Visible = false

            box.outline = trackDrawing(Drawing.new("Square"))
            box.outline.Thickness = 2
            box.outline.Filled = false
            box.outline.Color = Color3.new(0, 0, 0)
            box.outline.Visible = false

            box.corners = {}
            for i = 1, 8 do
                local l = trackDrawing(Drawing.new("Line"))
                l.Thickness = 1
                l.Visible = false
                box.corners[i] = l
            end

            box.name = trackDrawing(Drawing.new("Text"))
            box.name.Size = 13
            box.name.Outline = true
            box.name.Centre = true
            box.name.Visible = false

            box.dist = trackDrawing(Drawing.new("Text"))
            box.dist.Size = 11
            box.dist.Outline = true
            box.dist.Centre = true
            box.dist.Visible = false

            box.hpOutline = trackDrawing(Drawing.new("Line"))
            box.hpOutline.Thickness = 3
            box.hpOutline.Color = Color3.new(0, 0, 0)
            box.hpOutline.Visible = false

            box.hp = trackDrawing(Drawing.new("Line"))
            box.hp.Thickness = 2
            box.hp.Visible = false

            box.hpText = trackDrawing(Drawing.new("Text"))
            box.hpText.Size = 11
            box.hpText.Outline = true
            box.hpText.Visible = false

            box.tracer = trackDrawing(Drawing.new("Line"))
            box.tracer.Thickness = 1.2
            box.tracer.Visible = false

            box.headCircle = trackDrawing(Drawing.new("Circle"))
            box.headCircle.Thickness = 1.5
            box.headCircle.NumSides = 24
            box.headCircle.Filled = false
            box.headCircle.Visible = false

            box.arrow = trackDrawing(Drawing.new("Triangle"))
            box.arrow.Thickness = 1
            box.arrow.Filled = true
            box.arrow.Visible = false

            box.skeleton = {}
            for i = 1, 6 do
                local l = trackDrawing(Drawing.new("Line"))
                l.Thickness = 1.2
                l.Visible = false
                box.skeleton[i] = l
            end
        end)
    end

    pcall(function()
        local parentGui = (typeof(gethui) == "function") and gethui() or CoreGui
        local hl = Instance.new("Highlight")
        hl.Name = "PayomboyZ_Chams"
        hl.FillTransparency = 0.55
        hl.OutlineTransparency = 0
        hl.Enabled = false
        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        hl.Parent = parentGui
        table.insert(HUB.highlights, hl)
        box.highlight = hl

        local halo = Instance.new("Highlight")
        halo.Name = "PayomboyZ_Halo"
        halo.FillTransparency = 1
        halo.OutlineTransparency = 0
        halo.Enabled = false
        halo.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        halo.Parent = parentGui
        table.insert(HUB.highlights, halo)
        box.halo = halo
    end)

    return box
end

local function removePlayerESP(plr)
    local obj = playerEspObjects[plr]
    if not obj then return end
    for _, d in ipairs({ obj.frame, obj.fill, obj.outline, obj.name, obj.dist, obj.hpOutline, obj.hp, obj.hpText, obj.tracer, obj.headCircle, obj.arrow }) do
        if d then pcall(function() d:Remove() end) end
    end
    if obj.corners then for _, l in ipairs(obj.corners) do if l then pcall(function() l:Remove() end) end end end
    if obj.skeleton then for _, l in ipairs(obj.skeleton) do if l then pcall(function() l:Remove() end) end end end
    if obj.highlight then pcall(function() obj.highlight:Destroy() end) end
    if obj.halo then pcall(function() obj.halo:Destroy() end) end
    playerEspObjects[plr] = nil
end

for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then playerEspObjects[p] = makePlayerESP() end
end
track(Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then playerEspObjects[p] = makePlayerESP() end
end))
track(Players.PlayerRemoving:Connect(removePlayerESP))

local espRenderConn = RunService.RenderStepped:Connect(function()
    if HUB.dead then return end
    local myHRP = GetHRP()
    local myPos = myHRP and myHRP.Position or Vector3.zero
    local rainbowCol = Color3.fromHSV((tick() % 5) / 5, 0.8, 1)

    for p, obj in pairs(playerEspObjects) do
        local isVis = _CFG.MasterESP and _CFG.PlayerESP
        local char = p.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        local head = char and (char:FindFirstChild("Head") or hrp)

        if isVis and char and hum and hrp and head and hum.Health > 0 then
            local dist = (hrp.Position - myPos).Magnitude
            if _CFG.MaxESPDist > 0 and dist > _CFG.MaxESPDist then
                isVis = false
            end

            if isVis then
                local role = GetRole(p)
                local col = (_CFG.RoleESP and RoleColor(role)) or (_CFG.RainbowESP and rainbowCol) or Library.Theme.Border

                -- Highlight & Halo
                if obj.highlight then
                    if _CFG.Chams then
                        obj.highlight.Adornee = char
                        obj.highlight.FillColor = col
                        obj.highlight.OutlineColor = col
                        obj.highlight.FillTransparency = _CFG.ChamsFill
                        obj.highlight.OutlineTransparency = _CFG.ChamsOutline
                        obj.highlight.Enabled = true
                    else
                        obj.highlight.Enabled = false
                    end
                end
                if obj.halo then
                    if _CFG.Halo then
                        obj.halo.Adornee = char
                        obj.halo.OutlineColor = col
                        obj.halo.Enabled = true
                    else
                        obj.halo.Enabled = false
                    end
                end

                local leftX, topY, rightX, bottomY = getBox2D(char)
                if leftX and topY and rightX and bottomY then
                    local w = rightX - leftX
                    local h = bottomY - topY
                    if h < 8 then local pad = (8 - h) / 2; topY = topY - pad; bottomY = bottomY + pad; h = 8 end
                    if w < 6 then local pad = (6 - w) / 2; leftX = leftX - pad; rightX = rightX + pad; w = 6 end
                    leftX = leftX - 1; topY = topY - 1; rightX = rightX + 1; bottomY = bottomY + 1
                    w = rightX - leftX; h = bottomY - topY
                    local cx = (leftX + rightX) / 2
                    local cornerLen = math.clamp(w * 0.28, 4, 18)

                    -- Boxes
                    if _CFG.BoxESP and hasDrawing then
                        if _CFG.BoxStyle == "Corner" and obj.corners then
                            if obj.frame then obj.frame.Visible = false end
                            if obj.outline then obj.outline.Visible = false end
                            local pts = {
                                { Vector2.new(leftX, topY), Vector2.new(leftX + cornerLen, topY) },
                                { Vector2.new(leftX, topY), Vector2.new(leftX, topY + cornerLen) },
                                { Vector2.new(rightX, topY), Vector2.new(rightX - cornerLen, topY) },
                                { Vector2.new(rightX, topY), Vector2.new(rightX, topY + cornerLen) },
                                { Vector2.new(leftX, bottomY), Vector2.new(leftX + cornerLen, bottomY) },
                                { Vector2.new(leftX, bottomY), Vector2.new(leftX, bottomY - cornerLen) },
                                { Vector2.new(rightX, bottomY), Vector2.new(rightX - cornerLen, bottomY) },
                                { Vector2.new(rightX, bottomY), Vector2.new(rightX, bottomY - cornerLen) },
                            }
                            for i, l in ipairs(obj.corners) do
                                if l then
                                    l.Visible = true; l.Color = col; l.Thickness = _CFG.BoxThickness
                                    l.From = pts[i][1]; l.To = pts[i][2]
                                end
                            end
                        else
                            if obj.corners then for _, l in ipairs(obj.corners) do if l then l.Visible = false end end end
                            if obj.outline then
                                obj.outline.Visible = true
                                obj.outline.Size = Vector2.new(w + 2, h + 2)
                                obj.outline.Position = Vector2.new(leftX - 1, topY - 1)
                            end
                            if obj.frame then
                                obj.frame.Visible = true; obj.frame.Color = col; obj.frame.Thickness = _CFG.BoxThickness
                                obj.frame.Size = Vector2.new(w, h); obj.frame.Position = Vector2.new(leftX, topY)
                            end
                        end
                    else
                        if obj.outline then obj.outline.Visible = false end
                        if obj.frame then obj.frame.Visible = false end
                        if obj.corners then for _, l in ipairs(obj.corners) do if l then l.Visible = false end end end
                    end

                    -- Box Fill
                    if _CFG.BoxFill and obj.fill then
                        obj.fill.Visible = true; obj.fill.Color = col
                        obj.fill.Size = Vector2.new(w, h); obj.fill.Position = Vector2.new(leftX, topY)
                    elseif obj.fill then
                        obj.fill.Visible = false
                    end

                    -- Name ESP
                    if _CFG.NameESP and obj.name then
                        obj.name.Visible = true
                        obj.name.Text = string.format("%s (@%s)", p.DisplayName, p.Name)
                        obj.name.Color = col; obj.name.Size = _CFG.TextSize
                        obj.name.Position = Vector2.new(cx, topY - 15)
                    elseif obj.name then
                        obj.name.Visible = false
                    end

                    -- Dist ESP
                    if _CFG.DistESP and obj.dist then
                        obj.dist.Visible = true
                        obj.dist.Text = string.format("%.0f studs", dist)
                        obj.dist.Size = math.max(9, _CFG.TextSize - 2)
                        obj.dist.Color = Color3.fromRGB(220, 220, 230)
                        obj.dist.Position = Vector2.new(cx, bottomY + 4)
                    elseif obj.dist then
                        obj.dist.Visible = false
                    end

                    -- Health Bar
                    if _CFG.HealthESP and obj.hp and obj.hpOutline then
                        local hpFrac = math.clamp(hum.Health / math.max(hum.MaxHealth, 1), 0, 1)
                        local barX = leftX - 6
                        obj.hpOutline.Visible = true
                        obj.hpOutline.From = Vector2.new(barX, topY - 1); obj.hpOutline.To = Vector2.new(barX, bottomY + 1)
                        obj.hp.Visible = true
                        obj.hp.Color = Color3.fromRGB(math.floor(255 * (1 - hpFrac)), math.floor(255 * hpFrac), 0)
                        obj.hp.From = Vector2.new(barX, bottomY); obj.hp.To = Vector2.new(barX, bottomY - h * hpFrac)
                        if obj.hpText then
                            obj.hpText.Visible = true
                            obj.hpText.Text = tostring(math.floor(hum.Health))
                            obj.hpText.Position = Vector2.new(barX - 16, (topY + bottomY) / 2 - 6)
                        end
                    else
                        if obj.hpOutline then obj.hpOutline.Visible = false end
                        if obj.hp then obj.hp.Visible = false end
                        if obj.hpText then obj.hpText.Visible = false end
                    end

                    -- Tracer
                    if _CFG.TracerESP and obj.tracer then
                        obj.tracer.Visible = true; obj.tracer.Color = col
                        local vs = Camera.ViewportSize
                        local from = Vector2.new(vs.X / 2, vs.Y - 4)
                        if _CFG.TracerOrigin == "Top" then from = Vector2.new(vs.X / 2, 4)
                        elseif _CFG.TracerOrigin == "Center" then from = Vector2.new(vs.X / 2, vs.Y / 2)
                        elseif _CFG.TracerOrigin == "Mouse" then from = UserInputService:GetMouseLocation() end
                        obj.tracer.From = from; obj.tracer.To = Vector2.new(cx, bottomY)
                    elseif obj.tracer then
                        obj.tracer.Visible = false
                    end

                    -- Head Circle
                    if _CFG.HeadCircleESP and obj.headCircle then
                        local headPos, onScreen = Camera:WorldToViewportPoint(head.Position)
                        if onScreen then
                            local rad = math.clamp((400 / math.max(dist, 10)) * 10, 6, 26)
                            obj.headCircle.Visible = true; obj.headCircle.Color = col
                            obj.headCircle.Radius = rad; obj.headCircle.Position = Vector2.new(headPos.X, headPos.Y)
                        else
                            obj.headCircle.Visible = false
                        end
                    elseif obj.headCircle then
                        obj.headCircle.Visible = false
                    end

                    -- Skeleton
                    if _CFG.SkeletonESP and obj.skeleton then
                        local parts = {
                            Head = char:FindFirstChild("Head"),
                            Torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso") or hrp,
                            LArm = char:FindFirstChild("LeftUpperArm") or char:FindFirstChild("Left Arm"),
                            RArm = char:FindFirstChild("RightUpperArm") or char:FindFirstChild("Right Arm"),
                            LLeg = char:FindFirstChild("LeftUpperLeg") or char:FindFirstChild("Left Leg"),
                            RLeg = char:FindFirstChild("RightUpperLeg") or char:FindFirstChild("Right Leg"),
                        }
                        local function toSc(pPart)
                            if not pPart or not pPart:IsA("BasePart") then return nil end
                            local sp, on = Camera:WorldToViewportPoint(pPart.Position)
                            return on and Vector2.new(sp.X, sp.Y) or nil
                        end
                        local hP, tP, laP, raP, llP, rlP = toSc(parts.Head), toSc(parts.Torso), toSc(parts.LArm), toSc(parts.RArm), toSc(parts.LLeg), toSc(parts.RLeg)
                        local pairsList = { {hP, tP}, {tP, laP}, {tP, raP}, {tP, llP}, {tP, rlP} }
                        for i, pair in ipairs(pairsList) do
                            local l = obj.skeleton[i]
                            if l then
                                if pair[1] and pair[2] then
                                    l.Visible = true; l.Color = col; l.From = pair[1]; l.To = pair[2]
                                else
                                    l.Visible = false
                                end
                            end
                        end
                    elseif obj.skeleton then
                        for _, l in ipairs(obj.skeleton) do if l then l.Visible = false end end
                    end
                else
                    -- Off-screen Arrow
                    if _CFG.ArrowESP and obj.arrow then
                        local sp, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                        if not onScreen then
                            local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
                            local dir = Vector2.new(sp.X, sp.Y) - center
                            if sp.Z < 0 then dir = -dir end
                            if dir.Magnitude > 0 then
                                dir = dir.Unit
                                local pos = center + dir * math.min(Camera.ViewportSize.X, Camera.ViewportSize.Y) * 0.38
                                local ang = math.atan2(dir.Y, dir.X)
                                local s = 14
                                obj.arrow.Visible = true; obj.arrow.Color = col
                                obj.arrow.PointA = pos + Vector2.new(math.cos(ang), math.sin(ang)) * s
                                obj.arrow.PointB = pos + Vector2.new(math.cos(ang + 2.5), math.sin(ang + 2.5)) * s
                                obj.arrow.PointC = pos + Vector2.new(math.cos(ang - 2.5), math.sin(ang - 2.5)) * s
                            else
                                obj.arrow.Visible = false
                            end
                        else
                            obj.arrow.Visible = false
                        end
                    elseif obj.arrow then
                        obj.arrow.Visible = false
                    end
                end
            end
        end

        if not isVis then
            if obj.frame then obj.frame.Visible = false end
            if obj.fill then obj.fill.Visible = false end
            if obj.outline then obj.outline.Visible = false end
            if obj.corners then for _, l in ipairs(obj.corners) do if l then l.Visible = false end end end
            if obj.name then obj.name.Visible = false end
            if obj.dist then obj.dist.Visible = false end
            if obj.hpOutline then obj.hpOutline.Visible = false end
            if obj.hp then obj.hp.Visible = false end
            if obj.hpText then obj.hpText.Visible = false end
            if obj.tracer then obj.tracer.Visible = false end
            if obj.headCircle then obj.headCircle.Visible = false end
            if obj.arrow then obj.arrow.Visible = false end
            if obj.skeleton then for _, l in ipairs(obj.skeleton) do if l then l.Visible = false end end end
            if obj.highlight then obj.highlight.Enabled = false end
            if obj.halo then obj.halo.Enabled = false end
        end
    end
end)
track(espRenderConn)

-- 7. Coin & Gun World Highlights
local coinHighlights = {}
task.spawn(function()
    while not HUB.dead do
        if _CFG.CoinESP and _CFG.MasterESP then
            pcall(function()
                local found = {}
                for _, v in ipairs(Workspace:GetDescendants()) do
                    if v.Name == "Coin" and v:IsA("BasePart") and v.Transparency < 1 then
                        found[v] = true
                        if not coinHighlights[v] then
                            local hl = Instance.new("Highlight")
                            hl.Name = "PayomboyZ_CoinHL"
                            hl.FillColor = Color3.fromRGB(255, 215, 0)
                            hl.FillTransparency = 0.5
                            hl.OutlineColor = Color3.fromRGB(255, 255, 120)
                            hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                            hl.Parent = v
                            table.insert(HUB.highlights, hl)
                            coinHighlights[v] = hl
                        end
                    end
                end
                for part, hl in pairs(coinHighlights) do
                    if not found[part] then pcall(function() hl:Destroy() end); coinHighlights[part] = nil end
                end
            end)
        elseif next(coinHighlights) then
            for p, hl in pairs(coinHighlights) do pcall(function() hl:Destroy() end); coinHighlights[p] = nil end
        end
        task.wait(1.2)
    end
end)

local gunHighlights = {}
task.spawn(function()
    while not HUB.dead do
        if _CFG.GunESP and _CFG.MasterESP then
            pcall(function()
                local found = {}
                for _, v in ipairs(Workspace:GetDescendants()) do
                    if (v.Name == "GunDrop" and v:IsA("BasePart")) or (v:IsA("Tool") and (v.Name:lower():find("gun") or v.Name:lower():find("knife")) and not v:IsDescendantOf(LocalPlayer)) then
                        local p = v:IsA("BasePart") and v or v:FindFirstChild("Handle") or v
                        if p and p:IsA("BasePart") then
                            found[p] = true
                            if not gunHighlights[p] then
                                local hl = Instance.new("Highlight")
                                hl.Name = "PayomboyZ_GunHL"
                                hl.FillColor = Color3.fromRGB(160, 60, 255)
                                hl.FillTransparency = 0.4
                                hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                                hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                                hl.Parent = p
                                table.insert(HUB.highlights, hl)
                                gunHighlights[p] = hl
                            end
                        end
                    end
                end
                for part, hl in pairs(gunHighlights) do
                    if not found[part] then pcall(function() hl:Destroy() end); gunHighlights[part] = nil end
                end
            end)
        elseif next(gunHighlights) then
            for p, hl in pairs(gunHighlights) do pcall(function() hl:Destroy() end); gunHighlights[p] = nil end
        end
        task.wait(1.2)
    end
end)

-- 8. Auto Grab Dropped Gun Thread
task.spawn(function()
    while not HUB.dead do
        if _CFG.AutoGrabGun then
            local gun = getGunDrop()
            local hrp = GetHRP()
            if gun and hrp then
                local pos = gun:IsA("BasePart") and gun.Position or gun:GetPivot().Position
                hrp.CFrame = CFrame.new(pos + Vector3.new(0, 2, 0))
                if firetouchinterest then
                    local h = gun:IsA("BasePart") and gun or gun:FindFirstChild("Handle") or gun:FindFirstChildWhichIsA("BasePart")
                    if h then pcall(function() firetouchinterest(hrp, h, 0); firetouchinterest(hrp, h, 1) end) end
                end
                Notify("Sheriff", "วาร์ปเก็บปืนที่ตกอัตโนมัติแล้ว!", "success")
                task.wait(1.5)
            end
        end
        task.wait(0.5)
    end
end)

-- 9. Murderer Kill Aura Thread
task.spawn(function()
    while not HUB.dead do
        if _CFG.KillAura and isMurderer() then
            local myPos = GetHRP() and GetHRP().Position or Vector3.zero
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                    local hum = plr.Character:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local d = (plr.Character.HumanoidRootPart.Position - myPos).Magnitude
                        if d <= _CFG.KillAuraDist then
                            executeKillPlayer(plr)
                            task.wait(_CFG.KillAuraDelay)
                        end
                    end
                end
            end
        end
        task.wait(0.12)
    end
end)

-- 10. Auto Farm Coins & Candies Master Loop (Deep Map Geometry Aware)
task.spawn(function()
    while not HUB.dead do
        if (_CFG.AutoFarm or _CFG.AutoFarmCandies) and isRoundActive() then
            if checkBagFullAction() then
                task.wait(2.5)
            else
                local hrp = GetHRP()
                if hrp then
                    if _CFG.FleeFromMurderer and not isMurderer() then
                        local murd = getMurdererPlayer()
                        if murd and murd.Character and murd.Character:FindFirstChild("HumanoidRootPart") then
                            local d = (murd.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                            if d <= (_CFG.FleeDistance or 35) then
                                local safePos = getSafestMapSpot(_CFG.SafeSpotType)
                                hrp.CFrame = CFrame.new(safePos)
                                hrp.AssemblyLinearVelocity = Vector3.zero
                                task.wait(1.5)
                            end
                        end
                    end

                    local targets = getSpawnedCoinsAndCandies()
                    local bestTarget, bestDist = nil, math.huge
                    local myPos = hrp.Position

                    for _, item in ipairs(targets) do
                        local match = false
                        local want = _CFG.FarmTarget or "All"
                        if want:find("Coins") and item.Kind == "Coin" then
                            match = true
                        elseif want:find("Candies") and item.Kind == "Candy" then
                            match = true
                        elseif want:find("All") or (not _CFG.AutoFarmCandies and item.Kind == "Coin") or (_CFG.AutoFarmCandies and not _CFG.AutoFarm and item.Kind == "Candy") or (_CFG.AutoFarm and _CFG.AutoFarmCandies) then
                            match = true
                        end

                        if match and item.Part and item.Part.Parent and item.Part.Transparency < 1 then
                            local d = (item.Position - myPos).Magnitude
                            if d < bestDist and d <= 450 then
                                bestDist = d
                                bestTarget = item
                            end
                        end
                    end

                    if bestTarget and bestTarget.Part and bestTarget.Part.Parent and bestTarget.Part.Transparency < 1 then
                        local tPart = bestTarget.Part
                        local targetPos = bestTarget.Position
                        local mode = _CFG.FarmMode or "Tween"

                        if mode:find("Instant") then
                            hrp.CFrame = CFrame.new(targetPos + Vector3.new(0, 2.0, 0))
                            hrp.AssemblyLinearVelocity = Vector3.zero
                            if firetouchinterest then
                                pcall(function() firetouchinterest(hrp, tPart, 0); firetouchinterest(hrp, tPart, 1) end)
                            end
                            task.wait(_CFG.Dwell or 0.4)
                        elseif mode:find("Ghost") then
                            local undergroundPos = targetPos - Vector3.new(0, 8, 0)
                            hrp.CFrame = CFrame.new(undergroundPos)
                            task.wait(0.08)
                            hrp.CFrame = CFrame.new(targetPos + Vector3.new(0, 1.8, 0))
                            if firetouchinterest then
                                pcall(function() firetouchinterest(hrp, tPart, 0); firetouchinterest(hrp, tPart, 1) end)
                            end
                            task.wait(_CFG.Dwell or 0.4)
                        else
                            local targetCFrame = CFrame.new(targetPos + Vector3.new(0, 2.0, 0))
                            local timeToReach = math.clamp(bestDist / math.max(_CFG.FarmSpeed or 95, 10), 0.1, 1.2)

                            hrp.Anchored = true
                            local tw = TweenService:Create(hrp, TweenInfo.new(timeToReach, Enum.EasingStyle.Linear), { CFrame = targetCFrame })
                            tw:Play()
                            tw.Completed:Wait()
                            hrp.Anchored = false
                            hrp.AssemblyLinearVelocity = Vector3.zero

                            if firetouchinterest then
                                pcall(function() firetouchinterest(hrp, tPart, 0); firetouchinterest(hrp, tPart, 1) end)
                            end
                            task.wait(_CFG.Dwell or 0.6)
                        end

                        checkBagFullAction()
                    end
                end
            end
        else
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") and char.HumanoidRootPart.Anchored then
                char.HumanoidRootPart.Anchored = false
            end
        end
        task.wait(0.18)
    end
end)

-- 11. Auto Open Crates Thread
task.spawn(function()
    local remotes = ReplicatedStorage:FindFirstChild("Remotes")
    local shop = remotes and remotes:FindFirstChild("Shop")
    local openCrate = shop and shop:FindFirstChild("OpenCrate")
    if openCrate then
        while not HUB.dead do
            if _CFG.AutoClaimBoxes then
                pcall(function() openCrate:InvokeServer("Summer2026Box", "MysteryBox", "SummerKey2026") end)
                pcall(function() openCrate:InvokeServer("MysteryBox2", "MysteryBox", "Coins") end)
            end
            task.wait(1.6)
        end
    end
end)

-- 12. Auto Buy Weapon on Missing Loop
task.spawn(function()
    while not HUB.dead do
        if _CFG.AutoBuyLowAmmo and _CFG.SelectedShopGun and not _CFG.SelectedShopGun:find("%(no") then
            local checkName = _CFG.SelectedShopGun:match("^%[[^%]]+%]") or _CFG.SelectedShopGun
            local has = false
            local char = GetCharacter()
            local bp = LocalPlayer:FindFirstChild("Backpack")
            if char then for _, t in ipairs(char:GetChildren()) do if t:IsA("Tool") and t.Name:find(checkName, 1, true) then has = true end end end
            if bp and not has then for _, t in ipairs(bp:GetChildren()) do if t:IsA("Tool") and t.Name:find(checkName, 1, true) then has = true end end end
            if not has then
                buyShopWeapon(_CFG.SelectedShopGun)
            end
        end
        task.wait(3.0)
    end
end)

-- 13. UI Status & Profile Auto Updater
task.spawn(function()
    while not HUB.dead do
        if murderStatusPara then
            murderStatusPara:Set(isMurderer() and "[MURDERER] กำลังเปิดโหมดสังหาร" or "[INNOCENT] คุณไม่ใช่ Murderer ในรอบนี้")
        end
        if balanceCard then
            balanceCard:Set(string.format("เหรียญ: %d | เลเวล: %d | บทบาท: %s", GetCoins(), GetLevel(), GetRole()))
        end
        task.wait(2.5)
    end
end)

-- 14. Auto Shoot Murderer Thread (Sheriff Combat)
task.spawn(function()
    while not HUB.dead do
        if _CFG.AutoShootMurderer and isRoundActive() then
            local myChar = GetCharacter()
            local hasGun = false
            if myChar and myChar:FindFirstChild("Gun") then hasGun = true end
            local bp = LocalPlayer:FindFirstChild("Backpack")
            if bp and bp:FindFirstChild("Gun") then hasGun = true end

            if hasGun then
                local murd = getMurdererPlayer()
                if murd and murd.Character and murd.Character:FindFirstChild("HumanoidRootPart") then
                    local hum = murd.Character:FindFirstChildOfClass("Humanoid")
                    if hum and hum.Health > 0 then
                        local myHRP = GetHRP()
                        if myHRP then
                            local d = (murd.Character.HumanoidRootPart.Position - myHRP.Position).Magnitude
                            if d <= _CFG.ShootMurdererDist then
                                local targetPos = murd.Character.HumanoidRootPart.Position
                                if murd.Character:FindFirstChild("Head") then
                                    targetPos = murd.Character.Head.Position
                                end
                                shootGunAt(targetPos)
                                task.wait(0.8)
                            end
                        end
                    end
                end
            end
        end
        task.wait(0.25)
    end
end)

-- 15. Anti-Murderer Dodge & Escape Thread
task.spawn(function()
    while not HUB.dead do
        if _CFG.AntiMurderer and isRoundActive() and not isMurderer() then
            local murd = getMurdererPlayer()
            local myHRP = GetHRP()
            if murd and murd.Character and murd.Character:FindFirstChild("HumanoidRootPart") and myHRP then
                local d = (murd.Character.HumanoidRootPart.Position - myHRP.Position).Magnitude
                if d <= _CFG.AntiMurdererDist then
                    local escapeDir = (myHRP.Position - murd.Character.HumanoidRootPart.Position).Unit
                    myHRP.CFrame = myHRP.CFrame + Vector3.new(escapeDir.X * 35, 30, escapeDir.Z * 35)
                    myHRP.AssemblyLinearVelocity = Vector3.new(0, 50, 0)
                    Notify("Defense", "หลบการโจมตีของ Murderer แล้ว!", "info")
                    task.wait(1.5)
                end
            end
        end
        task.wait(0.15)
    end
end)

-- 16. Auto Disarm & Trap Neutralizer Thread
task.spawn(function()
    while not HUB.dead do
        if isRoundActive() and not isMurderer() then
            if _CFG.AntiTrapNeutralize then
                neutralizeAllTraps()
            end
            if _CFG.AutoDisarmTraps then
                detonateTrapsSafely()
            end
        end
        task.wait(0.6)
    end
end)

-- 17. Auto Open Selected Crate Thread
task.spawn(function()
    while not HUB.dead do
        if _CFG.AutoOpenSelectedCrate then
            openMM2Crate(_CFG.SelectedCrate, _CFG.SelectedCrateCurrency)
            task.wait(1.8)
        else
            task.wait(0.5)
        end
    end
end)

-- 18. Auto Open Event Crates Thread
task.spawn(function()
    while not HUB.dead do
        if _CFG.AutoOpenEventCrates then
            openMM2Crate(_CFG.SelectedEventCrate, "Candies")
            task.wait(2.0)
        else
            task.wait(0.5)
        end
    end
end)

-- 19. Loop Emote Thread
task.spawn(function()
    while not HUB.dead do
        if _CFG.LoopEmote and _CFG.SelectedEmote then
            playMM2Emote(_CFG.SelectedEmote)
            task.wait(3.5)
        else
            task.wait(1.0)
        end
    end
end)

-- 20. Auto Claim BattlePass Loop
task.spawn(function()
    while not HUB.dead do
        if _CFG.AutoClaimBattlePass then
            claimAllEventPassTiers()
            task.wait(45)
        else
            task.wait(2)
        end
    end
end)


-- ==============================================================================
-- [[ CLEANUP & UNLOAD HANDLER ]]
-- ==============================================================================

local function CleanupAll()
    if _CFG and _CFG.AutoSaveConfig and SaveConfig then
        pcall(function() SaveConfig(_CFG.SelectedConfigSlot or "Default") end)
    end
    HUB.dead = true
    _CFG.AutoFarm = false
    _CFG.KillAura = false
    _CFG.SilentAim = false
    _CFG.HitboxEnabled = false
    _CFG.FlyEnabled = false
    _CFG.Noclip = false
    _CFG.MasterESP = false

    for _, c in ipairs(HUB.conns) do pcall(function() c:Disconnect() end) end
    for _, d in ipairs(HUB.drawings) do pcall(function() d:Remove() end) end
    for _, h in ipairs(HUB.highlights) do pcall(function() h:Destroy() end) end

    -- Restore Hitboxes
    for part, orig in pairs(originalSizes) do
        if part and part.Parent then pcall(function() part.Size = orig; part.Transparency = 1 end) end
    end
    table.clear(originalSizes)

    -- Restore Character & Environment
    local hum = GetHumanoid()
    if hum then
        hum.PlatformStand = false
        hum.WalkSpeed = 16
        hum.JumpPower = 50
        hum.UseJumpPower = true
        hum.HipHeight = 2
    end
    Workspace.Gravity = 196.2
    Camera.FieldOfView = 70
    pcall(function() RunService:Set3dRenderingEnabled(true) end)

    if _CFG.Fullbright then
        Lighting.Brightness = savedLighting.Brightness
        Lighting.ClockTime = savedLighting.ClockTime
        Lighting.FogEnd = savedLighting.FogEnd
        Lighting.GlobalShadows = savedLighting.GlobalShadows
        Lighting.Ambient = savedLighting.Ambient
    end

    if ScreenGui then pcall(function() ScreenGui:Destroy() end) end
end

HUB.SaveConfig = SaveConfig
HUB.LoadConfig = LoadConfig
HUB.ResetDefaultConfig = ResetDefaultConfig
HUB.Unload = CleanupAll
_G.MM2Cleanup = CleanupAll

Notify("PayomboyZ", "Secret Hub MM2 Master UI โหลดสมบูรณ์พร้อมใช้งาน!", "success", 4)


local exportedAPI = {
    HUB = HUB,
    CFG = _CFG,
    getActiveMap = getActiveMap,
    getCoinContainer = getCoinContainer,
    getSpawnedCoinsAndCandies = getSpawnedCoinsAndCandies,
    getGunDrop = getGunDrop,
    getTraps = getTraps,
    ensureSafePlatform = ensureSafePlatform,
    getSafestMapSpot = getSafestMapSpot,
    neutralizeAllTraps = neutralizeAllTraps,
    detonateTrapsSafely = detonateTrapsSafely,
    playMM2Emote = playMM2Emote,
    stopCurrentEmote = stopCurrentEmote,
    shootGunAt = shootGunAt,
    executeKillPlayer = executeKillPlayer,
    openMM2Crate = openMM2Crate,
    claimAllEventPassTiers = claimAllEventPassTiers,
    getMurdererPlayer = getMurdererPlayer,
    getSheriffPlayer = getSheriffPlayer,
    isMurderer = isMurderer,
    isRoundActive = isRoundActive,
    Unload = CleanupAll
}

_G.PayomboyZ_MM2 = exportedAPI
local _G_ENV = (getgenv and getgenv()) or _G
if _G_ENV then pcall(function() _G_ENV.PayomboyZ_MM2 = exportedAPI end) end

Notify("PayomboyZ", "Secret Hub MM2 Master UI โหลดสำเร็จแล้ว!", "success", 4)

return exportedAPI
