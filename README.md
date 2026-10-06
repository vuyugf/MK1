-- =======================================================
-- MAGIC ARMOR MK1 - SISTEMA DE SAIR E REEQUIPAR
-- Compatível com Delta Executor (Mobile)
-- =======================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

if PlayerGui:FindFirstChild("ArmorControlUI") then
    PlayerGui.ArmorControlUI:Destroy()
end

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")

_G.IsEquipped = true
_G.ArmorStandModel = nil
_G.ProximityConnection = nil

local StoneGrey      = Color3.fromRGB(130, 135, 138)
local DarkStone      = Color3.fromRGB(70, 75, 80)
local VisorBlack     = Color3.fromRGB(15, 15, 15)
local DarkMetalColor = Color3.fromRGB(35, 38, 42)

-- =======================================================
-- 1. ESTRUTURA E CRIAÇÃO DA ARMADURA
-- =======================================================

local function ApplyCharacterStyle(char)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        hum.WalkSpeed = 36
        hum.JumpPower = 110
        local hs = hum:FindFirstChild("BodyHeightScale") or Instance.new("NumberValue", hum)
        hs.Value = 1.15
        local ws = hum:FindFirstChild("BodyWidthScale") or Instance.new("NumberValue", hum)
        ws.Value = 1.1
    end

    for _, child in ipairs(char:GetChildren()) do
        if child:IsA("Accessory") then child:Destroy() end
    end

    local shirt = char:FindFirstChildOfClass("Shirt")
    if shirt then shirt:Destroy() end
    local pants = char:FindFirstChildOfClass("Pants")
    if pants then pants:Destroy() end

    local bodyColors = char:FindFirstChildOfClass("BodyColors") or Instance.new("BodyColors", char)
    bodyColors.HeadColor3 = StoneGrey
    bodyColors.TorsoColor3 = StoneGrey
    bodyColors.LeftArmColor3 = StoneGrey
    bodyColors.RightArmColor3 = StoneGrey
    bodyColors.LeftLegColor3 = StoneGrey
    bodyColors.RightLegColor3 = StoneGrey
end

local function CreatePart(parent, targetPart, size, offset, color, material, name)
    if not targetPart then return nil end

    local part = Instance.new("Part")
    part.Name = name
    part.Size = size
    part.Color = color or StoneGrey
    part.Material = material or Enum.Material.Slate
    part.CanCollide = false
    part.Massless = true
    part.CFrame = targetPart.CFrame * offset
    part.Parent = parent

    local weld = Instance.new("WeldConstraint")
    weld.Part0 = targetPart
    weld.Part1 = part
    weld.Parent = part

    return part
end

local function BuildArmorOnEntity(entity)
    local ArmorFolder = Instance.new("Folder")
    ArmorFolder.Name = "MagicArmor_Full"
    ArmorFolder.Parent = entity

    local ShieldFolder = Instance.new("Folder")
    ShieldFolder.Name = "ShieldParts"
    ShieldFolder.Parent = ArmorFolder

    local GunFolder = Instance.new("Folder")
    GunFolder.Name = "GunParts"
    GunFolder.Parent = ArmorFolder

    local BaseArmorFolder = Instance.new("Folder")
    BaseArmorFolder.Name = "BaseArmorParts"
    BaseArmorFolder.Parent = ArmorFolder

    local head = entity:FindFirstChild("Head")
    local torso = entity:FindFirstChild("Torso") or entity:FindFirstChild("UpperTorso")
    local leftArm = entity:FindFirstChild("Left Arm") or entity:FindFirstChild("LeftUpperArm")
    local rightArm = entity:FindFirstChild("Right Arm") or entity:FindFirstChild("RightUpperArm")
    local leftLeg = entity:FindFirstChild("Left Leg") or entity:FindFirstChild("LeftUpperLeg")
    local rightLeg = entity:FindFirstChild("Right Leg") or entity:FindFirstChild("RightUpperLeg")

    if head then
        CreatePart(BaseArmorFolder, head, Vector3.new(1.3, 1.3, 1.3), CFrame.new(0, 0.1, 0), StoneGrey, Enum.Material.Slate, "HelmetBase")
        CreatePart(BaseArmorFolder, head, Vector3.new(0.25, 0.4, 1.3), CFrame.new(0, 0.75, 0), StoneGrey, Enum.Material.Slate, "HelmetCrest")
        CreatePart(BaseArmorFolder, head, Vector3.new(1.0, 0.5, 0.1), CFrame.new(0, 0.05, -0.66), StoneGrey, Enum.Material.Slate, "VisorFrame")
        for i = -1, 1 do
            CreatePart(BaseArmorFolder, head, Vector3.new(0.75, 0.07, 0.12), CFrame.new(0, 0.05 + (i * 0.1), -0.68), VisorBlack, Enum.Material.SmoothPlastic, "VisorStripe"..i)
        end
    end

    if torso then
        CreatePart(BaseArmorFolder, torso, Vector3.new(2.3, 2.2, 1.5), CFrame.new(0, 0.05, 0), StoneGrey, Enum.Material.Slate, "ChestPlate")
        CreatePart(BaseArmorFolder, torso, Vector3.new(2.4, 0.8, 1.6), CFrame.new(0, 0.4, 0), DarkStone, Enum.Material.Slate, "ChestDetail")
        CreatePart(BaseArmorFolder, torso, Vector3.new(1.9, 1.3, 1.55), CFrame.new(0, -0.6, 0), DarkMetalColor, Enum.Material.DiamondPlate, "ChainmailBase")
        CreatePart(BaseArmorFolder, torso, Vector3.new(1.8, 1.2, 0.2), CFrame.new(0, -0.6, -0.78), DarkMetalColor, Enum.Material.DiamondPlate, "ChainmailFront")
    end

    if leftArm then CreatePart(BaseArmorFolder, leftArm, Vector3.new(1.5, 1.1, 1.5), CFrame.new(0, 0.5, 0), StoneGrey, Enum.Material.Slate, "L_Shoulder") end
    if rightArm then CreatePart(BaseArmorFolder, rightArm, Vector3.new(1.5, 1.1, 1.5), CFrame.new(0, 0.5, 0), StoneGrey, Enum.Material.Slate, "R_Shoulder") end
    if leftLeg then CreatePart(BaseArmorFolder, leftLeg, Vector3.new(1.2, 1.6, 1.2), CFrame.new(0, -0.1, 0), StoneGrey, Enum.Material.Slate, "L_Leg") end
    if rightLeg then CreatePart(BaseArmorFolder, rightLeg, Vector3.new(1.2, 1.6, 1.2), CFrame.new(0, -0.1, 0), StoneGrey, Enum.Material.Slate, "R_Leg") end

    if leftArm then
        CreatePart(ShieldFolder, leftArm, Vector3.new(0.4, 3.4, 2.0), CFrame.new(-0.8, -0.1, 0), StoneGrey, Enum.Material.Slate, "ShieldBase")
        CreatePart(ShieldFolder, leftArm, Vector3.new(0.5, 2.5, 0.25), CFrame.new(-0.9, -0.1, 0), DarkStone, Enum.Material.Slate, "ShieldCrossV")
        CreatePart(ShieldFolder, leftArm, Vector3.new(0.5, 0.3, 1.3), CFrame.new(-0.9, 0.3, 0), DarkStone, Enum.Material.Slate, "ShieldCrossH")
    end

    if rightArm then
        local downRotation = CFrame.Angles(math.rad(-90), 0, 0)
        local sideOffset = CFrame.new(1.0, -0.6, -0.1) * downRotation
        CreatePart(GunFolder, rightArm, Vector3.new(1.0, 1.2, 1.8), sideOffset, DarkMetalColor, Enum.Material.Metal, "GatlingBody")
        
        for i = 1, 6 do
            local angle = math.rad(i * 60)
            local x = math.cos(angle) * 0.28
            local y = math.sin(angle) * 0.28
            local barrelOffset = sideOffset * CFrame.new(x, y, -1.8)
            CreatePart(GunFolder, rightArm, Vector3.new(0.18, 0.18, 2.8), barrelOffset, DarkMetalColor, Enum.Material.Metal, "Barrel"..i)
        end
    end
end

-- Equipa a armadura inicialmente
ApplyCharacterStyle(Character)
BuildArmorOnEntity(Character)

-- =======================================================
-- 2. SISTEMA DE SAIR DA ARMADURA E DETECÇÃO DE PERTO
-- =======================================================

local function EjectArmor()
    if not _G.IsEquipped then return end
    _G.IsEquipped = false

    local hrp = Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Remove armadura do jogador
    local oldArmor = Character:FindFirstChild("MagicArmor_Full")
    if oldArmor then oldArmor:Destroy() end

    -- Restaura velocidade normal
    if Humanoid then
        Humanoid.WalkSpeed = 16
        Humanoid.JumpPower = 50
        local hs = Humanoid:FindFirstChild("BodyHeightScale")
        if hs then hs.Value = 1.0 end
        local ws = Humanoid:FindFirstChild("BodyWidthScale")
        if ws then ws.Value = 1.0 end
    end

    -- Cria o Manequim da Armadura Parada no Chão
    local Stand = Instance.new("Model")
    Stand.Name = "ArmorStand"
    Stand.Parent = workspace

    local TorsoPart = Instance.new("Part")
    TorsoPart.Name = "Torso"
    TorsoPart.Size = Vector3.new(2, 2, 1)
    TorsoPart.CFrame = hrp.CFrame * CFrame.new(0, 0.5, 0)
    TorsoPart.Anchored = true
    TorsoPart.Transparency = 1
    TorsoPart.Parent = Stand

    local HeadPart = Instance.new("Part")
    HeadPart.Name = "Head"
    HeadPart.Size = Vector3.new(1.2, 1.2, 1.2)
    HeadPart.CFrame = TorsoPart.CFrame * CFrame.new(0, 1.5, 0)
    HeadPart.Anchored = true
    HeadPart.Transparency = 1
    HeadPart.Parent = Stand

    local LeftArmPart = Instance.new("Part")
    LeftArmPart.Name = "Left Arm"
    LeftArmPart.Size = Vector3.new(1, 2, 1)
    LeftArmPart.CFrame = TorsoPart.CFrame * CFrame.new(-1.5, 0, 0)
    LeftArmPart.Anchored = true
    LeftArmPart.Transparency = 1
    LeftArmPart.Parent = Stand

    local RightArmPart = Instance.new("Part")
    RightArmPart.Name = "Right Arm"
    RightArmPart.Size = Vector3.new(1, 2, 1)
    RightArmPart.CFrame = TorsoPart.CFrame * CFrame.new(1.5, 0, 0)
    RightArmPart.Anchored = true
    RightArmPart.Transparency = 1
    RightArmPart.Parent = Stand

    -- Monta o visual da armadura na estátua
    BuildArmorOnEntity(Stand)
    _G.ArmorStandModel = Stand

    -- Movimenta o jogador um pouco para fora
    Character:SetPrimaryPartCFrame(hrp.CFrame * CFrame.new(0, 0, -4))

    -- Verifica aproximação para reequipar automaticamente
    _G.ProximityConnection = RunService.Stepped:Connect(function()
        if not _G.IsEquipped and _G.ArmorStandModel and Character and Character:FindFirstChild("HumanoidRootPart") then
            local dist = (Character.HumanoidRootPart.Position - TorsoPart.Position).Magnitude
            if dist <= 4.5 then
                -- Reequipa a armadura
                if _G.ProximityConnection then _G.ProximityConnection:Disconnect() end
                if _G.ArmorStandModel then _G.ArmorStandModel:Destroy() end
                
                ApplyCharacterStyle(Character)
                BuildArmorOnEntity(Character)
                _G.IsEquipped = true
            end
        end
    end)
end

-- =======================================================
-- 3. INTERFACE GRÁFICA (GUI)
-- =======================================================

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ArmorControlUI"
ScreenGui.ResetOnSpawn = false

pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
if not ScreenGui.Parent then ScreenGui.Parent = PlayerGui end

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 210, 0, 225)
MainFrame.Position = UDim2.new(0.05, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.Active = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)
local MainStroke = Instance.new("UIStroke", MainFrame)
MainStroke.Thickness = 2
MainStroke.Color = Color3.fromRGB(46, 204, 113)

local dragging, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
MainFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local function CreateMenuButton(text, posY, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 11
    btn.Font = Enum.Font.GothamBold
    btn.Parent = MainFrame
    
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    return btn
end

local BtnShield = CreateMenuButton("ESCUDO: ON / OFF", 12, Color3.fromRGB(35, 45, 60))
local BtnGun    = CreateMenuButton("METRALHADORA: ON / OFF", 62, Color3.fromRGB(60, 35, 35))
local BtnAim    = CreateMenuButton("APONTAR METRALHADORA", 112, Color3.fromRGB(35, 60, 40))
local BtnEject  = CreateMenuButton("SAIR DA ARMADURA", 162, Color3.fromRGB(150, 40, 40))

-- =======================================================
-- 4. AÇÕES DOS BOTÕES
-- =======================================================

local shieldVisible = true
BtnShield.MouseButton1Click:Connect(function()
    local full = Character:FindFirstChild("MagicArmor_Full")
    if not full then return end
    local shieldFolder = full:FindFirstChild("ShieldParts")
    if not shieldFolder then return end

    shieldVisible = not shieldVisible
    for _, part in ipairs(shieldFolder:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = shieldVisible and 0 or 1
        end
    end
end)

local gunVisible = true
BtnGun.MouseButton1Click:Connect(function()
    local full = Character:FindFirstChild("MagicArmor_Full")
    if not full then return end
    local gunFolder = full:FindFirstChild("GunParts")
    if not gunFolder then return end

    gunVisible = not gunVisible
    for _, part in ipairs(gunFolder:GetChildren()) do
        if part:IsA("BasePart") then
            part.Transparency = gunVisible and 0 or 1
        end
    end
end)

local aiming = false
local aimConnection = nil

BtnAim.MouseButton1Click:Connect(function()
    local rightArm = Character:FindFirstChild("Right Arm") or Character:FindFirstChild("RightUpperArm")
    local torso = Character:FindFirstChild("Torso") or Character:FindFirstChild("UpperTorso")
    if not rightArm or not torso then return end
    
    local armJoint = rightArm:FindFirstChild("RightShoulder") or torso:FindFirstChild("Right Shoulder")
    if not armJoint then return end

    aiming = not aiming
    
    if aiming then
        BtnAim.Text = "ABAIXAR METRALHADORA"
        aimConnection = RunService.Stepped:Connect(function()
            if aiming and armJoint then
                armJoint.C0 = CFrame.new(1.2, 0.5, 0) * CFrame.Angles(math.rad(90), 0, 0)
            end
        end)
    else
        BtnAim.Text = "APONTAR METRALHADORA"
        if aimConnection then aimConnection:Disconnect() end
        armJoint.C0 = CFrame.new(1.5, 0.5, 0) * CFrame.Angles(0, 0, 0)
    end
end)

BtnEject.MouseButton1Click:Connect(function()
    EjectArmor()
end)
