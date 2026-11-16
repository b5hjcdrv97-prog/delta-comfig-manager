# delta-comfig-manager
https://sandbox:/mnt/data/separate_config_manager.lua

-- Standalone Config Manager for Delta
-- Not connected to KiciaHook's internal settings.

local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local player = Players.LocalPlayer

local ROOT = "KiciaHook_SeparateConfigs"

local function ensureFolder()
    if not isfolder(ROOT) then
        makefolder(ROOT)
    end
end

local function safeName(name)
    name = tostring(name or "")
    name = name:gsub("^%s+", ""):gsub("%s+$", "")
    name = name:gsub("[^%w_%-]", "_")
    return name ~= "" and name or nil
end

local function saveConfig(name, data)
    ensureFolder()
    local safe = safeName(name)
    assert(safe, "Enter a config name")
    writefile(
        ROOT .. "/" .. safe .. ".json",
        HttpService:JSONEncode(data)
    )
end

local function loadConfig(name)
    ensureFolder()
    local safe = safeName(name)
    assert(safe, "Enter a config name")
    local path = ROOT .. "/" .. safe .. ".json"
    assert(isfile(path), "Config not found")
    return HttpService:JSONDecode(readfile(path))
end

local function listConfigs()
    ensureFolder()
    local result = {}
    for _, file in ipairs(listfiles(ROOT)) do
        local name = file:match("([^/\\]+)%.json$")
        if name then
            table.insert(result, name)
        end
    end
    table.sort(result)
    return result
end

local function deleteConfig(name)
    ensureFolder()
    local safe = safeName(name)
    assert(safe, "Enter a config name")
    local path = ROOT .. "/" .. safe .. ".json"
    assert(isfile(path), "Config not found")
    delfile(path)
end

local settings = {
    crosshairEnabled = true,
    crosshairColor = "white",
    cameraMode = "Custom"
}

local gui = Instance.new("ScreenGui")
gui.Name = "SeparateConfigManager"
gui.ResetOnSpawn = false

local parent
pcall(function()
    parent = gethui()
end)

gui.Parent = parent or player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(310, 300)
frame.Position = UDim2.new(0, 20, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(27, 27, 32)
frame.Parent = gui

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 10)

local function button(label, x, y, callback)
    local b = Instance.new("TextButton")
    b.Size = UDim2.fromOffset(138, 38)
    b.Position = UDim2.fromOffset(x, y)
    b.Text = label
    b.TextColor3 = Color3.new(1, 1, 1)
    b.BackgroundColor3 = Color3.fromRGB(65, 75, 105)
    b.Parent = frame

    Instance.new("UICorner", b)

    b.MouseButton1Click:Connect(function()
        local ok, err = pcall(callback)
        if not ok then
            warn("[Config Manager] " .. tostring(err))
        end
    end)
end

local nameBox = Instance.new("TextBox")
nameBox.Size = UDim2.fromOffset(280, 36)
nameBox.Position = UDim2.fromOffset(15, 15)
nameBox.PlaceholderText = "Config name"
nameBox.Text = ""
nameBox.Parent = frame

local status = Instance.new("TextLabel")
status.Size = UDim2.fromOffset(280, 40)
status.Position = UDim2.fromOffset(15, 245)
status.Text = "Separate Config Manager"
status.TextColor3 = Color3.new(1, 1, 1)
status.BackgroundTransparency = 1
status.Parent = frame

button("Save Config", 12, 65, function()
    saveConfig(nameBox.Text, settings)
    status.Text = "Config saved!"
end)

button("Load Config", 160, 65, function()
    settings = loadConfig(nameBox.Text)
    status.Text = "Config loaded!"
    print(HttpService:JSONEncode(settings))
end)

button("List Configs", 12, 115, function()
    status.Text = table.concat(listConfigs(), ", ")
end)

button("Delete Config", 160, 115, function()
    deleteConfig(nameBox.Text)
    status.Text = "Config deleted!"
end)

button("Show Settings", 12, 165, function()
    print(HttpService:JSONEncode(settings))
    status.Text = "Settings printed!"
end)

button("Close", 160, 165, function()
    gui:Destroy()
end)

print("Separate Config Manager loaded!")
