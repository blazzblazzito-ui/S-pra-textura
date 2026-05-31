-- Carregando a Rayfield UI Library
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Nicolex hubz",
    LoadingTitle = "Carregando Nicolex hubz...",
    LoadingSubtitle = "Brookhaven RP",
    ConfigurationSaving = {
        Enabled = false,
        FolderName = nil,
        FileName = "NicolexHub"
    },
    Discord = {
        Enabled = false,
        Invite = "noinvitelink",
        RememberJoins = true
    },
    KeySystem = false
})

-- Variáveis Globais
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local TargetPlayerName = ""
local flying = false
local flySpeed = 50

-- ==================== TAB: LOCAL PLAYER ====================
local TabPlayer = Window:CreateTab("Local Player", "user")

TabPlayer:CreateSlider({
    Name = "Velocidade (WalkSpeed)",
    Range = {16, 250},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 16,
    Flag = "SliderSpeed",
    Callback = function(Value)
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.WalkSpeed = Value
        end
    end,
})

TabPlayer:CreateSlider({
    Name = "Pulo (JumpPower)",
    Range = {50, 250},
    Increment = 1,
    Suffix = "Power",
    CurrentValue = 50,
    Flag = "SliderJump",
    Callback = function(Value)
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            LocalPlayer.Character.Humanoid.UseJumpPower = true
            LocalPlayer.Character.Humanoid.JumpPower = Value
        end
    end,
})

TabPlayer:CreateToggle({
    Name = "Invisível",
    CurrentValue = false,
    Flag = "ToggleInvis",
    Callback = function(Value)
        if Value then
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("LowerTorso") then
                local clone = LocalPlayer.Character:Clone()
                clone.Parent = Workspace
                LocalPlayer.Character = clone
                Workspace.CurrentCamera.CameraSubject = clone:FindFirstChild("Humanoid")
            end
        else
            LocalPlayer.Character:BreakJoints()
        end
    end,
})

TabPlayer:CreateSlider({
    Name = "Velocidade do Voo (Fly Speed)",
    Range = {10, 200},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 50,
    Flag = "SliderFlySpeed",
    Callback = function(Value)
        flySpeed = Value
    end,
})

TabPlayer:CreateToggle({
    Name = "Ativar Voo (Fly)",
    CurrentValue = false,
    Flag = "ToggleFly",
    Callback = function(Value)
        flying = Value
        local char = LocalPlayer.Character
        
        if flying and char and char:FindFirstChild("HumanoidRootPart") then
            local bg = Instance.new("BodyGyro", char.HumanoidRootPart)
            local bv = Instance.new("BodyVelocity", char.HumanoidRootPart)
            bg.P = 9e4
            bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
            bv.maxForce = Vector3.new(9e9, 9e9, 9e9)
            
            if _G.FlyLoop then _G.FlyLoop:Disconnect() end
            
            _G.FlyLoop = RunService.RenderStepped:Connect(function()
                if flying and char:FindFirstChild("Humanoid") and char:FindFirstChild("HumanoidRootPart") then
                    char.Humanoid.PlatformStand = true
                    local cam = Workspace.CurrentCamera
                    local moveDir = char.Humanoid.MoveDirection
                    
                    bg.cframe = cam.CFrame
                    
                    if moveDir.Magnitude > 0 then
                        local camLook = cam.CFrame.LookVector
                        local camRight = cam.CFrame.RightVector
                        
                        local flatLook = Vector3.new(camLook.X, 0, camLook.Z).Unit
                        local flatRight = Vector3.new(camRight.X, 0, camRight.Z).Unit
                        
                        local forwardDot = moveDir:Dot(flatLook)
                        local rightDot = moveDir:Dot(flatRight)
                        
                        local moveDir3D = (camLook * forwardDot) + (camRight * rightDot)
                        
                        bv.velocity = moveDir3D.Unit * flySpeed
                    else
                        bv.velocity = Vector3.new(0, 0, 0)
                    end
                else
                    if bg then bg:Destroy() end
                    if bv then bv:Destroy() end
                    if char and char:FindFirstChild("Humanoid") then
                        char.Humanoid.PlatformStand = false
                    end
                    if _G.FlyLoop then _G.FlyLoop:Disconnect() end
                end
            end)
        else
            flying = false
            if char and char:FindFirstChild("Humanoid") then
                char.Humanoid.PlatformStand = false
            end
            for _, v in pairs(char.HumanoidRootPart:GetChildren()) do
                if v:IsA("BodyGyro") or v:IsA("BodyVelocity") then
                    v:Destroy()
                end
            end
            if _G.FlyLoop then _G.FlyLoop:Disconnect() end
        end
    end,
})

-- ==================== TAB: VISUAIS & ESP ====================
local TabVisuals = Window:CreateTab("Visuals", "eye")

TabVisuals:CreateToggle({
    Name = "ESP (Body/Name)",
    CurrentValue = false,
    Flag = "ToggleESP",
    Callback = function(Value)
        if Value then
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local highlight = Instance.new("Highlight")
                    highlight.Parent = player.Character
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.Name = "ESP_Nicolex"
                end
            end
        else
            for _, player in pairs(Players:GetPlayers()) do
                if player.Character and player.Character:FindFirstChild("ESP_Nicolex") then
                    player.Character.ESP_Nicolex:Destroy()
                end
            end
        end
    end,
})

-- ==================== TAB: ALVOS (TARGETS) ====================
local TabTarget = Window:CreateTab("Target / Troll", "crosshair")

TabTarget:CreateInput({
    Name = "Nome do Jogador Alvo",
    PlaceholderText = "Digite o nome aqui...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        for _, p in pairs(Players:GetPlayers()) do
            if string.sub(string.lower(p.Name), 1, string.len(Text)) == string.lower(Text) then
                TargetPlayerName = p.Name
                Rayfield:Notify({
                    Title = "Alvo Encontrado",
                    Content = "Travado em: " .. TargetPlayerName,
                    Duration = 3,
                    Image = "check",
                })
            end
        end
    end,
})

TabTarget:CreateButton({
    Name = "View Player (Spectate)",
    Callback = function()
        local target = Players:FindFirstChild(TargetPlayerName)
        if target and target.Character and target.Character:FindFirstChild("Humanoid") then
            Workspace.CurrentCamera.CameraSubject = target.Character.Humanoid
        end
    end,
})

TabTarget:CreateButton({
    Name = "Un-View (Voltar câmera)",
    Callback = function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
            Workspace.CurrentCamera.CameraSubject = LocalPlayer.Character.Humanoid
        end
    end,
})

TabTarget:CreateButton({
    Name = "Copiar Skin (Completa)",
    Callback = function()
        local target = Players:FindFirstChild(TargetPlayerName)
        if target and target.Character and LocalPlayer.Character then
            local myChar = LocalPlayer.Character
            local tChar = target.Character

            for _, v in pairs(myChar:GetChildren()) do
                if v:IsA("Accessory") or v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic") or v:IsA("BodyColors") or v:IsA("CharacterMesh") then
                    v:Destroy()
                end
            end

            for _, v in pairs(tChar:GetChildren()) do
                if v:IsA("Accessory") then
                    local clone = v:Clone()
                    myChar.Humanoid:AddAccessory(clone)
                elseif v:IsA("Shirt") or v:IsA("Pants") or v:IsA("ShirtGraphic") or v:IsA("BodyColors") or v:IsA("CharacterMesh") then
                    v:Clone().Parent = myChar
                end
            end

            local myHead = myChar:FindFirstChild("Head")
            local tHead = tChar:FindFirstChild("Head")
            if myHead and tHead then
                local myFace = myHead:FindFirstChildOfClass("Decal")
                local tFace = tHead:FindFirstChildOfClass("Decal")
                if myFace then myFace:Destroy() end
                if tFace then tFace:Clone().Parent = myHead end
                
                local myMesh = myHead:FindFirstChildOfClass("SpecialMesh")
                local tMesh = tHead:FindFirstChildOfClass("SpecialMesh")
                if myMesh then myMesh:Destroy() end
                if tMesh then tMesh:Clone().Parent = myHead end
            end

            Rayfield:Notify({
                Title = "Skin Copiada!",
                Content = "Avatar inteiro de " .. TargetPlayerName .. " foi clonado com sucesso.",
                Duration = 3,
                Image = "check",
            })
        end
    end,
})

TabTarget:CreateButton({
    Name = "Fling (Sofa Method Automático)",
    Callback = function()
        local target = Players:FindFirstChild(TargetPlayerName)
        local seat = LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid").SeatPart
        
        if not seat then
            Rayfield:Notify({Title = "Erro", Content = "Você precisa estar sentado em um item/carro!", Duration = 3})
            return
        end

        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local oldCFrame = seat.CFrame
            local spin = Instance.new("BodyAngularVelocity", seat)
            spin.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            spin.AngularVelocity = Vector3.new(0, 999999, 0)
            
            -- Pega e lança em < 0.1s
            for i = 1, 5 do
                seat.CFrame = target.Character.HumanoidRootPart.CFrame
                task.wait(0.01)
            end
            
            spin:Destroy()
            seat.CFrame = oldCFrame
        end
    end,
})

TabTarget:CreateButton({
    Name = "Kill (Void Fling Automático)",
    Callback = function()
        local target = Players:FindFirstChild(TargetPlayerName)
        local seat = LocalPlayer.Character:FindFirstChildWhichIsA("Humanoid").SeatPart
        
        if not seat then
            Rayfield:Notify({Title = "Erro", Content = "Você precisa estar sentado em um item/carro!", Duration = 3})
            return
        end

        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local oldCFrame = seat.CFrame
            local spin = Instance.new("BodyAngularVelocity", seat)
            spin.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            spin.AngularVelocity = Vector3.new(99999, 99999, 99999)
            
            -- Empurra para baixo do mapa
            for i = 1, 10 do
                seat.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, -5, 0)
                task.wait(0.01)
            end
            
            spin:Destroy()
            seat.CFrame = oldCFrame
        end
    end,
})

-- ==================== TAB: ÁUDIO ====================
local TabAudio = Window:CreateTab("Áudio", "music")

TabAudio:CreateInput({
    Name = "ID do Áudio",
    PlaceholderText = "Cole o ID da música...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        _G.AudioID = Text
    end,
})

TabAudio:CreateButton({
    Name = "Tocar no Alvo (Requer Boombox)",
    Callback = function()
        local char = LocalPlayer.Character
        local tool = char:FindFirstChildOfClass("Tool")
        if tool and tool:FindFirstChild("Handle") then
            local sound = tool.Handle:FindFirstChildOfClass("Sound") or Instance.new("Sound", tool.Handle)
            sound.SoundId = "rbxassetid://" .. tostring(_G.AudioID)
            sound.Volume = 10
            sound.Looped = true
            sound:Play()
            
            local target = Players:FindFirstChild(TargetPlayerName)
            if target and target.Character then
                char.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame
            end
        else
            Rayfield:Notify({
                Title = "Erro de Item",
                Content = "Equipe uma ferramenta (Boombox) na mão primeiro!",
                Duration = 3,
                Image = "x",
            })
        end
    end,
})
