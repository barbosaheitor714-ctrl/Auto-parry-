-- [[ AUTO PARRY EDUCACIONAL - ESTRUTURA BASE ]] --

-- Configurações (Ajustáveis pelo usuário)
local Config = {
    Enabled = true,
    DistanceThreshold = 15, -- Distância ideal para ativar o parry
    DynamicTiming = true,   -- Ajusta o tempo com base na velocidade do projétil
    Keybind = Enum.KeyCode.F -- Tecla que executa o bloqueio no jogo
}

-- Serviços do Roblox
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local LocalPlayer = Players.LocalPlayer

-- Função para simular o clique da tecla de Parry
local function triggerParry()
    -- Simula fisicamente o pressionar da tecla para fins de acessibilidade
    VirtualInputManager:SendKeyEvent(true, Config.Keybind, false, game)
    task.wait(0.05)
    VirtualInputManager:SendKeyEvent(false, Config.Keybind, false, game)
    print("[Auto Parry] Bloqueio ativado!")
end

-- Função principal de monitoramento
local function monitorThreats()
    if not Config.Enabled then return end
    
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local playerPos = character.HumanoidRootPart.Position

    -- NOTA: Em um cenário real, você buscaria pelo objeto da bola ou do ataque inimigo
    -- Exemplo genérico: procurando por uma "Bola" na Workspace
    local ball = workspace:FindFirstChild("Ball") or workspace:FindFirstChild("Projectile")
    
    if ball and ball:IsA("BasePart") then
        local ballPos = ball.Position
        local distance = (ballPos - playerPos).Magnitude
        local velocity = ball.AssemblyLinearVelocity -- Velocidade física do objeto
        
        -- Cálculo de Tempo até o Impacto (Time to Impact)
        -- Fórmula ideal: Tempo = Distância / Velocidade em direção ao jogador
        local speed = velocity.Magnitude
        
        if Config.DynamicTiming and speed > 0 then
            -- Ajusta o gatilho baseado na velocidade real do objeto
            local timeToHit = distance / speed
            
            -- Se o tempo de impacto for menor que o tempo de reação do servidor (~0.1s a 0.2s)
            if timeToHit <= 0.15 then
                triggerParry()
                task.wait(0.5) -- Cooldown para evitar spam e detecção
            end
        else
            -- Bloqueio simples por aproximação estática
            if distance <= Config.DistanceThreshold then
                triggerParry()
                task.wait(0.5)
            end
        end
    end
end

-- Loop de alta performance (roda a cada frame renderizado)
local connection
connection = RunService.RenderStepped:Connect(function()
    if not Config.Enabled then 
        connection:Disconnect() 
        return 
    end
    
    -- Proteção contra erros para o script não quebrar se o personagem morrer
    pcall(monitorThreats)
end)
