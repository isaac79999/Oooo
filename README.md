local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")
local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()

local oldRequire
oldRequire = hookfunction(getrenv().require, newcclosure(function(module)
    if module:IsA("ModuleScript") and (module.Name:find("AntiCheat") or module.Name:find("Orpheus")) then
        warn("🚫 ORPHEUS BLOQUEADO NO CARREGAMENTO")
        return {}
    end
    return oldRequire(module)
end))

local function disableKick(obj)
    local oldK
    oldK = hookfunction(obj.Kick, newcclosure(function(self, reason)
        if self == LocalPlayer then
            warn("🚫 TENTATIVA DE KICK BLOQUEADA: " .. tostring(reason))
            return
        end
        return oldK(self, reason)
    end))
end

disableKick(LocalPlayer)
disableKick(game)

local chestSize = 5

local function applyHitbox(player)
    if player == LocalPlayer then return end
    
    local function heart()
        local char = player.Character
        if char then
            local torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
            if torso then
                local hitbox = char:FindFirstChild("RealChestHitbox")
                if not hitbox then
                    hitbox = Instance.new("Part")
                    hitbox.Name = "RealChestHitbox"
                    hitbox.Transparency = 0.6
                    hitbox.BrickColor = BrickColor.new("Bright yellow")
                    hitbox.Material = Enum.Material.Neon
                    hitbox.CanCollide = false
                    hitbox.Anchored = true
                    hitbox.Parent = char
                end
                hitbox.Size = Vector3.new(chestSize, chestSize, chestSize)
                hitbox.CFrame = torso.CFrame
            end
        end
    end
    
    RunService.Heartbeat:Connect(heart)
end

for _, p in ipairs(Players:GetPlayers()) do applyHitbox(p) end
Players.PlayerAdded:Connect(applyHitbox)

local function createUI()
    local sg = Instance.new("ScreenGui")
    sg.Name = "Internal_Check"
    sg.ResetOnSpawn = false
    
    if syn and syn.protect_gui then syn.protect_gui(sg) end
    sg.Parent = CoreGui
    
    local frame = Instance.new("Frame", sg)
    frame.Size = UDim2.new(0, 140, 0, 40)
    frame.Position = UDim2.new(0.5, -70, 0.85, 0)
    frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    frame.BorderSizePixel = 0
    
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(1, 0, 1, 0)
    btn.Text = "Chest Hitbox: " .. chestSize
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.BackgroundTransparency = 1
    
    btn.MouseButton1Click:Connect(function()
        chestSize = (chestSize >= 15) and 3 or (chestSize + 3)
        btn.Text = "Chest Hitbox: " .. chestSize
    end)
end

createUI()
print("✅ SCRIPT CARREGADO. ORPHEUS BYPASSED.")
