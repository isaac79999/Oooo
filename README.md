--// LocalScript - Roblox Studio
-- Mais leve e sem travar

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer

local Remote = ReplicatedStorage:WaitForChild("RemoteNovos"):WaitForChild("TrocaAcontecendo")

local SuaOferta = LocalPlayer.PlayerGui:WaitForChild("ShopTools")
	:WaitForChild("all")
	:WaitForChild("TradeFrame")
	:WaitForChild("MainFrame")
	:WaitForChild("Ofertas")
	:WaitForChild("SuaOferta")

local modeloBase = SuaOferta:FindFirstChild("Agua")

-- evita loop infinito
local executando = false

-- hook leve
local mt = getrawmetatable(game)
local old = mt.__namecall

setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
	local args = { ... }
	local method = getnamecallmethod()

	-- só detecta InvokeServer do remote certo
	if not executando
		and self == Remote
		and method == "InvokeServer"
	then

		local dados = args[2]

		if typeof(dados) == "table" then
			local item = dados[1]
			local quantidade = dados[2]
			local invent = dados[3]

			-- evita crash
			executando = true

			task.spawn(function()

				-- manda só +2 vezes
				for i = 1, 2 do
					pcall(function()
						Remote:InvokeServer(
							args[1],
							{
								item,
								quantidade,
								invent
							}
						)
					end)

					task.wait(0.05)
				end

				executando = false
			end)

			-- visual
			if modeloBase then
				task.spawn(function()
					for i = 1, 3 do
						local clone = modeloBase:Clone()
						clone.Name = item
						clone.Visible = true
						clone.Parent = SuaOferta
					end
				end)
			end
		end
	end

	return old(self, ...)
end)

setreadonly(mt, true)

print("Hook leve ativo.")
