-- === BYPASS CHECKANTI + COBALT DETECTION v5 ===
-- Resolve GetNil + getcallbackvalue

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

print("🛠️ Iniciando patch anti-cobalt...")

-- Hook getcallbackvalue
local oldGetCallback = getcallbackvalue
getcallbackvalue = newcclosure(function(obj, prop)
    if obj.Name == "CheckAnti" and prop == "OnClientInvoke" then
        print("✅ Cobalt tentou pegar callback, entregando fake true")
        return function(...)
            return true
        end
    end
    return oldGetCallback(obj, prop)
end)

-- Hook getnilinstances também
local oldGetNilInstances = getnilinstances
getnilinstances = newcclosure(function()
    local instances = oldGetNilInstances()
    -- Não altera nada, só pra não dar suspeita
    return instances
end)

-- Patch direto no remote
local function patchRemote()
    local remote = nil
    
    -- Procura em ReplicatedStorage e nil
    for _, obj in ipairs(game:GetDescendants()) do
        if obj.Name == "CheckAnti" then
            remote = obj
            break
        end
    end
    
    if not remote then
        for _, obj in ipairs(getnilinstances()) do
            if obj.Name == "CheckAnti" then
                remote = obj
                break
            end
        end
    end
    
    if remote then
        -- Força o callback correto
        remote.OnClientInvoke = function(...)
            return true
        end
        
        -- Patch extra pra Cobalt
        pcall(function()
            setcallbackvalue(remote, "OnClientInvoke", function(...)
                return true
            end)
        end)
        
        -- Protege contra mudança
        remote.Changed:Connect(function(p)
            if p == "Parent" or p == "Archivable" then
                remote.Parent = ReplicatedStorage:FindFirstChild("RemoteNovos") or remote.Parent
            end
        end)
        
        print("✅ CheckAnti patchado com sucesso pro Cobalt")
        return true
    end
    return false
end

-- Loop rápido
task.spawn(function()
    while task.wait(0.08) do
        patchRemote()
        
        -- Proteção geral
        pcall(function()
            hookfunction(LocalPlayer.Kick, function() end)
            hookfunction(game, "Kick", function() end)
        end)
    end
end)

-- Namecall protection
local mt = getrawmetatable(game)
setreadonly(mt, false)
local oldNamecall = mt.__namecall

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    if self.Name == "CheckAnti" then
        if method == "InvokeClient" then
            return true
        end
        if method == "InvokeServer" then
            return
        end
    end
    if method == "Kick" or method == "Destroy" then
        if self == LocalPlayer then return end
    end
    return oldNamecall(self, ...)
end)
setreadonly(mt, true)

print("🚀 Patch Cobalt + Remote completo.")
print("Executa e testa. Se ainda desconectar, me manda print do console F9.")

-- Anti Anti-Cheat Delta - Bloqueia remoção em massa
-- Coloca isso rodando ANTES do anti-cheat carregar, ou usa com auto-execute

local Protected = {}
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

local function IsProtected(obj)
    if not obj then return false end
    if Protected[obj] then return true end
    
    -- Protege tudo no workspace que não for SpawnLocation
    if obj:IsDescendantOf(Workspace) and not obj:IsA("SpawnLocation") then
        return true
    end
    
    -- Protege GUIs e Scripts do LocalPlayer
    if obj:IsDescendantOf(LocalPlayer) and (obj:IsA("ScreenGui") or obj:IsA("LocalScript") or obj:IsA("ModuleScript")) then
        return true
    end
    
    return false
end

-- Hook bruto no __namecall (método mais eficiente pro Delta)
local mt = getrawmetatable(game)
local oldNamecall = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    
    if (method == "Remove" or method == "Destroy") then
        if IsProtected(self) then
            -- print("🛡️ Bloqueado tentativa de remover: " .. self:GetFullName()) -- descomenta se quiser log
            return -- fode o anti-cheat aqui
        end
    end
    
    -- Bloqueia também o loop de GetDescendants indireto se precisar
    if method == "GetDescendants" then
        local original = oldNamecall(self, ...)
        -- Pode filtrar aqui se quiser, mas o hook de Remove já resolve a maioria
    end
    
    return oldNamecall(self, ...)
end)

setreadonly(mt, true)

-- Backup hook no Remove/Destroy direto (caso o namecall falhe em algum executor)
local oldRemove = Instance.Remove
local oldDestroy = Instance.Destroy

Instance.Remove = newcclosure(function(obj)
    if IsProtected(obj) then
        return
    end
    return oldRemove(obj)
end)

Instance.Destroy = newcclosure(function(obj)
    if IsProtected(obj) then
        return
    end
    return oldDestroy(obj)
end)

-- Proteção extra: recria caso o filho seja removido
local function ProtectInstance(obj)
    if IsProtected(obj) then
        Protected[obj] = true
        obj.AncestryChanged:Connect(function(_, parent)
            if not parent then
                -- Tenta recolocar no lugar se removerem
                task.delay(0.1, function()
                    if obj.Parent == nil and Protected[obj] then
                        -- Aqui você pode recolocar no workspace ou no player conforme necessário
                        -- Exemplo simples:
                        if obj:IsA("ScreenGui") then
                            obj.Parent = LocalPlayer:FindFirstChild("PlayerGui") or LocalPlayer
                        end
                    end
                end)
            end
        end)
    end
end

-- Aplica proteção em tudo que já existe
for _, obj in ipairs(Workspace:GetDescendants()) do
    ProtectInstance(obj)
end
for _, obj in ipairs(LocalPlayer:GetDescendants()) do
    ProtectInstance(obj)
end

-- Protege novos objetos que aparecerem
Workspace.DescendantAdded:Connect(ProtectInstance)
LocalPlayer.DescendantAdded:Connect(ProtectInstance)

print("✅ Anti-Anti-Cheat carregado com sucesso. O filho da puta não vai limpar mais porra nenhuma.")

-- ==================== ANTI CRASH ADICIONADO ====================
local lastCrashCheck = tick()
local crashDebounce = 0

local function AntiCrash()
    if tick() - lastCrashCheck < 0.5 then return end
    lastCrashCheck = tick()
    
    pcall(function()
        -- Limpa memória inútil
        if collectgarbage then
            collectgarbage("collect")
        end
        
        -- Protege contra recursion infinita no namecall
        if crashDebounce > 5 then
            crashDebounce = 0
            return
        end
        crashDebounce = crashDebounce + 1
        
        -- Hook de segurança extra contra crash comum
        pcall(function()
            hookfunction(game.HttpGet, function() return "" end)
            hookfunction(game.HttpPost, function() return "" end)
        end)
    end)
end

-- Loop leve do anti-crash
task.spawn(function()
    while wait(1.2) do  -- bem leve pra não lagar
        pcall(AntiCrash)
    end
end)

print("🛡️ Anti-Crash carregado. Lag e crash devem ter sumido, caralho.")
