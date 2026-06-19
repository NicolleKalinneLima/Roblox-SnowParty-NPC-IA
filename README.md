# Roblox-SnowParty-NPC-IA
local event = game.ReplicatedStorage:WaitForChild("CapyDestination")
local npc = script.Parent
local humanoid = npc.Humanoid
local root = npc.HumanoidRootPart
local head = npc.Head

local PathfindingService = game:GetService("PathfindingService")
local Players = game:GetService("Players")
local ChatService = game:GetService("Chat")
local maxDistance = 30000

local HttpService = game:GetService("HttpService")
local API_KEY = "Sua_Chave_API_Groqw" 
local GROQ_URL = "https://api.groq.com/openai/v1/chat/completions" 

-- ==========================================
-- FUNÇÃO DA INTELIGÊNCIA ARTIFICIAL (GROQ)
-- ==========================================
local function perguntarParaIA(mensagemDoJogador)
	local contextoDoNPC = 
		"Você é a Capy, uma capivara guia turística super amigável no jogo do Roblox chamado 'Snow Party'.\n" ..
		"REGRAS DE LOCAIS:\n" ..
		"Os únicos locais do mapa são: 'Área da Piscina', 'Salão Principal', 'Capy Store', 'Área de Eventos' e 'Spawn'. Seu favorito é a Área da Piscina.\n\n" ..
		"REGRA DE MOVIMENTAÇÃO AUTOMÁTICA:\n" ..
		"Se o jogador pedir para você LEVAR ele a algum lugar, mostrar o mapa, ou pedir para ir ao seu lugar favorito, você DEVE aceitar e colocar obrigatoriamente uma dessas tags exatas no FINAL da sua resposta:\n" ..
		"- Se for para a Piscina: [IR_PARA:Piscina]\n" ..
		"- Se for para o Salão Principal: [IR_PARA:Salao]\n" ..
		"- Se for para a Capy Store: [IR_PARA:Loja]\n" ..
		"- Se for para a Área de Eventos: [IR_PARA:Evento]\n" ..
		"- Se for para o Spawn: [IR_PARA:Spawn]\n" ..
		"Exemplo de resposta: 'Claro, me siga até a piscina morna! [IR_PARA:Piscina]'\n" ..
		"Se ele estiver apenas conversando e não pediu para se mover, NÃO adicione nenhuma tag. Responda em português de forma curta (máximo 2 frases)."

	local payload = {
		model = "llama-3.1-8b-instant", 
		messages = {
			{ role = "system", content = contextoDoNPC },
			{ role = "user", content = mensagemDoJogador }
		}
	}

	local success, result = pcall(function()
		return HttpService:RequestAsync({
			Url = GROQ_URL,
			Method = "POST",
			Headers = {
				["Authorization"] = "Bearer " .. API_KEY,
				["Content-Type"] = "application/json"
			},
			Body = HttpService:JSONEncode(payload)
		})
	end)

	if success and result.Success then
		local data = HttpService:JSONDecode(result.Body)
		return data.choices[1].message.content
	else
		warn("Erro na API: ", result.Body)
		return "Desculpe, deu um tilt no meu cérebro de capivara."
	end
end

-- ==========================================
-- SISTEMA DE MOVIMENTAÇÃO (PATHFINDING)
-- ==========================================
local function IrPara(destino)
	local path = PathfindingService:CreatePath({
		AgentRadius = 2,
		AgentHeight = 5,
		AgentCanJump = true
	})

	path:ComputeAsync(root.Position, destino.Position)

	if path.Status == Enum.PathStatus.Success then
		local waypoints = path:GetWaypoints()
		for _, waypoint in ipairs(waypoints) do
			if waypoint.Action == Enum.PathWaypointAction.Jump then
				humanoid.Jump = true
			end
			humanoid:MoveTo(waypoint.Position)
			local chegou = humanoid.MoveToFinished:Wait()
			if not chegou then break end
		end
	end
end

local function OlharParaJogador(player)
	local character = player.Character
	if not character then return end
	local playerRoot = character:FindFirstChild("HumanoidRootPart")
	if not playerRoot then return end

	npc:PivotTo(CFrame.lookAt(root.Position, Vector3.new(playerRoot.Position.X, root.Position.Y, playerRoot.Position.Z)))
end

-- ==========================================
-- LÓGICA DO TOUR (AÇÕES DO MAPA)
-- ==========================================
local function IniciarTour(player, destino)
	if destino == "Piscina" then
		ChatService:Chat(head, "Claro! Me siga até a área da piscina.", Enum.ChatColor.White)
		wait(1.5)
		IrPara(workspace.TourPoints.PiscinaPoint)
		OlharParaJogador(player)
		ChatService:Chat(head, "Bem-vindo à área da piscina! Este é um dos lugares favoritos dos jogadores para relaxar.", Enum.ChatColor.White)

	elseif destino == "Salao" then
		ChatService:Chat(head, "Vamos conhecer o Salão Principal!", Enum.ChatColor.White)
		wait(1.5)
		IrPara(workspace.TourPoints.CaminhoSalao1)
		IrPara(workspace.TourPoints.CaminhoSalao2)
		IrPara(workspace.TourPoints.SalaoPoint)
		OlharParaJogador(player)
		ChatService:Chat(head, "Chegamos ao Salão Principal! Aqui você encontra a pista de dança e o palco.", Enum.ChatColor.White)

	elseif destino == "Loja" then
		ChatService:Chat(head, "Vamos até a Capy Store!", Enum.ChatColor.White)
		wait(1.5)
		IrPara(workspace.TourPoints.CapyStorePoint)
		OlharParaJogador(player)
		ChatService:Chat(head, "Bem-vindo à Capy Store! Aqui você pode gastar suas CapyCoins.", Enum.ChatColor.White)

	elseif destino == "Evento" then
		ChatService:Chat(head, "Vou te mostrar a Área de Eventos!", Enum.ChatColor.White)
		wait(1.5)
		IrPara(workspace.TourPoints.CaminhoEvento1)
		IrPara(workspace.TourPoints.CaminhoEvento2)
		IrPara(workspace.TourPoints.CaminhoEvento3)
		IrPara(workspace.TourPoints.EventoPoint)
		OlharParaJogador(player)
		ChatService:Chat(head, "Chegamos à Área de Eventos! Aqui acontecem atividades especiais.", Enum.ChatColor.White)

	elseif destino == "Spawn" then
		ChatService:Chat(head, "Vamos voltar para o Spawn!", Enum.ChatColor.White)
		wait(1.5)
		IrPara(workspace.TourPoints.SpawnPoint)
		OlharParaJogador(player)
		ChatService:Chat(head, "Chegamos ao Spawn! O início da nossa jornada.", Enum.ChatColor.White)
	end
end

-- ==========================================
-- ESCUTAR CHAT DO JOGADOR (INTEGRAÇÃO IA)
-- ==========================================
local function respondToPlayer(player, message)
	local character = player.Character
	if head and character then
		local playerRoot = character:FindFirstChild("HumanoidRootPart")
		if playerRoot then
			-- Só responde se estiver perto (15 studs)
			local distance = (head.Position - playerRoot.Position).Magnitude
			if distance <= 15 then
				ChatService:Chat(head, "Pensando...", Enum.ChatColor.Blue)

				local respostaIA = perguntarParaIA(message)

				-- 1. Verifica se a IA colocou o código secreto de andar
				local comandoAndar = string.match(respostaIA, "%[IR_PARA:(%w+)%]")

				-- 2. Limpa a tag da mensagem para o jogador não ver o código feio
				local textoLimpo = string.gsub(respostaIA, "%s*%[IR_PARA:%w+]", "")

				-- 3. Faz a Capy falar o texto limpo
				ChatService:Chat(head, textoLimpo, Enum.ChatColor.White)

				-- 4. Se tiver comando de andar, ela inicia o tour automaticamente!
				if comandoAndar then
					wait(1)
					IniciarTour(player, comandoAndar)
				end
			end
		end
	end
end

-- Conexão do Chat
Players.PlayerAdded:Connect(function(player)
	player.Chatted:Connect(function(message)
		respondToPlayer(player, message)
	end)
end)

-- Mantém o funcionamento dos botões da tela antigos se clicar neles
event.OnServerEvent:Connect(function(player, destino)
	IniciarTour(player, destino)
end)
