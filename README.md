--[[

	Auto Kraken + Auto Fish (Pet Simulator 99!)
	Created by Baltazar.exe

]]

request, writefile, readfile = request, writefile, readfile

if not game:IsLoaded() then repeat task.wait() until game:IsLoaded() task.wait(math.random(3, 5) + math.random()) end

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local player = Players.LocalPlayer
local HRP = player.Character and player.Character:WaitForChild("HumanoidRootPart")
local placeId = game.PlaceId
local jobId = game.JobId

if player.PlayerGui:FindFirstChild("__INTRO") then
	repeat task.wait() until not player.PlayerGui:FindFirstChild("__INTRO")
end

local webhookUrl = "test" -- webhook do Discord
local autoFishingRunning = false
local autoFishingThread

-- Barra verde 100%
local FishGame = require(ReplicatedStorage.Library.Client.EventFishingCmds.Game)
if not FishGame._baltazarModified then
	FishGame._baltazarModified = true
	FishGame.BeginOld = FishGame.Begin
	FishGame.Begin = function(a, b, c)
		b.BarSize = 1
		return FishGame.BeginOld(a, b, c)
	end
end

-- Auto pesca com posição fixa debaixo d'água
local fishingCmds = require(ReplicatedStorage.Library.Client.EventFishingCmds)

local function startAutoFishingLoop()
	if autoFishingRunning then return end
	autoFishingRunning = true

	autoFishingThread = task.spawn(function()
		local x = math.random(13740, 13880)
		local z = math.random(400, 510)
		local underwaterPos = Vector3.new(x, -70, z)
		HRP.Anchored = true
		HRP.CFrame = CFrame.new(underwaterPos)

		while autoFishingRunning do
			local castPos = Vector3.new(x + math.random(-5, 5), 4.2, z + math.random(-5, 5))
			pcall(function()
				fishingCmds.LocalCast(castPos)
			end)
			task.wait(1)
		end
	end)
end

local function stopAutoFishingLoop()
	autoFishingRunning = false
	if autoFishingThread then
		task.cancel(autoFishingThread)
		autoFishingThread = nil
	end
end

local function createStatusGui(text, color, duration)
	local gui = Instance.new("ScreenGui")
	gui.Name = "KrakenStatusGui"
	gui.ResetOnSpawn = false
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	gui.Parent = player:WaitForChild("PlayerGui")

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(0, 300, 0, 50)
	label.Position = UDim2.new(0.5, -150, 0.1, 0)
	label.BackgroundTransparency = 0.3
	label.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	label.TextScaled = true
	label.Font = Enum.Font.SourceSansBold
	label.TextStrokeTransparency = 0
	label.TextColor3 = color
	label.Text = text
	label.Parent = gui

	if duration then
		task.delay(duration, function()
			if gui then gui:Destroy() end
		end)
	end

	return gui
end

local function tryDetectKraken()
	local folder = workspace:FindFirstChild("__THINGS") and workspace.__THINGS:FindFirstChild("EventFishingPOIs")
	local model = folder and folder:FindFirstChildWhichIsA("Model", true)
	return model and model:FindFirstChild("kraken_mesh")
end

local function sendWebhook()
	local username = player.Name
	local payload = HttpService:JSONEncode({
		content = "🐙 Kraken Found by ||" .. username .. "||"
	})
	local headers = { ["Content-Type"] = "application/json" }
	pcall(function()
		request({
			Url = webhookUrl,
			Method = "POST",
			Headers = headers,
			Body = payload
		})
	end)
end

local function attemptTeleportToKraken(target)
	for i = 1, 30 do
		pcall(function()
			HRP.Anchored = false
			HRP.CFrame = CFrame.new(target.Position.X, -70, target.Position.Z)
		end)
		task.wait(0.5)
		if math.abs(HRP.Position.Y + 70) < 10 then
			HRP.Anchored = true
			return true
		end
	end
	return false
end

local function startServerHopLoop()
	stopAutoFishingLoop()
	task.delay(5, function()
		local cursor = ""
		local allIDs = {}
		local currentHour = os.date("!*t").hour

		local function readCache()
			local ok, result = pcall(function()
				return HttpService:JSONDecode(readfile("NotSameServers.json"))
			end)
			if ok and result and tonumber(result[1]) == currentHour then
				return result
			else
				return { currentHour }
			end
		end

		local function writeCache()
			pcall(function()
				writefile("NotSameServers.json", HttpService:JSONEncode(allIDs))
			end)
		end

		allIDs = readCache()
		if not table.find(allIDs, jobId) then
			table.insert(allIDs, jobId)
		end
		writeCache()

		local function hop()
			local url = "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"
			if cursor ~= "" then url = url .. "&cursor=" .. cursor end

			local ok, data = pcall(function()
				return HttpService:JSONDecode(game:HttpGet(url))
			end)

			if not ok or typeof(data) ~= "table" or not data.data then
				warn("⚠️ Failed to fetch server list. Retrying...")
				return false
			end

			cursor = data.nextPageCursor or ""

			for _, server in ipairs(data.data) do
				local id = tostring(server.id)
				if server.playing < server.maxPlayers and not table.find(allIDs, id) then
					table.insert(allIDs, id)
					writeCache()
					local tpOk = pcall(function()
						TeleportService:TeleportToPlaceInstance(placeId, id, player)
					end)
					return tpOk
				end
			end

			return false
		end

		task.spawn(function()
			while true do
				local attempts = 0
				while attempts < 30 do
					attempts += 1
					print(string.format("[%s] Attempt #%d", os.date("%H:%M:%S"), attempts))
					if hop() then return end
					task.wait(3)
				end
				warn("❌ Hop failed after 30 tries. Retrying in 3s.")
				task.wait(3)
			end
		end)
	end)
end

local function startKrakenChecker()
	task.spawn(function()
		while true do
			task.wait(5)
			if not tryDetectKraken() then
				warn("⚠️ Kraken no longer detected. Stopping fishing and hopping...")
				startServerHopLoop()
				break
			else
				print("✅ Kraken still present.")
			end
		end
	end)
end

-- Posição inicial: spawn do barco
HRP.CFrame = CFrame.new(Vector3.new(12183.01, 10.62, -36.98))
task.wait(2)

pcall(function()
	ReplicatedStorage:WaitForChild("Network"):WaitForChild("Boats_RequestSpawn"):InvokeServer("Boat 1")
end)

-- Kraken detecção
local kraken = tryDetectKraken()
local gui

if kraken then
	gui = createStatusGui("Kraken found!", Color3.fromRGB(0, 255, 0))
	if attemptTeleportToKraken(kraken) then
		sendWebhook()
		startKrakenChecker()
		startAutoFishingLoop()
	else
		warn("❌ Failed to teleport to Kraken.")
		gui:Destroy()
		createStatusGui("Teleport failed. Hopping...", Color3.fromRGB(255, 0, 0), 4)
		startServerHopLoop()
	end
else
	gui = createStatusGui("Checking Kraken spawn points...", Color3.fromRGB(255, 165, 0), 4)
	local krakenPositions = {
		Vector3.new(13827.95, -80.62, 404.89),
		Vector3.new(13879.96, -80.15, 439.78),
		Vector3.new(13803.33, -80.15, 506.65),
		Vector3.new(13742.28, -80.15, 453.34)
	}
	local pos = krakenPositions[math.random(1, #krakenPositions)]
	HRP.CFrame = CFrame.new(pos)
	task.wait(3)

	kraken = tryDetectKraken()
	if kraken then
		gui:Destroy()
		createStatusGui("Kraken spawned!", Color3.fromRGB(0, 255, 0), 4)
		pcall(function()
			ReplicatedStorage:WaitForChild("Network"):WaitForChild("Boats_RequestSpawn"):InvokeServer("Boat 5")
		end)
		if attemptTeleportToKraken(kraken) then
			sendWebhook()
			startKrakenChecker()
			startAutoFishingLoop()
		else
			warn("❌ Failed to teleport after spawn.")
			gui:Destroy()
			createStatusGui("Teleport failed. Hopping...", Color3.fromRGB(255, 0, 0), 4)
			startServerHopLoop()
		end
	else
		gui:Destroy()
		createStatusGui("Kraken not found. Hopping...", Color3.fromRGB(255, 0, 0), 4)
		startServerHopLoop()
	end
end
