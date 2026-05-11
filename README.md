--// LocalScript - PC
-- Hook usando hookfunction no InvokeServer

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer = Players.LocalPlayer

local Remote = ReplicatedStorage
	:WaitForChild("RemoteNovos")
	:WaitForChild("TrocaAcontecendo")

local SuaOferta = LocalPlayer.PlayerGui
	:WaitForChild("ShopTools")
	:WaitForChild("all")
	:WaitForChild("TradeFrame")
	:WaitForChild("MainFrame")
	:WaitForChild("Ofertas")
	:WaitForChild("SuaOferta")

local Modelo = SuaOferta:FindFirstChild("Agua")

local executando = false

local oldInvoke
oldInvoke = hookfunction(Remote.InvokeServer, function(self, ...)

	local args = { ... }

	if self == Remote and not executando then

		local dados = args[2]

		if typeof(dados) == "table" then

			local item = dados[1]
			local quantidade = dados[2]
			local invent = dados[3]

			executando = true

			-- +2 fires
			task.spawn(function()

				for i = 1, 2 do
					pcall(function()
						oldInvoke(
							self,
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
			if Modelo then
				task.spawn(function()

					for i = 1, 3 do
						local clone = Modelo:Clone()
						clone.Name = item
						clone.Visible = true
						clone.Parent = SuaOferta
					end

				end)
			end
		end
	end

	return oldInvoke(self, ...)
end)

print("Hook PC ativo.")
