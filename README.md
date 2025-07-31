Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local ESPConfig = {
    Box = false,
    Distance = false,
    Chams = false,
    Line = false,
    Name = false,
    Beast = false,
    PC = false,
    Cor = Color3.fromRGB(255, 0, 0)
}

local Window = Rayfield:CreateWindow({
    Name = "VisionX ESP Suite",
    LoadingTitle = "Iniciando VisionX...",
    LoadingSubtitle = "by VisionX",
    ShowText = "VisionX",
    Theme = "Default",
    ToggleUIKeybind = "K",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "VisionX_Config",
        FileName = "Config"
    },
    Discord = { Enabled = false },
    KeySystem = false
})

local Tab = Window:CreateTab("ESP Options", 4483362458)
Tab:CreateSection("Ativações")

local options = { "Box", "Distance", "Chams", "Line", "Name", "Beast", "PC" }
for _, opt in ipairs(options) do
    Tab:CreateToggle({
        Name = "ESP " .. opt,
        CurrentValue = false,
        Flag = "ESP_" .. opt,
        Callback = function(state)
            ESPConfig[opt] = state
        end
    })
end

Tab:CreateColorPicker({
    Name = "Cor do ESP",
    Color = ESPConfig.Cor,
    Flag = "ESPColor",
    Callback = function(color)
        ESPConfig.Cor = color
    end
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Beast + PCs
local function updatePCs()
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj.Name == "ComputerTable" and obj:IsA("Model") then
            local target = obj:FindFirstChild("ComputerScreen") or obj:FindFirstChildWhichIsA("BasePart")
            if ESPConfig.PC and target and not obj:FindFirstChild("PC_Highlight") then
                local hl = Instance.new("Highlight", obj)
                hl.Name = "PC_Highlight"
                hl.FillColor = ESPConfig.Cor
                hl.OutlineTransparency = 0.2
                hl.FillTransparency = 0.4
                hl.Adornee = obj
                hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            elseif not ESPConfig.PC and obj:FindFirstChild("PC_Highlight") then
                obj.PC_Highlight:Destroy()
            end
        end
    end
end

local function updateBeastESP()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr:FindFirstChild("Character") then
            local isBeast = plr:FindFirstChild("PlayerData") and plr.PlayerData:FindFirstChild("Beast") and plr.PlayerData.Beast.Value
            local highlight = plr.Character:FindFirstChild("BeastHighlight")
            if ESPConfig.Beast and isBeast and not highlight then
                local hl = Instance.new("Highlight", plr.Character)
                hl.Name = "BeastHighlight"
                hl.FillColor = Color3.fromRGB(255, 0, 0)
                hl.OutlineTransparency = 1
            elseif not ESPConfig.Beast and highlight then
                highlight:Destroy()
            end
        end
    end
end

-- Players
RunService.Heartbeat:Connect(function()
    updatePCs()
    updateBeastESP()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local root = player.Character:FindFirstChild("HumanoidRootPart")

            -- Box
            if ESPConfig.Box and not player.Character:FindFirstChild("ESP_Box") then
                local box = Instance.new("BoxHandleAdornment", player.Character)
                box.Name = "ESP_Box"
                box.Adornee = player.Character
                box.Size = Vector3.new(4, 5, 2)
                box.Color3 = ESPConfig.Cor
                box.AlwaysOnTop = true
                box.ZIndex = 5
                box.Transparency = 0.3
            elseif not ESPConfig.Box and player.Character:FindFirstChild("ESP_Box") then
                player.Character.ESP_Box:Destroy()
            end

            -- Chams
            if ESPConfig.Chams and not player.Character:FindFirstChild("Highlight") then
                local highlight = Instance.new("Highlight", player.Character)
                highlight.Name = "Highlight"
                highlight.FillColor = ESPConfig.Cor
                highlight.OutlineTransparency = 1
            elseif not ESPConfig.Chams and player.Character:FindFirstChild("Highlight") then
                player.Character.Highlight:Destroy()
            end

            -- Nome/Distância
            if ESPConfig.Name or ESPConfig.Distance then
                local gui = head:FindFirstChild("ESPLabel") or Instance.new("BillboardGui", head)
                gui.Name = "ESPLabel"
                gui.Size = UDim2.new(0, 200, 0, 40)
                gui.AlwaysOnTop = true
                gui.Adornee = head

                local label = gui:FindFirstChild("Text") or Instance.new("TextLabel", gui)
                label.Name = "Text"
                label.Size = UDim2.new(1, 0, 1, 0)
                label.BackgroundTransparency = 1
                label.TextScaled = true
                label.TextColor3 = ESPConfig.Cor
                label.Font = Enum.Font.SourceSansBold

                local dist = math.floor((head.Position - Camera.CFrame.Position).Magnitude)
                local txt = ESPConfig.Name and player.Name or ""
                txt = txt .. (ESPConfig.Distance and (" | " .. dist .. "m") or "")
                label.Text = txt
            elseif head:FindFirstChild("ESPLabel") then
                head.ESPLabel:Destroy()
            end

            -- Linha
            if ESPConfig.Line and root then
                local a0 = root:FindFirstChild("ESP_Attach") or Instance.new("Attachment", root)
                a0.Name = "ESP_Attach"
                local a1 = Camera:FindFirstChild("ESP_CamAttach") or Instance.new("Attachment", Camera)
                a1.Name = "ESP_CamAttach"

                local beam = root:FindFirstChild("ESP_Beam") or Instance.new("Beam", root)
                beam.Name = "ESP_Beam"
                beam.Attachment0 = a0
                beam.Attachment1 = a1
                beam.Width0 = 0.1
                beam.Width1 = 0.1
                beam.Color = ColorSequence.new(ESPConfig.Cor)
                beam.FaceCamera = true
            elseif root and root:FindFirstChild("ESP_Beam") and not ESPConfig.Line then
                root.ESP_Beam:Destroy()
                if root:FindFirstChild("ESP_Attach") then root.ESP_Attach:Destroy() end
                if Camera:FindFirstChild("ESP_CamAttach") then Camera.ESP_CamAttach:Destroy() end
            end
        end
    end
end)
