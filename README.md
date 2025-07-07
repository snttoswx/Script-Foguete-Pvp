repeat wait() until game:IsLoaded()

_G.HeadSize = 20
_G.Disabled = true
_G.AimbotEnabled = false
_G.ESPLineEnabled = false
_G.AimPart = "Head"
_G.IFOV = 100
_G.InputFOV = 100

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ModMenu"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local toggle = Instance.new("ImageButton")
toggle.Name = "ToggleModMenu"
toggle.Size = UDim2.new(0, 50, 0, 50)
toggle.Position = UDim2.new(0, 10, 0, 10)
toggle.BackgroundTransparency = 1
toggle.Image = "rbxassetid://142520545"
toggle.Parent = screenGui

local frame = Instance.new("Frame")
frame.Name = "MainMenu"
frame.Size = UDim2.new(0, 250, 0, 270)
frame.Position = UDim2.new(0, 70, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundTransparency = 1
title.Text = "CRIADO POR SNTTOSWX"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.fromRGB(0 , 150, 255)
title.TextSize = 18
title.TextWrapped = true
title.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 8)
layout.FillDirection = Enum.FillDirection.Vertical
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = frame

local function createButton(name)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(0, 220, 0, 40)
	button.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.Gotham
	button.TextSize = 16
	button.Text = name
	button.Name = name
	Instance.new("UICorner", button).CornerRadius = UDim.new(0, 8)
	button.Parent = frame
	return button
end

local aimbotBtn = createButton("Xit: OFF")
local espBtn = createButton("ESP-LIN: OFF")
local disableBtn = createButton("Desligar o Xit")

toggle.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

aimbotBtn.MouseButton1Click:Connect(function()
	_G.AimbotEnabled = not _G.AimbotEnabled
	aimbotBtn.Text = "Xit: " .. (_G.AimbotEnabled and "ON" or "OFF")
end)

espBtn.MouseButton1Click:Connect(function()
	_G.ESPLineEnabled = not _G.ESPLineEnabled
	espBtn.Text = "ESP-LIN: " .. (_G.ESPLineEnabled and "ON" or "OFF")
end)

disableBtn.MouseButton1Click:Connect(function()
	_G.AimbotEnabled = false
	_G.ESPLineEnabled = false
	_G.Disabled = false
	aimbotBtn.Text = "Xit: OFF"
	espBtn.Text = "ESP-LIN: OFF"
end)

local inputLabel = Instance.new("TextLabel")
inputLabel.Size = UDim2.new(0, 220, 0, 20)
inputLabel.BackgroundTransparency = 1
inputLabel.Text = "Digite o % do iFOV (ex: 20, 50, 100):"
inputLabel.Font = Enum.Font.Gotham
inputLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
inputLabel.TextSize = 14
inputLabel.Parent = frame

local inputBox = Instance.new("TextBox")
inputBox.Size = UDim2.new(0, 220, 0, 30)
inputBox.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
inputBox.Font = Enum.Font.Gotham
inputBox.TextSize = 16
inputBox.PlaceholderText = tostring(_G.InputFOV)
inputBox.ClearTextOnFocus = false
inputBox.Parent = frame
Instance.new("UICorner", inputBox).CornerRadius = UDim.new(0, 8)

inputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local num = tonumber(inputBox.Text)
        if num and num >= 0 and num <= 100 then
            _G.InputFOV = num
            if num <= 30 then
                _G.AimPart = "LeftFoot" -- mira na perna
            elseif num <= 70 then
                -- corrigido: usar "UpperTorso" para pegar o peito corretamente
                _G.AimPart = "UpperTorso"
            else
                _G.AimPart = "Head" -- mira na cabeça
            end
        else
            inputBox.Text = tostring(_G.InputFOV)
        end
    end
end)

local function applyHitbox(player)
	pcall(function()
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local hrp = player.Character.HumanoidRootPart
			hrp.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
			hrp.Transparency = 0.7
			hrp.BrickColor = BrickColor.new("Really white")
			hrp.Material = Enum.Material.Neon
			hrp.CanCollide = false
		end
	end)
end

local function createESP(player)
	if not player.Character then return end
	local hrp = player.Character:FindFirstChild("HumanoidRootPart")
	local head = player.Character:FindFirstChild("Head")
	if not hrp or not head then return end
	if hrp:FindFirstChild("ESP_Beam") then return end

	local camAttachment = Instance.new("Attachment", Camera)
	camAttachment.Name = "CamAttachment_" .. player.Name
	local hrpAttachment = Instance.new("Attachment", hrp)

	local beam = Instance.new("Beam")
	beam.Attachment0 = camAttachment
	beam.Attachment1 = hrpAttachment
	beam.Name = "ESP_Beam"
	beam.Color = ColorSequence.new(Color3.new(1, 1, 1))
	beam.Width0 = 0.25
	beam.Width1 = 0.25
	beam.FaceCamera = true
	beam.LightEmission = 1
	beam.Transparency = NumberSequence.new(0)
	beam.Parent = hrp

	local billboard = Instance.new("BillboardGui", head)
	billboard.Name = "ESP_Tag"
	billboard.Size = UDim2.new(0, 200, 0, 50)
	billboard.AlwaysOnTop = true
	billboard.StudsOffset = Vector3.new(0, 3, 0)

	local textLabel = Instance.new("TextLabel", billboard)
	textLabel.Size = UDim2.new(1, 0, 1, 0)
	textLabel.BackgroundTransparency = 1
	textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	textLabel.TextStrokeTransparency = 0
	textLabel.TextSize = 16
	textLabel.Font = Enum.Font.GothamBold
	textLabel.Text = player.Name .. " | 🤍 " .. math.floor(player.Character:FindFirstChild("Humanoid").Health)
end

local function updateESP(player)
	if player.Character and player.Character:FindFirstChild("Head") then
		local tag = player.Character.Head:FindFirstChild("ESP_Tag")
		if tag and tag:FindFirstChildOfClass("TextLabel") then
			tag.TextLabel.Text = player.Name .. " | 🤍 " .. math.floor(player.Character.Humanoid.Health)
		end
	end
end

local function clearESP(player)
	if player.Character then
		local hrp = player.Character:FindFirstChild("HumanoidRootPart")
		if hrp and hrp:FindFirstChild("ESP_Beam") then hrp.ESP_Beam:Destroy() end
		local head = player.Character:FindFirstChild("Head")
		if head and head:FindFirstChild("ESP_Tag") then head.ESP_Tag:Destroy() end
	end
	for _, v in pairs(Camera:GetChildren()) do
		if v:IsA("Attachment") and v.Name:find("CamAttachment_") then v:Destroy() end
	end
end

local function getClosestPlayer()
	local closest = nil
	local shortest = math.huge
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild(_G.AimPart) then
			local pos, onScreen = Camera:WorldToScreenPoint(p.Character[_G.AimPart].Position)
			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
				if dist < shortest and dist <= _G.IFOV then
					shortest = dist
					closest = p
				end
			end
		end
	end
	return closest
end

local function aimAt(target)
	if target and target.Character and target.Character:FindFirstChild(_G.AimPart) then
		local pos = target.Character[_G.AimPart].Position
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, pos)
	end
end

RunService.RenderStepped:Connect(function()
	if _G.Disabled then
		for _, p in ipairs(Players:GetPlayers()) do
			if p ~= LocalPlayer then
				applyHitbox(p)
				if _G.ESPLineEnabled then
					createESP(p)
					updateESP(p)
				else
					clearESP(p)
				end
			end
		end
		if _G.AimbotEnabled then
			local target = getClosestPlayer()
			aimAt(target)
		end
	end
end)repeat wait() until game:IsLoaded()

_G.HeadSize = 20
_G.Disabled = true
_G.AimbotEnabled = false
_G.ESPLineEnabled = false
_G.AimPart = "Head"
_G.IFOV = 100
_G.InputFOV = 100

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "ModMenu"
screenGui.ResetOnSpawn = false
screenGui.Parent = game:GetService("CoreGui")

local toggle = Instance.new("ImageButton")
toggle.Name = "ToggleModMenu"
toggle.Size = UDim2.new(0, 50, 0, 50)
toggle.Position = UDim2.new(0, 10, 0, 10)
toggle.BackgroundTransparency = 1
toggle.Image = "rbxassetid://142520545"
toggle.Parent = screenGui

local frame = Instance.new("Frame")
frame.Name = "MainMenu"
frame.Size = UDim2.new(0, 250, 0, 270)
frame.Position = UDim2.new(0, 70, 0, 10)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
frame.BorderSizePixel = 0
frame.Visible = false
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 35)
title.BackgroundTransparency = 1
title.Text = "CRIADO POR SNTTOSWX"
title.Font = Enum.Font.GothamBold
title.TextColor3 = Color3.fromRGB(0 , 150, 255)
title.TextSize = 18
title.TextWrapped = true
title.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 8)
layout.FillDirection = Enum.FillDirection.Vertical
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = frame

local function createButton(name)
	local button = Instance.new("TextButton")
	button.Size = UDim2.new(0, 220, 0, 40)
	button.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Font = Enum.Font.Gotham
	button.TextSize = 16
	button.Text = name
	button.Name = name
	Instance.new("UICorner", button).CornerRadius = UDim.new(0, 8)
	button.Parent = frame
	return button
end

local aimbotBtn = createButton("Xit: OFF")
local espBtn = createButton("ESP-LIN: OFF")
local disableBtn = createButton("Desligar o Xit")

toggle.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

aimbotBtn.MouseButton1Click:Connect(function()
	_G.AimbotEnabled = not _G.AimbotEnabled
	aimbotBtn.Text = "Xit: " .. (_G.AimbotEnabled and "ON" or "OFF")
end)

espBtn.MouseButton1Click:Connect(function()
	_G.ESPLineEnabled = not _G.ESPLineEnabled
	espBtn.Text = "ESP-LIN: " .. (_G.ESPLineEnabled and "ON" or "OFF")
end)

disableBtn.MouseButton1Click:Connect(function()
	_G.AimbotEnabled = false
	_G.ESPLineEnabled = false
	_G.Disabled = false
	aimbotBtn.Text = "Xit: OFF"
	espBtn.Text = "ESP-LIN: OFF"
end)

local inputLabel = Instance.new("TextLabel")
inputLabel.Size = UDim2.new(0, 220, 0, 20)
inputLabel.BackgroundTransparency = 1
inputLabel.Text = "Digite o % do iFOV (ex: 20, 50, 100):"
inputLabel.Font = Enum.Font.Gotham
inputLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
inputLabel.TextSize = 14
inputLabel.Parent = frame

local inputBox = Instance.new("TextBox")
inputBox.Size = UDim2.new(0, 220, 0, 30)
inputBox.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
inputBox.TextColor3 = Color3.fromRGB(255, 255, 255)
inputBox.Font = Enum.Font.Gotham
inputBox.TextSize = 16
inputBox.PlaceholderText = tostring(_G.InputFOV)
inputBox.ClearTextOnFocus = false
inputBox.Parent = frame
Instance.new("UICorner", inputBox).CornerRadius = UDim.new(0, 8)

inputBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local num = tonumber(inputBox.Text)
        if num and num >= 0 and num <= 100 then
            _G.InputFOV = num
            if num <= 30 then
                _G.AimPart = "LeftFoot" -- mira na perna
            elseif num <= 70 then
                -- corrigido: usar "UpperTorso" para pegar o peito corretamente
                _G.AimPart = "UpperTorso"
            else
                _G.AimPart = "Head" -- mira na cabeça
            end
        else
            inputBox.Text = tostring(_G.InputFOV)
        end
    end
end)

local function applyHitbox(player)
	pcall(function()
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local hrp = player.Character.HumanoidRootPart
			hrp.Size = Vector3.new(_G.HeadSize, _G.HeadSize, _G.HeadSize)
			hrp.Transparency = 0.7
			hrp.BrickColor = BrickColor.new("Really white")
			hrp.Material = Enum.Material.Neon
			hrp.CanCollide = false
		end
	end)
end

local function createESP(player)
	if not player.Character then return end
	local hrp = player.Character:FindFirstChild("HumanoidRootPart")
	local head = player.Character:FindFirstChild("Head")
	if not hrp or not head then return end
	if hrp:FindFirstChild("ESP_Beam") then return end

	local camAttachment = Instance.new("Attachment", Camera)
	camAttachment.Name = "CamAttachment_" .. player.Name
	local hrpAttachment = Instance.new("Attachment", hrp)

	local beam = Instance.new("Beam")
	beam.Attachment0 = camAttachment
	beam.Attachment1 = hrpAttachment
	beam.Name = "ESP_Beam"
	beam.Color = ColorSequence.new(Color3.new(1, 1, 1))
	beam.Width0 = 0.25
	beam.Width1 = 0.25
	beam.FaceCamera = true
	beam.LightEmission = 1
	beam.Transparency = NumberSequence.new(0)
	beam.Parent = hrp

	local billboard = Instance.new("BillboardGui", head)
	billboard.Name = "ESP_Tag"
	billboard.Size = UDim2.new(0, 200, 0, 50)
	billboard.AlwaysOnTop = true
	billboard.StudsOffset = Vector3.new(0, 3, 0)

	local textLabel = Instance.new("TextLabel", billboard)
	textLabel.Size = UDim2.new(1, 0, 1, 0)
	textLabel.BackgroundTransparency = 1
	textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	textLabel.TextStrokeTransparency = 0
	textLabel.TextSize = 16
	textLabel.Font = Enum.Font.GothamBold
	textLabel.Text = player.Name .. " | 🤍 " .. math.floor(player.Character:FindFirstChild("Humanoid").Health)
end

local function updateESP(player)
	if player.Character and player.Character:FindFirstChild("Head") then
		local tag = player.Character.Head:FindFirstChild("ESP_Tag")
		if tag and tag:FindFirstChildOfClass("TextLabel") then
			tag.TextLabel.Text = player.Name .. " | 🤍 " .. math.floor(player.Character.Humanoid.Health)
		end
	end
end

local function clearESP(player)
	if player.Character then
		local hrp = player.Character:FindFirstChild("HumanoidRootPart")
		if hrp and hrp:FindFirstChild("ESP_Beam") then hrp.ESP_Beam:Destroy() end
		local head = player.Character:FindFirstChild("Head")
		if head and head:FindFirstChild("ESP_Tag") then head.ESP_Tag:Destroy() end
	end
	for _, v in pairs(Camera:GetChildren()) do
		if v:IsA("Attachment") and v.Name:find("CamAttachment_") then v:Destroy() end
	end
end

local function getClosestPlayer()
	local closest = nil
	local shortest = math.huge
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild(_G.AimPart) then
			local pos, onScreen = Camera:WorldToScreenPoint(p.Character[_G.AimPart].Position)
			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
				if dist < shortest and dist <= _G.IFOV then
					shortest = dist
					closest = p
				end
			end
		end
	end
	return closest
end

local function aimAt(target)
	if target and target.Character and target.Character:FindFirstChild(_G.AimPart) then
		local pos = target.Character[_G.AimPart].Position
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, pos)
	end
end

RunService.RenderStepped:Connect(function()
	if _G.Disabled then
		for _, p in ipairs(Players:GetPlayers()) do
			if p ~= LocalPlayer then
				applyHitbox(p)
				if _G.ESPLineEnabled then
					createESP(p)
					updateESP(p)
				else
					clearESP(p)
				end
			end
		end
		if _G.AimbotEnabled then
			local target = getClosestPlayer()
			aimAt(target)
		end
	end
end)
