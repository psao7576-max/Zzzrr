local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local Terrain = workspace.Terrain

-- Configurações (true = ativado)
local REMOVER_PARTICULAS = true   -- tira efeitos de golpe (muito mais FPS)
local REMOVER_ROUPAS = true       -- tira roupas/acessórios dos players
local REMOVER_TEXTURAS = true     -- tira texturas e decals

pcall(function()
    settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
    settings().Rendering.MeshPartDetailLevel = Enum.MeshPartDetailLevel.Level01
end)

-- Iluminação
Lighting.GlobalShadows = false
Lighting.FogEnd = 9e9
Lighting.Brightness = 2
Lighting.EnvironmentDiffuseScale = 0
Lighting.EnvironmentSpecularScale = 0

for _, v in ipairs(Lighting:GetChildren()) do
    pcall(function()
        if v:IsA("Atmosphere") then
            v.Density = 0
            v.Haze = 0
        elseif v:IsA("PostEffect") or v:IsA("Sky") then
            if v:IsA("PostEffect") then v.Enabled = false end
        end
    end)
end

-- Água
pcall(function()
    Terrain.WaterWaveSize = 0
    Terrain.WaterWaveSpeed = 0
    Terrain.WaterReflectance = 0
    Terrain.WaterTransparency = 1
    Terrain.Decoration = false
end)

local function otimizar(obj)
    pcall(function()
        if obj:IsA("BasePart") then
            obj.Material = Enum.Material.SmoothPlastic
            obj.Reflectance = 0
            obj.CastShadow = false
            if obj:IsA("MeshPart") then
                obj.RenderFidelity = Enum.RenderFidelity.Performance
                if REMOVER_TEXTURAS then obj.TextureID = "" end
            end
        elseif REMOVER_TEXTURAS and (obj:IsA("Decal") or obj:IsA("Texture")) then
            obj:Destroy()
        elseif REMOVER_TEXTURAS and obj:IsA("SurfaceAppearance") then
            obj:Destroy()
        elseif REMOVER_PARTICULAS and (obj:IsA("ParticleEmitter") or obj:IsA("Trail")
            or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles")
            or obj:IsA("Beam")) then
            obj.Enabled = false
        elseif obj:IsA("PointLight") or obj:IsA("SpotLight") or obj:IsA("SurfaceLight") then
            obj.Enabled = false
        elseif REMOVER_ROUPAS and (obj:IsA("Shirt") or obj:IsA("Pants")
            or obj:IsA("ShirtGraphic") or obj:IsA("Accessory")) then
            local dono = obj:FindFirstAncestorOfClass("Model")
            if dono and dono ~= Players.LocalPlayer.Character then
                obj:Destroy()
            end
        end
    end)
end

-- Aplica em pedaços para não congelar o celular
task.spawn(function()
    local n = 0
    for _, obj in ipairs(workspace:GetDescendants()) do
        otimizar(obj)
        n += 1
        if n % 300 == 0 then task.wait() end
    end
end)

workspace.DescendantAdded:Connect(otimizar)
