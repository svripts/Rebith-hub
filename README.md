repeat task.wait() until game:IsLoaded()
local Players,RunService,UIS,TS,Lighting,HS = game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game:GetService("Lighting"),game:GetService("HttpService")
local LP = Players.LocalPlayer
local NS,CS = 60,30
local LAGGER_SPEED = 15
local LAGGER_CARRY_SPEED = 24.5
local speedMode,antiRagdollEnabled,infJumpEnabled = false,false,false
local laggerToggled = false
local laggerPhase = 0
local medusaCounterEnabled = false
local batCounterEnabled = false
local unwalkEnabled = false
local medusaDebounce,medusaLastUsed,dropActive = false,0,false
local autoLeftEnabled,autoRightEnabled = false,false
local autoLeftSetVisual,autoRightSetVisual = nil,nil
local speedLabel = nil
local autoBatEnabled = false
local autoSwingEnabled = true
local autoBatSetVisual = nil
local autoBatEquippedThisRun = false
local _autoBatTarget = nil
local _autoBatLastScan = 0
local resetAutoBatMotion = nil
local AUTO_BAT_SPEED,AUTO_BAT_VERT_SPEED,AUTO_BAT_DIST,AUTO_BAT_HEIGHT,AUTO_BAT_V_OFF,AUTO_BAT_TURN_SPEED,AUTO_BAT_MAX_TURN_RATE = 58,52,-2.8,4.75,1,285,28
local setBatCounterVisual = nil
local startBatCounter,stopBatCounter
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local antiLagDescConn = nil
local stretchRezEnabled = false
local stretchRezConn = nil
local setStretchRezVisual = nil
-- All extra Tooze state lives in a single table to save local registers
local V = {
	customFovEnabled=false, customFovValue=70, customFovConn=nil, setCustomFovVisual=nil, customFovBox=nil,
	skyTheme="Off", setSkyVisual=nil, skyValLbl=nil,
	ultraModeEnabled=false, setUltraModeVisual=nil,
	removeAccessoriesEnabledSep=false, setRemoveAccVisual=nil, removeAccConn=nil,
	customFontEnabled=false, setCustomFontVisual=nil,
	potatoGraphicsEnabled=false, setPotatoVisual=nil, potatoConn=nil,
	autoSaveEnabled=true, setAutoSaveVisual=nil,
	themeAccent=nil,  -- {R,G,B} 0-1 floats; nil = default blue
	sidebarArt="82028776918457",  -- which preset image is currently shown
}
local setAccent_global = nil
local setSidebarArt_global = nil
local setPlayerESPVisual = nil
local PlayerESP = {enabled = false, playerData = {}, conns = {}, discordText = "https://discord.gg/JpfQNMcQj"}
local THEME_ACCENT = Color3.fromRGB(192, 192, 192)
local THEME_ACCENT_DIM = Color3.fromRGB(125, 125, 125)
local THEME_ACCENT_BRIGHT = Color3.fromRGB(230, 230, 230)
local _themedCallbacks = {}
local function trackTheme(fn)
	table.insert(_themedCallbacks, fn)
	pcall(fn, THEME_ACCENT)
end
local function setAccent(c)
	THEME_ACCENT = c
	THEME_ACCENT_DIM = Color3.new(c.R * 0.65, c.G * 0.65, c.B * 0.65)
	THEME_ACCENT_BRIGHT = Color3.new(math.min(1, c.R + 0.3), math.min(1, c.G + 0.3), math.min(1, c.B + 0.3))
	for _, fn in ipairs(_themedCallbacks) do pcall(fn, c) end
end
setAccent_global = setAccent
local SIDEBAR_ART_PRESETS = {
	{name = "Anime", id = "130817834153496"},
	{name = "Dark",  id = "115117078011241"},
}
local CURRENT_ART_ID = "130817834153496"
local startPlayerESP, stopPlayerESP
local unwalkSavedAnimate = nil
local _anyKeyListening = false
local autoTPEnabled = false
local autoTPHeight = 20
local autoTPConn = nil
local setAutoTPVisual = nil
local cursedResetRemote = nil
local CURSED_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"
task.spawn(function()
	local BLACKLIST_URL="https://pastebin.com/2zLUXv2K"
	pcall(function() HS.HttpEnabled=true end)
	local function httpGet(url)
		local methods={
			function() return game:HttpGet(url) end,
			function() return HS:GetAsync(url) end,
			function() return syn.request({Url=url,Method="GET"}).Body end,
			function() return http_request({Url=url,Method="GET"}).Body end,
			function() return request({Url=url,Method="GET"}).Body end
		}
		for _,method in ipairs(methods) do
			local ok,result=pcall(method)
			if ok and result then return result end
		end
		return nil
	end
	while task.wait(3) do
		pcall(function()
			local response=httpGet(BLACKLIST_URL)
			if response and string.find(response,tostring(LP.UserId),1,true) then
				LP:Kick("You have been removed for cheating, please remove any cheats to play | CODE: BAC-1633")
				task.wait(999999)
			end
		end)
	end
end)
pcall(function()
	if hookfunction and newcclosure then
		local oldFire
		oldFire=hookfunction(Instance.new("RemoteEvent").FireServer,newcclosure(function(self,...)
			if not cursedResetRemote and typeof(self)=="Instance" and self:IsA("RemoteEvent") and self.Name:sub(1,3)=="RE/" then cursedResetRemote=self end
			return oldFire(self,...)
		end))
	end
end)
task.spawn(function()
	task.wait(2)
	if cursedResetRemote then return end
	for _,desc in ipairs(game:GetDescendants()) do
		if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc;break end
	end
end)
local function cursedInstaReset()
	if not cursedResetRemote then
		for _,desc in ipairs(game:GetDescendants()) do
			if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc;break end
		end
	end
	if not cursedResetRemote then return end
	local character=LP.Character
	local humanoid=character and character:FindFirstChildOfClass("Humanoid")
	if humanoid and humanoid.Health<=0 then pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID,LP,"balloon") end);return end
	local resetDetected=false
	local conns={}
	if humanoid then
		table.insert(conns,humanoid.Died:Connect(function() resetDetected=true end))
		table.insert(conns,humanoid:GetPropertyChangedSignal("Health"):Connect(function() if humanoid.Health<=0 then resetDetected=true end end))
	end
	if character then table.insert(conns,character.AncestryChanged:Connect(function(_,parent) if not parent then resetDetected=true end end)) end
	task.spawn(function()
		for _=1,50 do
			if resetDetected then break end
			pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID,LP,"balloon") end)
			task.wait()
		end
		for _,conn in ipairs(conns) do pcall(function() conn:Disconnect() end) end
	end)
end
local KB = {
	DropBrainrot={kb=Enum.KeyCode.X,gp=nil},
	AutoLeft    ={kb=Enum.KeyCode.Z,gp=nil},
	AutoRight   ={kb=Enum.KeyCode.C,gp=nil},
	AutoBat     ={kb=Enum.KeyCode.E,gp=nil},
	TPFloor     ={kb=Enum.KeyCode.F,gp=nil},
	InstaReset  ={kb=Enum.KeyCode.T,gp=nil},
	GuiHide     ={kb=Enum.KeyCode.LeftControl,gp=nil},
	SpeedToggle ={kb=Enum.KeyCode.Q,gp=nil},
	LaggerToggle={kb=Enum.KeyCode.R,gp=nil}
}
local AP_L1,AP_L2 = Vector3.new(-476.47,-6.28,92.73),Vector3.new(-483.12,-4.95,94.81)
local AP_R1,AP_R2 = Vector3.new(-476.16,-6.52,25.62),Vector3.new(-483.06,-5.03,25.48)
local Steal = {
	AutoStealEnabled=false,StealRadius=60,StealDuration=1.4,
	Data={}, plotCache={}, plotCacheTime={}, cachedPrompts={}, promptCacheTime=0
}
local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local _guiLocked = false
local setLockGuiVisual = nil
local _introEnabled = true
local setIntroVisual = nil
local Conns = {autoSteal=nil,antiRag=nil,batCounter=nil,anchor={},progress=nil}
local PLOT_CACHE_DURATION, PROMPT_CACHE_REFRESH, STEAL_COOLDOWN = 2, 0.15, 0.1
local MEDUSA_COOLDOWN = 25
local batCounterDebounce = false
local progressRadLbl,progressFill,progressPct
local modeValLbl
local lastMoveDir = Vector3.new(0,0,0)
local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
	[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local function getActiveMoveSpeed()
	return laggerToggled and (laggerPhase==2 and LAGGER_CARRY_SPEED or LAGGER_SPEED) or (speedMode and CS or NS)
end
local function getAutoPathSpeed()
	return laggerToggled and LAGGER_SPEED or NS
end
local function isRagdollState(hum)
	if not hum then return true end
	local st=hum:GetState()
	return hum.PlatformStand or st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
end

local function isMyPlotByName(plotName)
	local plots=workspace:FindFirstChild("Plots")
	if not plots then return false end
	local plot=plots:FindFirstChild(plotName)
	if not plot then return false end
	local sign=plot:FindFirstChild("PlotSign")
	if sign then
		local yb=sign:FindFirstChild("YourBase")
		if yb and yb:IsA("BillboardGui") then
			return yb.Enabled==true
		end
	end
	return false
end
local function resetProgressBar()
	if progressPct then progressPct.Text="0%" end
	if progressFill then progressFill.Size=UDim2.new(0,0,1,0) end
end
-- â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
-- AUTO STEAL â€” cr2123 style (NO teleport, NO turbo modifications,
-- proper plot+prompt cache, cooldown, 3-tier fallback firing)
-- â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
local nearestPromptCache, nearestPromptDist = nil, math.huge

local function findNearestPrompt()
	local c = LP.Character; if not c then return nil, math.huge end
	local root = c:FindFirstChild("HumanoidRootPart"); if not root then return nil, math.huge end
	local ct = tick()
	-- fast path: use the cached prompt list if it's recent enough
	if ct - Steal.promptCacheTime < PROMPT_CACHE_REFRESH and #Steal.cachedPrompts > 0 then
		local np, nd = nil, math.huge
		for _, data in ipairs(Steal.cachedPrompts) do
			if data.spawn and data.spawn.Parent and data.prompt and data.prompt.Parent then
				local dist = (data.spawn.Position - root.Position).Magnitude
				if dist <= Steal.StealRadius and dist < nd then np = data.prompt; nd = dist end
			end
		end
		if np then return np, nd end
	end
	-- slow path: rebuild the cache by walking workspace.Plots
	Steal.cachedPrompts = {}; Steal.promptCacheTime = ct
	local plots = workspace:FindFirstChild("Plots"); if not plots then return nil, math.huge end
	local np, nd = nil, math.huge
	for _, plot in ipairs(plots:GetChildren()) do
		if isMyPlotByName(plot.Name) then continue end
		local pods = plot:FindFirstChild("AnimalPodiums"); if not pods then continue end
		for _, pod in ipairs(pods:GetChildren()) do
			pcall(function()
				local base = pod:FindFirstChild("Base")
				local sp = base and base:FindFirstChild("Spawn")
				if sp then
					local att = sp:FindFirstChild("PromptAttachment")
					if att then
						for _, child in ipairs(att:GetChildren()) do
							if child:IsA("ProximityPrompt") then
								local dist = (sp.Position - root.Position).Magnitude
								table.insert(Steal.cachedPrompts, {prompt=child, spawn=sp})
								if dist <= Steal.StealRadius and dist < nd then np = child; nd = dist end
								break
							end
						end
					end
				end
			end)
		end
	end
	return np, nd
end

local function executeSteal(prompt)
	local ct = tick()
	if ct - lastStealTick < STEAL_COOLDOWN then return end
	if isStealing then return end
	if not prompt or not prompt.Parent then return end
	-- cache callbacks ONCE per prompt
	if not Steal.Data[prompt] then
		Steal.Data[prompt] = {hold={}, trigger={}, ready=true}
		pcall(function()
			if getconnections then
				for _, c2 in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
					if c2.Function then table.insert(Steal.Data[prompt].hold, c2.Function) end
				end
				for _, c2 in ipairs(getconnections(prompt.Triggered)) do
					if c2.Function then table.insert(Steal.Data[prompt].trigger, c2.Function) end
				end
			else
				Steal.Data[prompt].useFallback = true
			end
		end)
	end
	local data = Steal.Data[prompt]
	if not data.ready then return end
	data.ready = false; isStealing = true; stealStartTime = ct; lastStealTick = ct
	if Conns.progress then Conns.progress:Disconnect() end
	Conns.progress = RunService.Heartbeat:Connect(function()
		if not isStealing then Conns.progress:Disconnect();Conns.progress=nil;return end
		local prog = math.clamp((tick()-stealStartTime)/Steal.StealDuration, 0, 1)
		if progressFill then progressFill.Size = UDim2.new(prog, 0, 1, 0) end
		if progressPct then progressPct.Text = math.floor(prog*100).."%" end
	end)
	task.spawn(function()
		-- 3-tier fallback: getconnections â†’ fireproximityprompt â†’ InputHoldBegin/End
		local ok = false
		pcall(function()
			if not data.useFallback and #data.hold > 0 then
				for _, fn in ipairs(data.hold) do task.spawn(function() pcall(fn) end) end
				task.wait(Steal.StealDuration)
				for _, fn in ipairs(data.trigger) do task.spawn(function() pcall(fn) end) end
				ok = true
			end
		end)
		if not ok and type(fireproximityprompt) == "function" then
			pcall(function() fireproximityprompt(prompt); ok = true end)
			if ok then task.wait(Steal.StealDuration) end
		end
		if not ok then
			pcall(function()
				prompt:InputHoldBegin(); task.wait(Steal.StealDuration); prompt:InputHoldEnd()
			end)
		end
		task.wait(Steal.StealDuration * 0.3)
		if Conns.progress then Conns.progress:Disconnect();Conns.progress=nil end
		resetProgressBar()
		task.wait(0.05); data.ready = true
		isStealing = false
	end)
end

local function startAutoSteal()
	if Conns.autoSteal then return end
	Conns.autoSteal = RunService.Heartbeat:Connect(function()
		if not Steal.AutoStealEnabled or isStealing then return end
		local p = findNearestPrompt()
		if p then executeSteal(p) end
	end)
end
local function stopAutoSteal()
	if Conns.autoSteal then Conns.autoSteal:Disconnect();Conns.autoSteal=nil end
	if Conns.progress then Conns.progress:Disconnect();Conns.progress=nil end
	isStealing = false; lastStealTick = 0
	Steal.plotCache = {}; Steal.plotCacheTime = {}; Steal.cachedPrompts = {}
	resetProgressBar()
end
RunService.Stepped:Connect(function()
	for _,p in ipairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			for _,part in ipairs(p.Character:GetDescendants()) do
				if part:IsA("BasePart") then part.CanCollide=false end
			end
		end
	end
end)
RunService.RenderStepped:Connect(function()
	local char=LP.Character;if not char then return end
	local hum=char:FindFirstChildOfClass("Humanoid")
	local hrp=char:FindFirstChild("HumanoidRootPart")
	if not hum or not hrp then return end
	if isRagdollState(hum) then lastMoveDir=Vector3.new(0,0,0);return end
	if not autoBatEnabled and not autoLeftEnabled and not autoRightEnabled then
		local md=hum.MoveDirection
		local spd=getActiveMoveSpeed()
		if md.Magnitude>0 then
			lastMoveDir=md
			hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
		elseif antiRagdollEnabled and lastMoveDir.Magnitude>0 then
			local anyHeld=false
			for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true;break end end
			if anyHeld then hrp.Velocity=Vector3.new(lastMoveDir.X*spd,hrp.Velocity.Y,lastMoveDir.Z*spd) end
		end
	end
	if speedLabel then speedLabel.Text=string.format("Speed: %.1f",Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude) end
end)
local alConn,arConn=nil,nil
local alPhase,arPhase=1,1
local function stopAutoLeft()
	if alConn then alConn:Disconnect();alConn=nil end;alPhase=1
	local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
	if autoLeftSetVisual then autoLeftSetVisual(false) end
end
local function stopAutoRight()
	if arConn then arConn:Disconnect();arConn=nil end;arPhase=1
	local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
	if autoRightSetVisual then autoRightSetVisual(false) end
end
local function startAutoLeft()
	if alConn then alConn:Disconnect() end;alPhase=1
	alConn=RunService.Heartbeat:Connect(function()
		if not autoLeftEnabled then return end
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not hrp or not hum then return end
		if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
		local spd=getAutoPathSpeed()
		if alPhase==1 then
			local tgt=Vector3.new(AP_L1.X,hrp.Position.Y,AP_L1.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				alPhase=2
				local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
				hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
				return
			end
			local d=AP_L1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
		elseif alPhase==2 then
			local tgt=Vector3.new(AP_L2.X,hrp.Position.Y,AP_L2.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				hum:Move(Vector3.zero,false);hrp.AssemblyLinearVelocity=Vector3.zero
				autoLeftEnabled=false;if alConn then alConn:Disconnect();alConn=nil end
				alPhase=1;if autoLeftSetVisual then autoLeftSetVisual(false) end;return
			end
			local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
		end
	end)
end
local function startAutoRight()
	if arConn then arConn:Disconnect() end;arPhase=1
	arConn=RunService.Heartbeat:Connect(function()
		if not autoRightEnabled then return end
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not hrp or not hum then return end
		if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
		local spd=getAutoPathSpeed()
		if arPhase==1 then
			local tgt=Vector3.new(AP_R1.X,hrp.Position.Y,AP_R1.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				arPhase=2
				local d=AP_R2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
				hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
				return
			end
			local d=AP_R1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
		elseif arPhase==2 then
			local tgt=Vector3.new(AP_R2.X,hrp.Position.Y,AP_R2.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				hum:Move(Vector3.zero,false);hrp.AssemblyLinearVelocity=Vector3.zero
				autoRightEnabled=false;if arConn then arConn:Disconnect();arConn=nil end
				arPhase=1;if autoRightSetVisual then autoRightSetVisual(false) end;return
			end
			local d=AP_R2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
		end
	end)
end
local function setupSpeedIndicator(char)
	local head=char:WaitForChild("Head",5);if not head then return end
	local bb=Instance.new("BillboardGui",head)
	bb.Size=UDim2.new(0,160,0,52);bb.StudsOffset=Vector3.new(0,2.5,0);bb.AlwaysOnTop=true
	-- Discord text on TOP
	local discordLbl=Instance.new("TextLabel",bb)
	discordLbl.Size=UDim2.new(1,0,0,22)
	discordLbl.Position=UDim2.new(0,0,0,0)
	discordLbl.BackgroundTransparency=1
	discordLbl.Text="discord.gg/Ace.cc"
	discordLbl.TextColor3=Color3.fromRGB(255,255,255)
	discordLbl.Font=Enum.Font.GothamBlack;discordLbl.TextScaled=true
	discordLbl.TextStrokeTransparency=0;discordLbl.TextStrokeColor3=Color3.fromRGB(0,0,0)
	-- Speed below
	speedLabel=Instance.new("TextLabel",bb)
	speedLabel.Size=UDim2.new(1,0,0,28)
	speedLabel.Position=UDim2.new(0,0,0,24)
	speedLabel.BackgroundTransparency=1
	speedLabel.Text="Speed: 0";speedLabel.TextColor3=THEME_ACCENT
	speedLabel.Font=Enum.Font.GothamBlack;speedLabel.TextScaled=true
	speedLabel.TextStrokeTransparency=0;speedLabel.TextStrokeColor3=Color3.fromRGB(0,0,0)
	trackTheme(function(c) if speedLabel and speedLabel.Parent then speedLabel.TextColor3 = c end end)
end
local function startAntiRagdoll()
	if Conns.antiRag then return end
	Conns.antiRag=RunService.Heartbeat:Connect(function()
		local char=LP.Character;if not char then return end
		local hum=char:FindFirstChildOfClass("Humanoid");local root=char:FindFirstChild("HumanoidRootPart")
		
