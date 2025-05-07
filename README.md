local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- UI
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "DADAServers"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 350, 0, 400)
frame.Position = UDim2.new(0.5, -175, 0.5, -200)
frame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
frame.Active = true
frame.Draggable = true

local uicorner = Instance.new("UICorner", frame)
uicorner.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "🌐 DADA Server Browser"
title.TextColor3 = Color3.fromRGB(255, 255, 100)
title.Font = Enum.Font.GothamBold
title.TextScaled = true
title.BackgroundTransparency = 1

local refresh = Instance.new("TextButton", frame)
refresh.Size = UDim2.new(0.4, 0, 0, 30)
refresh.Position = UDim2.new(0.05, 0, 0, 50)
refresh.Text = "🔁 Обновить"
refresh.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
refresh.TextColor3 = Color3.fromRGB(255, 255, 255)
refresh.Font = Enum.Font.Gotham
refresh.TextScaled = true
Instance.new("UICorner", refresh).CornerRadius = UDim.new(0, 6)

local scroll = Instance.new("ScrollingFrame", frame)
scroll.Size = UDim2.new(1, -20, 1, -100)
scroll.Position = UDim2.new(0, 10, 0, 90)
scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
scroll.BackgroundTransparency = 1
scroll.ScrollBarThickness = 6

-- Функция запроса серверов
local function getServers()
	scroll:ClearAllChildren()
	local servers = {}
	local gameId = game.PlaceId
	local nextPage = "https://games.roblox.com/v1/games/"..gameId.."/servers/Public?sortOrder=Desc&limit=100"

	while nextPage do
		local success, result = pcall(function()
			return HttpService:JSONDecode(game:HttpGet(nextPage))
		end)
		if not success then break end
		for _, server in ipairs(result.data) do
			if server.playing < server.maxPlayers then
				table.insert(servers, server)
			end
		end
		nextPage = result.nextPageCursor and "https://games.roblox.com/v1/games/"..gameId.."/servers/Public?sortOrder=Desc&limit=100&cursor="..result.nextPageCursor or nil
	end

	-- Отображение
	local y = 0
	for i, server in ipairs(servers) do
		local serverFrame = Instance.new("Frame", scroll)
		serverFrame.Size = UDim2.new(1, -10, 0, 40)
		serverFrame.Position = UDim2.new(0, 0, 0, y)
		serverFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
		Instance.new("UICorner", serverFrame).CornerRadius = UDim.new(0, 6)

		local label = Instance.new("TextLabel", serverFrame)
		label.Size = UDim2.new(0.6, 0, 1, 0)
		label.Text = "👥 "..server.playing.."/"..server.maxPlayers.." | Ping: "..(server.ping or "???")
		label.BackgroundTransparency = 1
		label.TextColor3 = Color3.fromRGB(255, 255, 255)
		label.Font = Enum.Font.Gotham
		label.TextScaled = true

		local join = Instance.new("TextButton", serverFrame)
		join.Size = UDim2.new(0.35, 0, 0.8, 0)
		join.Position = UDim2.new(0.63, 0, 0.1, 0)
		join.Text = "Join"
		join.TextColor3 = Color3.fromRGB(255, 255, 100)
		join.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
		join.Font = Enum.Font.GothamBold
		join.TextScaled = true
		Instance.new("UICorner", join).CornerRadius = UDim.new(0, 6)

		join.MouseButton1Click:Connect(function()
			TeleportService:TeleportToPlaceInstance(gameId, server.id, player)
		end)

		y = y + 45
	end

	scroll.CanvasSize = UDim2.new(0, 0, 0, y + 10)
end

-- Кнопка обновить
refresh.MouseButton1Click:Connect(getServers)

-- Первый запуск
getServers()
# Server.lua
