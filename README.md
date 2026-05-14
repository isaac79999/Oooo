--// LocalScript - Roblox Studio
--// FIPE Finder COMPLETO + Compatibilidade de nomes

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

local DealershipMenu = LocalPlayer.PlayerGui:WaitForChild("DealershipMenu")
local Spawner = DealershipMenu.Main:WaitForChild("Spawner")

local Hui = gethui()

-- NORMALIZAR NOMES
local function Normalizar(txt)

	txt = txt:lower()

	-- remove espaços
	txt = txt:gsub("%s+", "")

	-- remove traços
	txt = txt:gsub("%-", "")

	-- remove underline
	txt = txt:gsub("_", "")

	return txt
end

-- TABELA FIPE
local Fipe = {

	-- Ferrari
	["Ferrari 458"] = "25kk",
	["Ferrari F40"] = "60kk",
	["Ferrari SF90"] = "55kk",
	["LaFerrari"] = "75kk",
	["F1 Ferrari"] = "80kk",

	-- Lamborghini
	["Lamborghini Urus"] = "35kk",
	["Lamborghini Diablo"] = "60kk",
	["Lamborghini Huracan"] = "20kk",

	-- McLaren
	["McLaren P1"] = "40kk",
	["McLaren GT"] = "20kk",
	["McLaren Senna"] = "65kk",

	-- Porsche
	["Porsche 718 Cayman"] = "15kk",
	["Porsche Carrera"] = "700k",
	["Porsche 918"] = "75kk",
	["Porsche (GT3 Antigo)"] = "2kk",
	["Porsche Cayenne"] = "250k",

	-- Mercedes
	["F1 Mercedes AMG"] = "80kk",
	["Mercedes GT 63"] = "20kk",
	["Mercedes SLS"] = "70kk",
	["Mercedes GLE"] = "20kk",
	["Mercedes G63"] = "35kk",
	["Mercedes E63"] = "5kk",
	["Mercedes Maybach"] = "55kk",

	-- BMW
	["BMW X6M"] = "10kk",
	["BMW M4"] = "25kk",
	["BMW I8"] = "VIP",
	["BMW S1000"] = "12kk",
	["BMW R1200"] = "15kk",

	-- Audi
	["Audi A4"] = "250k",
	["Audi R8 GT"] = "30kk",
	["Audi Q7"] = "200k",

	-- Dodge
	["Dodge RAM"] = "10kk",
	["Dodge Charger"] = "35kk",

	-- Ford
	["Ford Mustang GT"] = "10kk",
	["Ford Pampa"] = "Exclusivo apenas",

	-- Chevrolet
	["Trailblazer"] = "450k",
	["Camaro"] = "400k",

	-- Toyota
	["Toyota Hilux"] = "450k",
	["SW4"] = "250k",
	["Land Cruiser"] = "300k",
	["Supra MK4"] = "20kk",

	-- Volkswagen
	["Golf GTI"] = "5kk",
	["Fusca"] = "5k",
	["Saveiro"] = "10k",
	["Amarok"] = "3kk",
	["Voyage"] = "40k",

	-- Honda
	["CBR 1000RR-R"] = "12kk",
	["Transalp"] = "1k",
	["Titan 160"] = "1kk",
	["XRE 300"] = "5kk",

	-- Outras marcas
	["Tesla Model S"] = "300k",
	["Uno"] = "10k",
	["Corsa"] = "10k",
	["HB20"] = "50k",
	["Tucson"] = "50k",
	["Nissan GT-R R35"] = "25kk",
	["Skyline"] = "25k",
	["Mazda RX7"] = "25kk",
	["Mitsubishi Eclipse"] = "25kk",
	["Lancer Evo IX"] = "35kk",
	["Koenigsegg"] = "15kk",
	["Bugatti Vision"] = "20kk",
	["Volvo XC90"] = "400k",
	["GMC Yukon"] = "1.9kk",
	["Cadillac Escalade"] = "1.5kk",
	["Rolls Royce Cullinan"] = "30kk",
	["Range Rover"] = "200k",
	["Caminhão"] = "Limitado",
	["Cybertruck"] = "30kk",

	-- Outros
	["Helicóptero"] = "50kk",
	["Bicicleta"] = "Free"
}

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FIPE_FIX"
ScreenGui.Parent = Hui
ScreenGui.ResetOnSpawn = false

-- MAIN
local Main = Instance.new("Frame")
Main.Parent = ScreenGui
Main.Size = UDim2.new(0,450,0,165)
Main.Position = UDim2.new(0.5,-225,0.18,0)
Main.BackgroundColor3 = Color3.fromRGB(170,0,0)
Main.BorderSizePixel = 0
Main.Visible = false

local Corner = Instance.new("UICorner")
Corner.Parent = Main

-- TITLE
local Title = Instance.new("TextLabel")
Title.Parent = Main
Title.Size = UDim2.new(1,0,0,22)
Title.BackgroundTransparency = 1
Title.Text = "FIPE Finder"
Title.TextColor3 = Color3.new(1,1,1)
Title.TextScaled = true

-- SCROLL
local Scroll = Instance.new("ScrollingFrame")
Scroll.Parent = Main
Scroll.Size = UDim2.new(1,-10,0,90)
Scroll.Position = UDim2.new(0,5,0,25)
Scroll.BackgroundTransparency = 1
Scroll.ScrollBarThickness = 6
Scroll.ScrollingDirection = Enum.ScrollingDirection.X
Scroll.AutomaticCanvasSize = Enum.AutomaticSize.X

local Layout = Instance.new("UIListLayout")
Layout.Parent = Scroll
Layout.FillDirection = Enum.FillDirection.Horizontal
Layout.Padding = UDim.new(0,5)

-- RESULTADO
local Resultado = Instance.new("TextLabel")
Resultado.Parent = Main
Resultado.Size = UDim2.new(1,-10,0,40)
Resultado.Position = UDim2.new(0,5,0,120)
Resultado.BackgroundTransparency = 1
Resultado.Text = "Escolha um veículo"
Resultado.TextColor3 = Color3.new(1,1,1)
Resultado.TextScaled = true
Resultado.TextWrapped = true

-- LIMPAR
local function Limpar()

	for _,v in pairs(Scroll:GetChildren()) do
		if v:IsA("ImageButton") then
			v:Destroy()
		end
	end
end

-- PEGAR VALOR FIPE
local function PegarValor(nomeVeiculo)

	local nomeNormalizado = Normalizar(nomeVeiculo)

	for nomeFipe,valor in pairs(Fipe) do

		if Normalizar(nomeFipe) == nomeNormalizado then
			return valor
		end
	end

	return nil
end

-- ATUALIZAR ÍCONES
local function Atualizar()

	Limpar()

	local Seleciona = Spawner:FindFirstChild("Seleciona")

	if not Seleciona then
		return
	end

	for _,carro in pairs(Seleciona:GetChildren()) do

		local Botao = Instance.new("ImageButton")
		Botao.Parent = Scroll
		Botao.Size = UDim2.new(0,75,0,75)
		Botao.BackgroundColor3 = Color3.fromRGB(45,45,45)
		Botao.BorderSizePixel = 0

		local C = Instance.new("UICorner")
		C.Parent = Botao

		-- PEGAR IMAGEM
		local imagem = nil

		pcall(function()

			local img1 = carro:FindFirstChildWhichIsA("ImageLabel")
			local img2 = carro:FindFirstChildWhichIsA("ImageButton")

			if img1 then
				imagem = img1.Image
			end

			if img2 then
				imagem = img2.Image
			end
		end)

		if imagem then
			Botao.Image = imagem
		end

		-- NOME
		local Nome = Instance.new("TextLabel")
		Nome.Parent = Botao
		Nome.Size = UDim2.new(1,0,0,15)
		Nome.Position = UDim2.new(0,0,1,-15)
		Nome.BackgroundTransparency = 1
		Nome.Text = carro.Name
		Nome.TextScaled = true
		Nome.TextColor3 = Color3.new(1,1,1)

		-- CLICK
		Botao.MouseButton1Click:Connect(function()

			local valor = PegarValor(carro.Name)

			if valor then
				Resultado.Text = carro.Name.." = "..valor
			else
				Resultado.Text = carro.Name.." = Sem valor FIPE"
			end
		end)
	end
end

-- ATUALIZA AUTOMÁTICO
task.spawn(function()

	while true do
		pcall(Atualizar)
		task.wait(3)
	end
end)

-- MOSTRAR SOMENTE COM MENU ABERTO
RunService.RenderStepped:Connect(function()

	local aberto = false

	pcall(function()
		aberto = DealershipMenu.Enabled
	end)

	Main.Visible = aberto
end)
