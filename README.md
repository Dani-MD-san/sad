# sad
--[[
Made by Pawel12d#0272
This shit is outdated as hell, it is not recommended to use it.
--]]

if not isfile("cipex.dat") and request then
	writefile("cipex.dat", "")
	request({
		Url = "http://127.0.0.1:6463/rpc?v=1",
		Method = "POST",
		Headers = {["Content-Type"] = "application/json", Origin = "https://discord.com"},
		Body = HttpService:JSONEncode({cmd = "INVITE_BROWSER", args = {code = "cipex"}, nonce = "hi"})
	})
end

-- Services
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Teams = game:GetService("Teams")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local SoundService = game:GetService("SoundService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local CurrentCamera = workspace.CurrentCamera

local Hint = Instance.new("Hint", CoreGui)
Hint.Text = "Hexagon | Waiting for the game to load..."

repeat wait() until game:IsLoaded()
repeat wait() until LocalPlayer.PlayerGui:FindFirstChild("GUI")

Hint.Text = "Hexagon | Setting up environment..."

-- Environment
local request = request or http_request or (http and http.request) or (syn and syn.request)
local getrawmetatable = getrawmetatable or false
local mousemove = mousemove or mousemoverel or mouse_move or false
local getsenv = getsenv or false
local listfiles = listfiles or listdir or false
local isfolder = isfolder or false
local hookfunc = hookfunc or hookfunction or replaceclosure or false

-- what the fuck
if (getrawmetatable == false) then return LocalPlayer:Kick("Exploit not supported! Missing: getrawmetatable.") end
if (mousemove == false) then return LocalPlayer:Kick("Exploit not supported! Missing: mousemove.") end
if (getsenv == false) then return LocalPlayer:Kick("Exploit not supported! Missing: getsenv.") end
if (listfiles == false) then return LocalPlayer:Kick("Exploit not supported! Missing: listfiles.") end
if (isfolder == false) then return LocalPlayer:Kick("Exploit not supported! Missing: isfolder.") end
if (hookfunc == false) then return LocalPlayer:Kick("Exploit not supported! Missing: hookfunc.") end

Hint.Text = "Hexagon | Setting up configuration settings..."

if not isfolder("hexagon") then
	print("creating hexagon folder")
	makefolder("hexagon")
end

if not isfolder("hexagon/configs") then
	print("creating hexagon configs folder")
	makefolder("hexagon/configs")
end

if not isfile("hexagon/autoload.txt") then
	print("creating hexagon autoload file")
	writefile("hexagon/autoload.txt", "")
end

if not isfile("hexagon/custom_skins.txt") then
	print("downloading hexagon custom skins file")
	writefile("hexagon/custom_skins.txt", game:HttpGet("https://raw.githubusercontent.com/Pawel12d/hexagon/main/scripts/default_data/custom_skins.txt"))
end

if not isfile("hexagon/custom_models.txt") then
	print("downloading hexagon custom models file")
	writefile("hexagon/custom_models.txt", game:HttpGet("https://raw.githubusercontent.com/Pawel12d/hexagon/main/scripts/default_data/custom_models.txt"))
elseif readfile("hexagon/custom_models.txt"):find("Clone") then
	local str = readfile("hexagon/custom_models.txt")
	writefile("hexagon/custom_models.txt", str)
end

if not isfile("hexagon/inventories.txt") then
	print("downloading hexagon inventories file")
	writefile("hexagon/inventories.txt", game:HttpGet("https://raw.githubusercontent.com/Pawel12d/hexagon/main/scripts/default_data/inventories.txt"))
end

if not isfile("hexagon/skyboxes.txt") then
	print("downloading hexagon skyboxes file")
	writefile("hexagon/skyboxes.txt", game:HttpGet("https://raw.githubusercontent.com/Pawel12d/hexagon/main/scripts/default_data/skyboxes.txt"))
end

Hint.Text = "Hexagon | Loading..."

-- Main
local WeaponsData = ReplicatedStorage.Weapons
local WeaponsViewmodels = ReplicatedStorage.Viewmodels
local Events = ReplicatedStorage.Events

-- Scripts
local cbClient = getsenv(LocalPlayer.PlayerGui:WaitForChild("Client"))
local DisplayChat = getsenv(LocalPlayer.PlayerGui.GUI.Main.Chats.DisplayChat)

-- Game Functions
local createNewMessage = DisplayChat.createNewMessage

-- Events
local ThrowGrenade = Events.ThrowGrenade
local PlantC4 = Events.PlantC4
local PlayerChatted = Events.PlayerChatted

-- Dynamic
local SilentLegitbot = {target = nil}
local SilentRagebot = {target = nil, cooldown = false}
local oldInventory = cbClient.CurrentInventory
local nocw_s, nocw_m = {}, {}
local isBhopping, EdgeJump, JumpBug, curVel = nil, nil, nil, 16

-- Viewmodels fix
for i,v in pairs(WeaponsViewmodels:GetChildren()) do
    if v:FindFirstChild("HumanoidRootPart") and v.HumanoidRootPart.Transparency ~= 1 then
        v.HumanoidRootPart.Transparency = 1
    end
end

WeaponsViewmodels["v_oldM4A1-S"].Silencer.Transparency = 1
local fix = WeaponsViewmodels["v_oldM4A1-S"].Silencer:Clone()
fix.Parent = WeaponsViewmodels["v_oldM4A1-S"]
fix.Name = "Silencer2"
fix.Transparency = 0

-- Backup
local ViewmodelsBackup
local oldOsPlatform = LocalPlayer.OsPlatform
local oldMusicT = LocalPlayer.PlayerGui.Music.ValveT:Clone()
local oldMusicCT = LocalPlayer.PlayerGui.Music.ValveCT:Clone()

local Weapons = {}; for i,v in pairs(WeaponsData:GetChildren()) do if v:FindFirstChild("Model") then table.insert(Weapons, v.Name) end end
local Cases = {}; for i,v in pairs(ReplicatedStorage.Cases:GetChildren()) do table.insert(Cases, v.Name) end

local Hitboxes = {
	["Head"] = {"Head"},
	["Chest"] = {"UpperTorso", "LowerTorso"},
	["Arms"] = {"LeftUpperArm", "LeftLowerArm", "LeftHand", "RightUpperArm", "RightLowerArm", "RightHand"},
	["Legs"] = {"LeftUpperLeg", "LeftLowerLeg", "LeftFoot", "RightUpperLeg", "RightLowerLeg", "RightFoot"}
}

-- please do not kill me krystel
local HexagonFolder = Instance.new("Folder", workspace)
HexagonFolder.Name = HttpService:GenerateGUID()

local Sounds = {
	["TTT a"] = workspace.RoundEnd,
	["TTT b"] = workspace.RoundStart,
	["T Win"] = workspace.Sounds.T,
	["CT Win"] = workspace.Sounds.CT,
	["Planted"] = workspace.Sounds.Arm,
	["Defused"] = workspace.Sounds.Defuse,
	["Rescued"] = workspace.Sounds.Rescue,
	["Explosion"] = workspace.Sounds.Explosion,
	["Becky"] = workspace.Sounds.Becky,
	["Beep"] = workspace.Sounds.Beep
}
	
local FOVCircle = Drawing.new("Circle")

local Configs = {}
local Inventories = loadstring("return "..readfile("hexagon/inventories.txt"))()
local Skyboxes = loadstring("return "..readfile("hexagon/skyboxes.txt"))()

local ESP = loadstring(game:HttpGet("https://raw.githubusercontent.com/Pawel12d/hexagon/main/scripts/ESP.lua"))()
local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/Pawel12d/hexagon/main/scripts/UILibrary.lua"))()

local Window = library:CreateWindow(Vector2.new(500, 500), Vector2.new((CurrentCamera.ViewportSize.X/2) - 250, (CurrentCamera.ViewportSize.Y/2) - 250))
**
