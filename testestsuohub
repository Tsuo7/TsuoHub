local workspace = game:GetService("Workspace")
local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")
local localPlayer = players.LocalPlayer
local BASE_THRESHOLD = 0.2
local VELOCITY_SCALING_FACTOR_FAST = 0.050
local VELOCITY_SCALING_FACTOR_SLOW = 0.1
local IMMEDIATE_PARRY_DISTANCE = 11
local IMMEDIATE_HIGH_VELOCITY_THRESHOLD = 85
local UserInputService = game:GetService("UserInputService")
local heartbeatConnection
local focusedBall, displayBall = nil, nil
local character = localPlayer.Character or localPlayer.CharacterAdded:Wait()
local ballsFolder = workspace:WaitForChild("Balls")
local parryButtonPress = replicatedStorage.Remotes.ParryButtonPress
local abilityButtonPress = replicatedStorage.Remotes.AbilityButtonPress
local sliderValue = 25
local distanceVisualizer = nil
local isRunning = false
local notifyparried = false
local PlayerGui = localPlayer:WaitForChild("PlayerGui")
local Hotbar = PlayerGui:WaitForChild("Hotbar")
local UseRage = false

local uigrad1 = Hotbar.Block.border1.UIGradient
local uigrad2 = Hotbar.Ability.border2.UIGradient

local function isPlayerOnMobile()
    return UserInputService.TouchEnabled and not (UserInputService.KeyboardEnabled or UserInputService.GamepadEnabled)
end

local function onCharacterAdded(newCharacter)
    character = newCharacter
    abilitiesFolder = character:WaitForChild("Abilities")
end

localPlayer.CharacterAdded:Connect(onCharacterAdded)

local TruValue = Instance.new("StringValue")
if workspace:FindFirstChild("AbilityThingyk1212") then
    workspace:FindFirstChild("AbilityThingyk1212"):Remove()
    task.wait(0.1)
    TruValue.Parent = game:GetService("Workspace")
        TruValue.Name = "AbilityThingyk1212"
        TruValue.Value = "Dash"
    else
        TruValue.Parent = game:GetService("Workspace")
        TruValue.Name = "AbilityThingyk1212"
        TruValue.Value = "Dash"
end

local RayfieldURL = isPlayerOnMobile() and 
                    'https://raw.githubusercontent.com/hungquan99/Arrayfield/main/NewUI' or 
                    'https://raw.githubusercontent.com/hungquan99/Arrayfield/main/NewUI'

local Rayfield = loadstring(game:HttpGet(RayfieldURL))()


local Window = Rayfield:CreateWindow({
   Name = "Hung Hub Premium | Blade Ball",
   LoadingTitle = "Hung Hub Premium",
   LoadingSubtitle = "by hungquan99",
   ConfigurationSaving = {
      Enabled = false,
      FolderName = "Hung Hub Premium",
      FileName = "Blade Ball"
   },
   Discord = {
      Enabled = true,
      Invite = "gNKfqqjYvg",
      RememberJoins = true
   },
   KeySystem = false,
   KeySettings = {
      Title = "Hung Hub",
      Subtitle = "Key System",
      Note = "",
      FileName = "HungKeySystem",
      SaveKey = true,
      GrabKeyFromSite = false,
      Key = "anbatukam"
   }
})

local AutoParry = Window:CreateTab("Main", 14467433545)
local Ability = Window:CreateTab("Abilities", 14467433545)
local Misc = Window:CreateTab("Misc", 14467433545)

if character then
    print("Character found | Hung Hub Premium")
else
    print("Character not found | Hung Hub Premium")
    return
end

local function notify(title, content, duration)
    Rayfield:Notify({
        Title = title,
        Content = content,
        Duration = duration or 0.7,
        Image = 14467433545
    })
end

local function chooseNewFocusedBall()
    local balls = ballsFolder:GetChildren()
    for _, ball in ipairs(balls) do
        if ball:GetAttribute("realBall") ~= nil and ball:GetAttribute("realBall") == true then
            focusedBall = ball
            print(focusedBall.Name)
            break
        elseif ball:GetAttribute("target") ~= nil then
            focusedBall = ball
            print(focusedBall.Name)
            break
        end
    end
    
    if focusedBall == nil then
        print("Debug: Could not find a ball that's the realBall or has a target")
        wait(1)
        chooseNewFocusedBall()
    end
    return focusedBall
end

local function getDynamicThreshold(ballVelocityMagnitude)
    if ballVelocityMagnitude > 60 then
        return math.max(0.20, BASE_THRESHOLD - (ballVelocityMagnitude * VELOCITY_SCALING_FACTOR_FAST))
    else
        return math.min(0.01, BASE_THRESHOLD + (ballVelocityMagnitude * VELOCITY_SCALING_FACTOR_SLOW))
    end
end

local function timeUntilImpact(ballVelocity, distanceToPlayer, playerVelocity)
    if not character then return end
    local directionToPlayer = (character.HumanoidRootPart.Position - focusedBall.Position).Unit
    local velocityTowardsPlayer = ballVelocity:Dot(directionToPlayer) - playerVelocity:Dot(directionToPlayer)
    
    if velocityTowardsPlayer <= 0 then
        return math.huge
    end
    
    return (distanceToPlayer - sliderValue) / velocityTowardsPlayer
end

local function updateDistanceVisualizer()
    local charPos = character and character.PrimaryPart and character.PrimaryPart.Position
    if charPos and focusedBall then
        if distanceVisualizer then
            distanceVisualizer:Destroy()
        end

        local timeToImpactValue = timeUntilImpact(focusedBall.Velocity, (focusedBall.Position - charPos).Magnitude, character.PrimaryPart.Velocity)
        local ballFuturePosition = focusedBall.Position + focusedBall.Velocity * timeToImpactValue

        distanceVisualizer = Instance.new("Part")
        distanceVisualizer.Size = Vector3.new(1, 1, 1)
        distanceVisualizer.Anchored = true
        distanceVisualizer.CanCollide = false
        distanceVisualizer.Position = ballFuturePosition
        distanceVisualizer.Parent = workspace    
    end
end

local function checkIfTarget()
    for _, v in pairs(ballsFolder:GetChildren()) do
        if v:IsA("Part") and v.BrickColor == BrickColor.new("Really red") then 
            print("Ball is targetting player | Hung Hub Premium")
            return true 
        end 
    end 
    return false
end

local function isCooldownInEffect(uigradient)
    return uigradient.Offset.Y < 0.5
end


local function checkBallDistance()
    if not character or not checkIfTarget() then return end

    local charPos = character.PrimaryPart.Position
    local charVel = character.PrimaryPart.Velocity

    if focusedBall and not focusedBall.Parent then
        print("Focused ball lost parent, choosing a new focused ball | Hung Hub Premium")
        chooseNewFocusedBall()
    end
    if not focusedBall then 
        print("No focused ball | Hung Hub Premium")
        chooseNewFocusedBall()
    end

    local ball = focusedBall
    local distanceToPlayer = (ball.Position - charPos).Magnitude
    local ballVelocityTowardsPlayer = ball.Velocity:Dot((charPos - ball.Position).Unit)
    
    if distanceToPlayer < 15 then
        parryButtonPress:Fire()
        task.wait()
    end

    if timeUntilImpact(ball.Velocity, distanceToPlayer, charVel) < getDynamicThreshold(ballVelocityTowardsPlayer) then
        if (character.Abilities["Raging Deflection"].Enabled or character.Abilities["Rapture"].Enabled) and UseRage == true then
            if not isCooldownInEffect(uigrad2) then
                abilityButtonPress:Fire()
            end

            if isCooldownInEffect(uigrad2) and not isCooldownInEffect(uigrad1) then
                parryButtonPress:Fire()
                if notifyparried == true then
                    local NotificationHolder = loadstring(game:HttpGet("https://raw.githubusercontent.com/BocusLuke/UI/main/STX/Module.Lua"))()
                    local Notification = loadstring(game:HttpGet("https://raw.githubusercontent.com/BocusLuke/UI/main/STX/Client.Lua"))()
                    local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "Manually Parried Ball!",
                        Duration = 1,
                        Icon = "rbxassetid://14467433545"
                    })
                end
            end

        elseif not isCooldownInEffect(uigrad1) then
            print(isCooldownInEffect(uigrad1))
            parryButtonPress:Fire()
            if notifyparried == true then
                local StarterGui = game:GetService("StarterGui")
                StarterGui:SetCore("SendNotification", {
                    Title = "Hung Hub",
                    Text = "Automatically Parried Ball!",
                    Duration = 1,
                    Icon = "rbxassetid://14467433545"
                })
            end
            task.wait(0.3)
        end
    end
end


local function autoParryCoroutine()
    while isRunning do
        checkBallDistance()
        updateDistanceVisualizer()
        task.wait()
    end
end



localPlayer.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    chooseNewFocusedBall()
    updateDistanceVisualizer()
end)

localPlayer.CharacterRemoving:Connect(function()
    if distanceVisualizer then
        distanceVisualizer:Destroy()
        distanceVisualizer = nil
    end
end)



local function startAutoParry()
    print("Started Auto Parry | Hung Hub Premium")
    
    chooseNewFocusedBall()
    
    isRunning = true
    local co = coroutine.create(autoParryCoroutine)
    coroutine.resume(co)
end

local function stopAutoParry()
    isRunning = false
end

local parryon = false
local autoparrydistance = 11

local Debug = false -- Set this to true if you want my debug output.
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 9e9) -- A second argument in waitforchild what could it mean?
local Balls = workspace:WaitForChild("Balls", 9e9)

-- Functions

local function print(...) -- Debug print.
    if Debug then
        warn(...)
    end
end

local function VerifyBall(Ball) -- Returns nil if the ball isn't a valid projectile; true if it's the right ball.
    if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true then
        return true
    end
end

local function IsTarget() -- Returns true if we are the current target.
    return (Player.Character and Player.Character:FindFirstChild("Highlight"))
end

local function Parry() -- Parries.
    Remotes:WaitForChild("ParryButtonPress"):Fire()
end
-- The actual code

Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return
    end
    
    print("Ball Spawned: {Ball}")
    
    local OldPosition = Ball.Position
    local OldTick = tick()
    
    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        if IsTarget() then
            local Distance = (Ball.Position - workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude 
            
            print("Distance: {Distance}\nVelocity: {Velocity}\nTime: {Distance / Velocity}")
        
            if (Distance / Velocity) <= autoparrydistance then
                if parryon == true then
                    Parry()
                end
            end
        end
        
        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)

local InfoSection = AutoParry:CreateSection("Info")

local DiscordButton = AutoParry:CreateButton({
    Name = "Discord Server",
    SectionParent = InfoSection,
    Callback = function()
        setclipboard("https://discord.gg/zDVbpyJ3VX")
        local NotificationHolder = loadstring(game:HttpGet("https://raw.githubusercontent.com/BocusLuke/UI/main/STX/Module.Lua"))()
        local Notification = loadstring(game:HttpGet("https://raw.githubusercontent.com/BocusLuke/UI/main/STX/Client.Lua"))()
        local StarterGui = game:GetService("StarterGui")
            StarterGui:SetCore("SendNotification", {
                Title = "Hung Hub",
                Text = "Copied Hung Hub discord invite link!",
                Duration = 5,
                Icon = "rbxassetid://14467433545"
            })
    end,
})


local AutoParrySection = AutoParry:CreateSection("Parry")

local AutoParryToggle = AutoParry:CreateToggle({
    Name = "Auto Parry Normal (Better Than Free Version)",
    CurrentValue = false,
    Flag = "AutoParryV1Flag",
    SectionParent = AutoParrySection,
    Callback = function(Value)
        if Value then
            parryon = Value
            startAutoParry()
            local StarterGui = game:GetService("StarterGui")
            StarterGui:SetCore("SendNotification", {
                Title = "Hung Hub",
                Text = "Auto Parry has been started!",
                Duration = 3,
                Icon = "rbxassetid://14467433545"
            })
        else
            stopAutoParry()
            local StarterGui = game:GetService("StarterGui")
            StarterGui:SetCore("SendNotification", {
                Title = "Hung Hub",
                Text = "Auto Parry has been disabled!",
                Duration = 3,
                Icon = "rbxassetid://14467433545"
            })
        end
    end,
})

local AutoTpParryToggle = AutoParry:CreateToggle({
    Name = "Auto TP Parry",
    CurrentValue = false,
    Flag = "AutoParryV2Flag",
    SectionParent = AutoParrySection,
    Callback = function(Value)
        if Value then
            local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "Auto TP Parry has been started!",
                        Duration = 3,
                        Icon = "rbxassetid://14467433545"
                    })
            getgenv().god = true
while getgenv().god and task.wait() do
    for _,ball in next, workspace.Balls:GetChildren() do
        if ball then
            if game:GetService("Players").LocalPlayer.Character and game:GetService("Players").LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.Position, ball.Position)
                if game:GetService("Players").LocalPlayer.Character:FindFirstChild("Highlight") then
                    game:GetService("Players").LocalPlayer.Character.HumanoidRootPart.CFrame = ball.CFrame * CFrame.new(0, 0, (ball.Velocity).Magnitude * -0.5)
                    game:GetService("ReplicatedStorage").Remotes.ParryButtonPress:Fire()
                end
            end
        end
    end
end
        end
        if not Value then
            getgenv().god = false
            local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "Auto TP Parry has been disabled!",
                        Duration = 3,
                        Icon = "rbxassetid://14467433545"
                    })
        end
    end,
})

local AutoRagingDeflect = AutoParry:CreateToggle({
    Name = "Auto Rage Parry/Rapture Parry",
    CurrentValue = false,
    Flag = "AutoRagingDeflectFlag",
    SectionParent = AutoParrySection,
    Callback = function(Value)
        if Value then
            startAutoParry()
            UseRage = Value
            local StarterGui = game:GetService("StarterGui")
            StarterGui:SetCore("SendNotification", {
                Title = "Hung Hub",
                Text = "Auto Parry with Ability has been started!",
                Duration = 3,
                Icon = "rbxassetid://14467433545"
            })
        else
            stopAutoParry()
            UseRage = Value
            local StarterGui = game:GetService("StarterGui")
            StarterGui:SetCore("SendNotification", {
                Title = "Hung Hub",
                Text = "Auto Parry with Ability has been disabled!",
                Duration = 3,
                Icon = "rbxassetid://14467433545"
            })
        end
    end,
})

 local SpamParry = AutoParry:CreateKeybind({
    Name = "Spam Parry (Hold)",
    CurrentKeybind = "X",
    HoldToInteract = true,
    Flag = "ToggleParrySpam", 
    SectionParent = AutoParrySection,
    Callback = function(Keybind)
        parryButtonPress:Fire()
    end,
 })
 
 local FakePlatform = AutoParry:CreateSlider({
    Name = "Fake Platform",
    Range = {0, 200},
    Increment = 1,
    Suffix = "Height",
    CurrentValue = 0,
    Flag = "Lmao",
    SectionParent = AutoParrySection,
    Callback = function(Value)
        character.Humanoid.HipHeight = Value ---Fakeplatform
    end,
 })

local Configuration = AutoParry:CreateSection("Parry Setting")

local DistanceSlider = AutoParry:CreateSlider({
    Name = "Distance Configuration",
    Range = {5, 50},
    Increment = 0.5,
    Suffix = "Distance",
    CurrentValue = 11,
    Flag = "DistanceSlider",
    SectionParent = Configuration,
    Callback = function(Value)
        autoparrydistance = Value
    end,
 })

local ToggleParryOffPlus = AutoParry:CreateKeybind({
    Name = "+1 Range",
    CurrentKeybind = "V",
    HoldToInteract = false,
    Flag = "ToggleParryOffPlus",
    SectionParent = Configuration,
    Callback = function()
         if autoparrydistance < 200 then
            autoparrydistance = autoparrydistance + 1
             DistanceSlider:Set(autoparrydistance)
             notify("Range Increased", "New Range: " .. autoparrydistance)
         end
    end,
 })
 
 local ToggleParryOffMinus = AutoParry:CreateKeybind({
    Name = "-1 Range",
    CurrentKeybind = "B",
    HoldToInteract = false,
    Flag = "ToggleParryOffMinus",
    SectionParent = Configuration,
    Callback = function()
         if autoparrydistance > 0 then
            autoparrydistance = autoparrydistance - 1
             DistanceSlider:Set(autoparrydistance)
             notify("Range Decreased", "New Range: " .. autoparrydistance)
         end
    end,
 })

local AbilitySec = Ability:CreateParagraph({Title = "Note", Content = "Will update more abilites sonn cuz I'm very lazy rn :("})

local AbilitySection = Ability:CreateSection("Abilites")

local Dash = Ability:CreateButton({
    Name = "Dash",
    SectionParent = AbilitySection,
    Callback = function()
        local args = {
            [1] = "Dash"
        }
        
        game:GetService("ReplicatedStorage").Remotes.Store.RequestEquipAbility:InvokeServer(unpack(args))
        
        game:GetService("ReplicatedStorage").Remotes.Store.GetOwnedAbilities:InvokeServer()
        
        game:GetService("ReplicatedStorage").Remotes.kebaind:FireServer()
        
            local function AbilityValue2()
        local TruValue = Instance.new("StringValue")
        workspace:FindFirstChild("AbilityThingyk1212"):Remove()
                TruValue.Parent = game:GetService("Workspace")
                TruValue.Name = "AbilityThingyk1212"
                TruValue.Value = "Dash"
        
        end
        
        for i,v in pairs(abilitiesFolder:GetChildren()) do
        
        
        for i,b in pairs(abilitiesFolder:GetChildren()) do
            local Ability = b
            
            if v.Enabled == true then
                local EquippedAbility = v
                local ChosenAbility = {}
                spawn(function()
                ChosenAbility = AbilityValue2()
            end)
        
            task.wait(0.05)
                local AbilityValue = workspace.AbilityThingyk1212
                if b.Name == AbilityValue.Value then
        
                    v.Enabled = false
                    b.Enabled = true
            end
        end
        end
        end
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = "Hung Hub",
            Text = "You got dash ability!",
            Duration = 3,
            Icon = "rbxassetid://14467433545"
        })
    end,
})

local PhaseBypass = Ability:CreateButton({
    Name = "Phase Bypass",
    SectionParent = AbilitySection,
    Callback = function()
        local args = {
            [1] = "Phase Bypass"
        }
        
        game:GetService("ReplicatedStorage").Remotes.Store.RequestEquipAbility:InvokeServer(unpack(args))
        
        game:GetService("ReplicatedStorage").Remotes.Store.GetOwnedAbilities:InvokeServer()
        
        game:GetService("ReplicatedStorage").Remotes.kebaind:FireServer()
                    
        local function AbilityValue2()
        local TruValue = Instance.new("StringValue")
        workspace:FindFirstChild("AbilityThingyk1212"):Remove()
                TruValue.Parent = game:GetService("Workspace")
                TruValue.Name = "AbilityThingyk1212"
                TruValue.Value = "Phase Bypass"
        end
        
        for i,v in pairs(abilitiesFolder:GetChildren()) do
        
        
        for i,b in pairs(abilitiesFolder:GetChildren()) do
            local Ability = b
            
            if v.Enabled == true then
                local EquippedAbility = v
                local ChosenAbility = {}
                spawn(function()
                ChosenAbility = AbilityValue2()
            end)
        
            task.wait(0.05)
                local AbilityValue = workspace.AbilityThingyk1212
                if b.Name == AbilityValue.Value then
        
                    v.Enabled = false
                    b.Enabled = true
            end
        end
        end
        end
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = "Hung Hub",
            Text = "You got phase bypass ability!",
            Duration = 3,
            Icon = "rbxassetid://14467433545"
        })
    end,
})

local ShadowStep = Ability:CreateButton({
    Name = "Shadow Step",
    SectionParent = AbilitySection,
    Callback = function()
        local args = {
            [1] = "Shadow Step"
        }
        
        game:GetService("ReplicatedStorage").Remotes.Store.RequestEquipAbility:InvokeServer(unpack(args))
        
        game:GetService("ReplicatedStorage").Remotes.Store.GetOwnedAbilities:InvokeServer()
        
        game:GetService("ReplicatedStorage").Remotes.kebaind:FireServer()
                    
        local function AbilityValue2()
        local TruValue = Instance.new("StringValue")
        workspace:FindFirstChild("AbilityThingyk1212"):Remove()
                TruValue.Parent = game:GetService("Workspace")
                TruValue.Name = "AbilityThingyk1212"
                TruValue.Value = "Shadow Step"
        end
        
        for i,v in pairs(abilitiesFolder:GetChildren()) do
        
        
        for i,b in pairs(abilitiesFolder:GetChildren()) do
            local Ability = b
            
            if v.Enabled == true then
                local EquippedAbility = v
                local ChosenAbility = {}
                spawn(function()
                ChosenAbility = AbilityValue2()
            end)
        
            task.wait(0.05)
                local AbilityValue = workspace.AbilityThingyk1212
                if b.Name == AbilityValue.Value then
        
                    v.Enabled = false
                    b.Enabled = true
            end
        end
        end
        end
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = "Hung Hub",
            Text = "You got shadow step ability!",
            Duration = 3,
            Icon = "rbxassetid://14467433545"
        })
    end,
})

local SuperJump = Ability:CreateButton({
    Name = "Super Jump",
    SectionParent = AbilitySection,
    Callback = function()
        local args = {
            [1] = "Super Jump"
        }
        
        game:GetService("ReplicatedStorage").Remotes.Store.RequestEquipAbility:InvokeServer(unpack(args))
        
        game:GetService("ReplicatedStorage").Remotes.Store.GetOwnedAbilities:InvokeServer()
        
        game:GetService("ReplicatedStorage").Remotes.kebaind:FireServer()
                    
        local function AbilityValue2()
        local TruValue = Instance.new("StringValue")
        workspace:FindFirstChild("AbilityThingyk1212"):Remove()
                TruValue.Parent = game:GetService("Workspace")
                TruValue.Name = "AbilityThingyk1212"
                TruValue.Value = "Super Jump"
        end
        
        for i,v in pairs(abilitiesFolder:GetChildren()) do
        
        
        for i,b in pairs(abilitiesFolder:GetChildren()) do
            local Ability = b
            
            if v.Enabled == true then
                local EquippedAbility = v
                local ChosenAbility = {}
                spawn(function()
                ChosenAbility = AbilityValue2()
            end)
        
            task.wait(0.05)
                local AbilityValue = workspace.AbilityThingyk1212
                if b.Name == AbilityValue.Value then
        
                    v.Enabled = false
                    b.Enabled = true
            end
        end
        end
        end
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = "Hung Hub",
            Text = "You got super jump ability!",
            Duration = 3,
            Icon = "rbxassetid://14467433545"
        })
    end,
})

local ThunderDash = Ability:CreateButton({
    Name = "Thunder Dash",
    SectionParent = AbilitySection,
    Callback = function()
        local args = {
            [1] = "Thunder Dash"
        }
        
        game:GetService("ReplicatedStorage").Remotes.Store.RequestEquipAbility:InvokeServer(unpack(args))
        
        game:GetService("ReplicatedStorage").Remotes.Store.GetOwnedAbilities:InvokeServer()
        
        game:GetService("ReplicatedStorage").Remotes.kebaind:FireServer()
                    
        local function AbilityValue2()
        local TruValue = Instance.new("StringValue")
        workspace:FindFirstChild("AbilityThingyk1212"):Remove()
                TruValue.Parent = game:GetService("Workspace")
                TruValue.Name = "AbilityThingyk1212"
                TruValue.Value = "Thunder Dash"
        end
        
        for i,v in pairs(abilitiesFolder:GetChildren()) do
        
        
        for i,b in pairs(abilitiesFolder:GetChildren()) do
            local Ability = b
            
            if v.Enabled == true then
                local EquippedAbility = v
                local ChosenAbility = {}
                spawn(function()
                ChosenAbility = AbilityValue2()
            end)
        
            task.wait(0.05)
                local AbilityValue = workspace.AbilityThingyk1212
                if b.Name == AbilityValue.Value then
        
                    v.Enabled = false
                    b.Enabled = true
            end
        end
        end
        end
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = "Hung Hub",
            Text = "You got thunder dash ability!",
            Duration = 3,
            Icon = "rbxassetid://14467433545"
        })
    end,
})

local WindCloak = Ability:CreateButton({
    Name = "Wind Cloak",
    SectionParent = AbilitySection,
    Callback = function()
        local args = {
            [1] = "Wind Cloak"
        }
        
        game:GetService("ReplicatedStorage").Remotes.Store.RequestEquipAbility:InvokeServer(unpack(args))
        
        game:GetService("ReplicatedStorage").Remotes.Store.GetOwnedAbilities:InvokeServer()
        
        game:GetService("ReplicatedStorage").Remotes.kebaind:FireServer()
                    
        local function AbilityValue2()
        local TruValue = Instance.new("StringValue")
        workspace:FindFirstChild("AbilityThingyk1212"):Remove()
                TruValue.Parent = game:GetService("Workspace")
                TruValue.Name = "AbilityThingyk1212"
                TruValue.Value = "Wind Cloak"
        end
        
        for i,v in pairs(abilitiesFolder:GetChildren()) do
        
        
        for i,b in pairs(abilitiesFolder:GetChildren()) do
            local Ability = b
            
            if v.Enabled == true then
                local EquippedAbility = v
                local ChosenAbility = {}
                spawn(function()
                ChosenAbility = AbilityValue2()
            end)
        
            task.wait(0.05)
                local AbilityValue = workspace.AbilityThingyk1212
                if b.Name == AbilityValue.Value then
        
                    v.Enabled = false
                    b.Enabled = true
            end
        end
        end
        end
        local StarterGui = game:GetService("StarterGui")
        StarterGui:SetCore("SendNotification", {
            Title = "Hung Hub",
            Text = "You got wind cloak ability!",
            Duration = 3,
            Icon = "rbxassetid://14467433545"
        })
    end,
})

local InfSkill = Misc:CreateSection("Inf Skill (No CD)")

local upgrades = localPlayer.Upgrades

local InfDash = Misc:CreateButton({
    Name = "Inf Dash",
    SectionParent = InfSkill,
    Callback = function()
        upgrades:WaitForChild("Dash").Value = 999999999999999999
            local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "You got inf dash!",
                        Duration = 3,
                        Icon = "rbxassetid://14467433545"
                    })
    end,
})

local InfST = Misc:CreateButton({
    Name = "Inf Shadow Step",
    SectionParent = InfSkill,
    Callback = function()
        upgrades:WaitForChild("Shadow Step").Value = 999999999999999999
            local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "You got inf shadow step!",
                        Duration = 3,
                        Icon = "rbxassetid://14467433545"
                    })
    end,
})

local InfSuperJump = Misc:CreateButton({
    Name = "Inf Super Jump",
    SectionParent = InfSkill,
    Callback = function()
        upgrades:WaitForChild("Super Jump").Value = 999999999999999999
            local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "You got inf super jump!",
                        Duration = 3,
                        Icon = "rbxassetid://14467433545"
                    })
    end,
})

local InfThunderDash = Misc:CreateButton({
    Name = "Inf Thunder Dash",
    SectionParent = InfSkill,
    Callback = function()
        upgrades:WaitForChild("Thunder Dash").Value = 999999999999999999
            local StarterGui = game:GetService("StarterGui")
                    StarterGui:SetCore("SendNotification", {
                        Title = "Hung Hub",
                        Text = "You got inf thunder dash!",
                        Duration = 3,
                        Icon = "rbxassetid://14467433545"
                    })
    end,
})

local Others = Misc:CreateSection("Other")

local x2Code = {
    "WEEK4",
    "SORRY4DELAY"
}

local RedeemButton = Misc:CreateButton({
    Name = "Redeem All Codes",
    SectionParent = Others,
    Callback = function()
        function RedeemCode(value)
            game:GetService("ReplicatedStorage").Remotes.SubmitCodeRequest:InvokeServer(value)
        end
        for i,v in pairs(x2Code) do
            RedeemCode(v)
        end
        local StarterGui = game:GetService("StarterGui")
                StarterGui:SetCore("SendNotification", {
                    Title = "Hung Hub",
                    Text = "Redeemed all codes!",
                    Duration = 3,
                    Icon = "rbxassetid://14467433545"
                })
    end,
})

local SvHop = Misc:CreateButton({
    Name = "Server Hop",
    SectionParent = Others,
    Callback = function()
        local StarterGui = game:GetService("StarterGui")
                StarterGui:SetCore("SendNotification", {
                    Title = "Hung Hub",
                    Text = "Hopping to another server!",
                    Duration = 3,
                    Icon = "rbxassetid://14467433545"
                })
            local PlaceID = game.PlaceId
                                local AllIDs = {}
                                local foundAnything = ""
                                local actualHour = os.date("!*t").hour
                                local Deleted = false
                                function TPReturner()
                                local Site;
                                if foundAnything == "" then
                                Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
                                else
                                Site = game.HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
                                end
                                local ID = ""
                                if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
                                foundAnything = Site.nextPageCursor
                                end
                                local num = 0;
                                for i,v in pairs(Site.data) do
                                local Possible = true
                                ID = tostring(v.id)
                                if tonumber(v.maxPlayers) > tonumber(v.playing) then
                                for _,Existing in pairs(AllIDs) do
                                if num ~= 0 then
                                if ID == tostring(Existing) then
                                Possible = false
                                end
                                else
                                if tonumber(actualHour) ~= tonumber(Existing) then
                                local delFile = pcall(function()
                                -- delfile("NotSameServers.json")
                                AllIDs = {}
                                table.insert(AllIDs, actualHour)
                                end)
                                end
                                end
                                num = num + 1
                                end
                                if Possible == true then
                                table.insert(AllIDs, ID)
                                wait()
                                pcall(function()
                                -- writefile("NotSameServers.json", game:GetService('HttpService'):JSONEncode(AllIDs))
                                wait()
                                game:GetService("TeleportService"):TeleportToPlaceInstance(PlaceID, ID, game.Players.LocalPlayer)
                                end)
                                wait(4)
                                end
                                end
                                end
                                end
                                function Teleport()
                                while wait() do
                                pcall(function()
                                TPReturner()
                                if foundAnything ~= "" then
                                TPReturner()
                                end
                                end)
                                end
                                end
                                Teleport()
    end,
})
