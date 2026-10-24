local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer

-- Configuração de Cores
local COLOR_MURDER = Color3.fromRGB(255, 0, 0)       -- Vermelho (Assassino)
local COLOR_SHERIFF = Color3.fromRGB(0, 100, 255)     -- Azul (Xerife)
local COLOR_INNOCENT = Color3.fromRGB(0, 255, 100)    -- Verde (Inocente)
local COLOR_DROPPED_GUN = Color3.fromRGB(170, 0, 255) -- Roxo (Arma no Chão)

-- Função genérica para criar ou atualizar o Highlight (visível através da parede)
local function applyHighlight(object, color, name)
	if not object then return end
	local highlight = object:FindFirstChild(name)
	if not highlight then
		highlight = Instance.new("Highlight")
		highlight.Name = name
		highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
		highlight.FillTransparency = 0.4
		highlight.OutlineTransparency = 0
		highlight.Parent = object
	end
	
	highlight.FillColor = color
	highlight.OutlineColor = color
end

-- Função para identificar o papel do jogador (com suporte a skins)
local function getPlayerRole(player)
	local character = player.Character
	local backpack = player:FindFirstChild("Backpack")
	
	if not character then return "Innocent" end

	local function checkContainer(container)
		if not container then return nil end
		for _, tool in ipairs(container:GetChildren()) do
			if tool:IsA("Tool") then
				local name = tool.Name:lower()
				-- Identifica se é faca (Murderer)
				if name:find("knife") or name:find("faca") or tool:FindFirstChild("Stab") or tool:FindFirstChild("Slash") then
					return "Murderer"
				-- Identifica se é arma/revolver (Sheriff)
				elseif name:find("gun") or name:find("revolver") or name:find("pistol") or tool:FindFirstChild("Shoot") then
					return "Sheriff"
				end
			end
		end
		return nil
	end

	return checkContainer(character) or checkContainer(backpack) or "Innocent"
end

-- Atualiza o ESP dos Jogadores
local function applyPlayerESP(player)
	if player == LocalPlayer then return end

	local character = player.Character
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end

	local role = getPlayerRole(player)
	local color = COLOR_INNOCENT

	if role == "Murderer" then
		color = COLOR_MURDER
	elseif role == "Sheriff" then
		color = COLOR_SHERIFF
	end

	applyHighlight(character, color, "RoleESP")
end

-- Procura e aplica ESP Roxo nas Armas caídas na Workspace
local function scanDroppedGuns()
	for _, item in ipairs(Workspace:GetChildren()) do
		-- Verifica se a arma está solta na Workspace (Tool caída ou Model/Part de drop)
		if item:IsA("Tool") then
			local name = item.Name:lower()
			if name:find("gun") or name:find("revolver") or name:find("pistol") or item:FindFirstChild("Shoot") then
				applyHighlight(item, COLOR_DROPPED_GUN, "GunESP")
			end
		elseif item:IsA("Model") or item:IsA("BasePart") then
			local name = item.Name:lower()
			if name == "gundrop" or name == "gun" or name:find("revolver") then
				applyHighlight(item, COLOR_DROPPED_GUN, "GunESP")
			end
		end
	end
end

-- Loop contínuo protegido com pcall para rodar liso e sem travar
RunService.Heartbeat:Connect(function()
	pcall(function()
		-- 1. Executa o ESP dos Jogadores
		for _, player in ipairs(Players:GetPlayers()) do
			if player ~= LocalPlayer then
				applyPlayerESP(player)
			end
		end
		
		-- 2. Executa o ESP da Arma no Chão na Workspace
		scanDroppedGuns()
	end)
end)

-- Notificação de inicialização
game:GetService("StarterGui"):SetCore("SendNotification", {
	Title = "Max.hub MM2",
	Text = "ESP Completo Ativo (Players + Workspace)!",
	Duration = 4
})
