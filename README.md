-- ============ GUI ============
VisualSetters={} local SliderSetters={} local KeyButtons={} local waitingForKeybind=nil

local G = {
    BG    = Color3.fromRGB(10,10,12),
    CARD  = Color3.fromRGB(16,16,20),
    CARD2 = Color3.fromRGB(22,22,28),
    BORD  = Color3.fromRGB(45,45,55),
    TXT   = Color3.fromRGB(220,220,225),
    DIM   = Color3.fromRGB(100,100,115),
    ACC   = Color3.fromRGB(220,220,220),
    ON    = Color3.fromRGB(60,200,100),
    OFF   = Color3.fromRGB(40,40,50),
    RED   = Color3.fromRGB(210,60,60),
}

local guiVisible=true
local sg=Instance.new("ScreenGui")
sg.Name="ZyphrotHub_V3" sg.ResetOnSpawn=false sg.IgnoreGuiInset=true
pcall(function() sg.Parent=CoreGui end)
if not sg.Parent then sg.Parent=Player:WaitForChild("PlayerGui") end

local function playSound(id,vol)
    pcall(function()
        local s=Instance.new("Sound",SoundService) s.SoundId=id s.Volume=vol or 0.3 s:Play()
        game:GetService("Debris"):AddItem(s,1)
    end)
end

-- TextChat send helper
local function sendChat(msg)
    task.spawn(function()
        pcall(function()
            local tcs=game:GetService("TextChatService")
            local ch=tcs:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
            ch:SendAsync(msg)
        end)
    end)
end

-- TAUNT state
local tauntEnabled=false local tauntConn=nil
local function startTaunt()
    if tauntConn then return end
    tauntConn=task.spawn(function()
        while tauntEnabled do
            sendChat("Phantom better")
            task.wait(30)
        end
    end)
end
local function stopTaunt()
    tauntEnabled=false tauntConn=nil
end

-- ============ TOP BAR BUTTONS (top left, small pill row) ============
local topBar=Instance.new("Frame",sg)
topBar.Size=UDim2.fromOffset(130,32) topBar.Position=UDim2.new(0,8,0,8)
topBar.BackgroundColor3=G.CARD topBar.BackgroundTransparency=0.05 topBar.BorderSizePixel=0
Instance.new("UICorner",topBar).CornerRadius=UDim.new(0,8)
Instance.new("UIStroke",topBar).Color=G.BORD
local topLayout=Instance.new("UIListLayout",topBar)
topLayout.FillDirection=Enum.FillDirection.Horizontal topLayout.Padding=UDim.new(0,2) topLayout.SortOrder=Enum.SortOrder.LayoutOrder
local topPad=Instance.new("UIPadding",topBar) topPad.PaddingLeft=UDim.new(0,4) topPad.PaddingRight=UDim.new(0,4) topPad.PaddingTop=UDim.new(0,4) topPad.PaddingBottom=UDim.new(0,4)

-- ZH toggle button
local zhBtn=Instance.new("TextButton",topBar)
zhBtn.Size=UDim2.fromOffset(56,24) zhBtn.BackgroundColor3=G.CARD2 zhBtn.BackgroundTransparency=0.1
zhBtn.Text="ZH" zhBtn.Font=Enum.Font.GothamBlack zhBtn.TextSize=11
zhBtn.TextColor3=G.TXT zhBtn.BorderSizePixel=0 zhBtn.LayoutOrder=1
Instance.new("UICorner",zhBtn).CornerRadius=UDim.new(0,6)
Instance.new("UIStroke",zhBtn).Color=G.BORD

-- TAUNT button
local tauntBtn=Instance.new("TextButton",topBar)
tauntBtn.Size=UDim2.fromOffset(60,24) tauntBtn.BackgroundColor3=G.CARD2 tauntBtn.BackgroundTransparency=0.1
tauntBtn.Text="TAUNT: OFF" tauntBtn.Font=Enum.Font.GothamBold tauntBtn.TextSize=9
tauntBtn.TextColor3=G.DIM tauntBtn.BorderSizePixel=0 tauntBtn.LayoutOrder=2
Instance.new("UICorner",tauntBtn).CornerRadius=UDim.new(0,6)
local tauntStroke=Instance.new("UIStroke",tauntBtn) tauntStroke.Color=G.BORD

zhBtn.MouseButton1Click:Connect(function()
    guiVisible=not guiVisible main.Visible=guiVisible
    playSound("rbxassetid://6895079813",0.3)
end)

tauntBtn.MouseButton1Click:Connect(function()
    tauntEnabled=not tauntEnabled
    if tauntEnabled then
        tauntBtn.Text="TAUNT: ON" tauntBtn.TextColor3=G.ON tauntStroke.Color=G.ON
        task.spawn(function()
            tauntConn=true
            while tauntEnabled do sendChat("Phantom better") task.wait(30) end
        end)
    else
        tauntEnabled=false tauntBtn.Text="TAUNT: OFF" tauntBtn.TextColor3=G.DIM tauntStroke.Color=G.BORD
    end
    playSound("rbxassetid://6895079813",0.3)
end)

-- ============ MAIN WINDOW ============
local main=Instance.new("Frame",sg)
main.Size=UDim2.fromOffset(540*s,380*s) main.Position=UDim2.new(0.5,-270*s,0.5,-190*s)
main.BackgroundColor3=G.BG main.BackgroundTransparency=0.05 main.BorderSizePixel=0 main.Active=true
Instance.new("UICorner",main).CornerRadius=UDim.new(0,12)
Instance.new("UIStroke",main).Color=G.BORD
MakeDraggable(main)

-- SIDEBAR
local sidebar=Instance.new("Frame",main)
sidebar.Size=UDim2.fromOffset(130*s,380*s) sidebar.BackgroundColor3=G.CARD
sidebar.BackgroundTransparency=0.06 sidebar.BorderSizePixel=0
Instance.new("UICorner",sidebar).CornerRadius=UDim.new(0,12)
local sfFix=Instance.new("Frame",sidebar) sfFix.Size=UDim2.fromOffset(12*s,380*s) sfFix.Position=UDim2.new(1,-12*s,0,0)
sfFix.BackgroundColor3=G.CARD sfFix.BackgroundTransparency=0.06 sfFix.BorderSizePixel=0

-- Logo
local logoLbl=Instance.new("TextLabel",sidebar) logoLbl.Size=UDim2.fromOffset(130*s,44*s)
logoLbl.BackgroundTransparency=1 logoLbl.Text="ZYPHROT\nHUB" logoLbl.Font=Enum.Font.GothamBlack
logoLbl.TextSize=14*s logoLbl.TextColor3=G.TXT logoLbl.ZIndex=2

local logoDiv=Instance.new("Frame",sidebar) logoDiv.Size=UDim2.fromOffset(110*s,1) logoDiv.Position=UDim2.fromOffset(10*s,46*s)
logoDiv.BackgroundColor3=G.BORD logoDiv.BorderSizePixel=0

-- Tab container
local tabContainer=Instance.new("Frame",sidebar)
tabContainer.Size=UDim2.new(1,0,1,-(130*s)) tabContainer.Position=UDim2.fromOffset(0,50*s)
tabContainer.BackgroundTransparency=1
local tabLayout=Instance.new("UIListLayout",tabContainer) tabLayout.Padding=UDim.new(0,3*s)
local tabPad=Instance.new("UIPadding",tabContainer)
tabPad.PaddingLeft=UDim.new(0,7*s) tabPad.PaddingRight=UDim.new(0,7*s) tabPad.PaddingTop=UDim.new(0,4*s)

-- User info bottom
local uFrame=Instance.new("Frame",sidebar) uFrame.Size=UDim2.fromOffset(130*s,50*s)
uFrame.Position=UDim2.new(0,0,1,-50*s) uFrame.BackgroundTransparency=1
local uDiv2=Instance.new("Frame",uFrame) uDiv2.Size=UDim2.fromOffset(110*s,1) uDiv2.Position=UDim2.fromOffset(10*s,0)
uDiv2.BackgroundColor3=G.BORD uDiv2.BorderSizePixel=0
local uAv=Instance.new("ImageLabel",uFrame) uAv.Size=UDim2.fromOffset(24*s,24*s) uAv.Position=UDim2.fromOffset(8*s,12*s)
uAv.BackgroundColor3=G.CARD2 uAv.BorderSizePixel=0
uAv.Image="rbxthumb://type=AvatarHeadShot&id="..Player.UserId.."&w=150&h=150"
Instance.new("UICorner",uAv).CornerRadius=UDim.new(1,0)
local uNm=Instance.new("TextLabel",uFrame) uNm.Size=UDim2.fromOffset(90*s,16*s) uNm.Position=UDim2.fromOffset(36*s,14*s)
uNm.BackgroundTransparency=1 uNm.Text=Player.Name uNm.Font=Enum.Font.GothamBold uNm.TextSize=9*s
uNm.TextColor3=G.TXT uNm.TextXAlignment=Enum.TextXAlignment.Left uNm.TextTruncate=Enum.TextTruncate.AtEnd
local uSub=Instance.new("TextLabel",uFrame) uSub.Size=UDim2.fromOffset(90*s,12*s) uSub.Position=UDim2.fromOffset(36*s,28*s)
uSub.BackgroundTransparency=1 uSub.Text="Premium" uSub.Font=Enum.Font.GothamMedium uSub.TextSize=8*s
uSub.TextColor3=G.DIM uSub.TextXAlignment=Enum.TextXAlignment.Left

-- Content area
local contentArea=Instance.new("Frame",main)
contentArea.Size=UDim2.new(1,-(130*s),1,0) contentArea.Position=UDim2.fromOffset(130*s,0)
contentArea.BackgroundTransparency=1 contentArea.ClipsDescendants=true

-- Stats
local StatsFrame=Instance.new("Frame",sg) StatsFrame.Size=UDim2.fromOffset(105,38)
StatsFrame.Position=UDim2.new(0,8,1,-50) StatsFrame.BackgroundColor3=G.CARD StatsFrame.BackgroundTransparency=0.1
StatsFrame.BorderSizePixel=0 StatsFrame.Visible=Enabled.Stats
Instance.new("UICorner",StatsFrame).CornerRadius=UDim.new(0,7)
Instance.new("UIStroke",StatsFrame).Color=G.BORD
local fpsLbl=Instance.new("TextLabel",StatsFrame) fpsLbl.Size=UDim2.new(1,0,0.5,0) fpsLbl.BackgroundTransparency=1
fpsLbl.Text="FPS: --" fpsLbl.Font=Enum.Font.GothamBold fpsLbl.TextSize=10 fpsLbl.TextColor3=G.ON
local pingLbl=Instance.new("TextLabel",StatsFrame) pingLbl.Size=UDim2.new(1,0,0.5,0) pingLbl.Position=UDim2.new(0,0,0.5,0)
pingLbl.BackgroundTransparency=1 pingLbl.Text="PING: --" pingLbl.Font=Enum.Font.GothamBold pingLbl.TextSize=10 pingLbl.TextColor3=G.TXT
task.spawn(function()
    local fc,ft=0,tick()
    while sg.Parent do
        RunService.RenderStepped:Wait() fc=fc+1
        local now=tick() if now-ft>=0.5 then
            local fps=math.round(fc/(now-ft)) fc=0 ft=now fpsLbl.Text="FPS: "..fps
            fpsLbl.TextColor3=fps>=55 and G.ON or fps>=30 and Color3.fromRGB(255,200,50) or G.RED
        end task.wait()
    end
end)
task.spawn(function()
    while sg.Parent do
        pcall(function()
            local p=math.round(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
            pingLbl.Text="PING: "..p.."ms"
            pingLbl.TextColor3=p<80 and G.ON or p<150 and Color3.fromRGB(255,200,50) or G.RED
        end) task.wait(1)
    end
end)

-- Pages + tabs
local pages={} local tabBtns={} local activeTab=nil

local function switchTab(name)
    if activeTab==name then return end activeTab=name
    for n,pg in pairs(pages) do
        pg.Visible=n==name
        if n==name then
            pg.Position=UDim2.fromOffset(20*s,8*s)
            TweenService:Create(pg,TweenInfo.new(0.2,Enum.EasingStyle.Quart,Enum.EasingDirection.Out),{Position=UDim2.fromOffset(8*s,8*s)}):Play()
        end
    end
    for n,tb in pairs(tabBtns) do
        local on=n==name
        TweenService:Create(tb,TweenInfo.new(0.15),{
            BackgroundColor3=on and G.CARD2 or G.CARD,
            TextColor3=on and G.TXT or G.DIM
        }):Play()
        local st=tb:FindFirstChildOfClass("UIStroke")
        if st then TweenService:Create(st,TweenInfo.new(0.15),{Transparency=on and 0.1 or 0.7}):Play() end
    end
    playSound("rbxassetid://6895079813",0.15)
end

local TAB_ICONS={["Movement"]="RUN",["Combat"]="ATK",["Automation"]="BOT",["World & Visuals"]="VIS",["Settings"]="SET"}

local function createTab(name)
    local btn=Instance.new("TextButton",tabContainer)
    btn.Size=UDim2.new(1,0,0,30*s) btn.BackgroundColor3=G.CARD btn.BackgroundTransparency=0.2
    btn.Text=(TAB_ICONS[name] or name) btn.Font=Enum.Font.GothamBold btn.TextSize=10*s
    btn.TextColor3=G.DIM btn.BorderSizePixel=0
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7*s)
    local st=Instance.new("UIStroke",btn) st.Color=G.ACC st.Thickness=1 st.Transparency=0.7

    local pg=Instance.new("ScrollingFrame",contentArea)
    pg.Size=UDim2.new(1,-(14*s),1,-(14*s)) pg.Position=UDim2.fromOffset(8*s,8*s)
    pg.BackgroundTransparency=1 pg.ScrollBarThickness=3*s pg.ScrollBarImageColor3=G.BORD
    pg.BorderSizePixel=0 pg.Visible=false pg.CanvasSize=UDim2.new(0,0,0,0)
    local layout=Instance.new("UIListLayout",pg) layout.Padding=UDim.new(0,4*s) layout.SortOrder=Enum.SortOrder.LayoutOrder
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() pg.CanvasSize=UDim2.new(0,0,0,layout.AbsoluteContentSize.Y+16*s) end)

    btn.MouseButton1Click:Connect(function() switchTab(name) end)
    tabBtns[name]=btn pages[name]=pg return pg
end

-- Toggle row
local function createToggle(page,labelText,enabledKey,keybindKey,callback)
    local row=Instance.new("Frame",page)
    row.Size=UDim2.new(1,0,0,36*s) row.BackgroundColor3=G.CARD row.BackgroundTransparency=0.1 row.BorderSizePixel=0
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,8*s)
    local rowStroke=Instance.new("UIStroke",row) rowStroke.Color=G.BORD rowStroke.Thickness=1 rowStroke.Transparency=0.5

    local lbl=Instance.new("TextLabel",row) lbl.Size=UDim2.new(0.58,0,1,0) lbl.Position=UDim2.fromOffset(10*s,0)
    lbl.BackgroundTransparency=1 lbl.Text=labelText lbl.Font=Enum.Font.GothamBold
    lbl.TextSize=11*s lbl.TextColor3=G.TXT lbl.TextXAlignment=Enum.TextXAlignment.Left

    local defaultOn=Enabled[enabledKey]
    local pillW,pillH=34*s,18*s
    local pill=Instance.new("Frame",row) pill.Size=UDim2.fromOffset(pillW,pillH)
    pill.Position=UDim2.new(1,-pillW-10*s,0.5,-pillH/2) pill.BackgroundColor3=defaultOn and G.ON or G.OFF
    pill.BorderSizePixel=0 Instance.new("UICorner",pill).CornerRadius=UDim.new(1,0)
    local cir=Instance.new("Frame",pill) cir.Size=UDim2.fromOffset(pillH-4,pillH-4)
    cir.Position=defaultOn and UDim2.new(1,-(pillH-2),0.5,-(pillH-4)/2) or UDim2.new(0,2,0.5,-(pillH-4)/2)
    cir.BackgroundColor3=Color3.new(1,1,1) cir.BorderSizePixel=0
    Instance.new("UICorner",cir).CornerRadius=UDim.new(1,0)

    if keybindKey then
        local kBtn=Instance.new("TextButton",row) kBtn.Size=UDim2.fromOffset(26*s,18*s)
        kBtn.Position=UDim2.new(1,-pillW-40*s,0.5,-9*s) kBtn.BackgroundColor3=G.CARD2 kBtn.BackgroundTransparency=0.1
        kBtn.Text=KEYBINDS[keybindKey].Name kBtn.Font=Enum.Font.GothamBold kBtn.TextSize=8*s
        kBtn.TextColor3=G.DIM kBtn.BorderSizePixel=0
        Instance.new("UICorner",kBtn).CornerRadius=UDim.new(0,4*s)
        Instance.new("UIStroke",kBtn).Color=G.BORD
        KeyButtons[keybindKey]=kBtn
        kBtn.MouseButton1Click:Connect(function() waitingForKeybind=keybindKey kBtn.Text="..." end)
    end

    local isOn=defaultOn
    local function setVisual(state,skipCb)
        isOn=state
        TweenService:Create(pill,TweenInfo.new(0.15),{BackgroundColor3=state and G.ON or G.OFF}):Play()
        TweenService:Create(cir,TweenInfo.new(0.15,Enum.EasingStyle.Back),{
            Position=state and UDim2.new(1,-(pillH-2),0.5,-(pillH-4)/2) or UDim2.new(0,2,0.5,-(pillH-4)/2)
        }):Play()
        TweenService:Create(rowStroke,TweenInfo.new(0.15),{Color=state and G.ACC or G.BORD,Transparency=state and 0.2 or 0.5}):Play()
        if not skipCb then callback(isOn) end
    end
    VisualSetters[enabledKey]=setVisual

    local cb=Instance.new("TextButton",row) cb.Size=UDim2.new(1,0,1,0) cb.BackgroundTransparency=1 cb.Text=""
    cb.MouseButton1Click:Connect(function() isOn=not isOn Enabled[enabledKey]=isOn setVisual(isOn) playSound("rbxassetid://6895079813",0.25) end)
    return setVisual
end

-- Slider row
local function createSlider(page,labelText,minVal,maxVal,valueKey,isFloat,callback)
    local row=Instance.new("Frame",page)
    row.Size=UDim2.new(1,0,0,52*s) row.BackgroundColor3=G.CARD row.BackgroundTransparency=0.1 row.BorderSizePixel=0
    Instance.new("UICorner",row).CornerRadius=UDim.new(0,8*s)
    Instance.new("UIStroke",row).Color=G.BORD

    local defaultVal=Values[valueKey]
    local lbl=Instance.new("TextLabel",row) lbl.Size=UDim2.new(0.6,0,0,20*s) lbl.Position=UDim2.fromOffset(10*s,7*s)
    lbl.BackgroundTransparency=1 lbl.Text=labelText lbl.Font=Enum.Font.GothamBold
    lbl.TextSize=10*s lbl.TextColor3=G.TXT lbl.TextXAlignment=Enum.TextXAlignment.Left

    local vBox=Instance.new("TextBox",row) vBox.Size=UDim2.fromOffset(44*s,20*s)
    vBox.Position=UDim2.new(1,-54*s,0,7*s) vBox.BackgroundColor3=G.CARD2 vBox.BackgroundTransparency=0.1
    vBox.Text=tostring(defaultVal) vBox.Font=Enum.Font.GothamBold vBox.TextSize=10*s
    vBox.TextColor3=G.TXT vBox.ClearTextOnFocus=false vBox.BorderSizePixel=0
    Instance.new("UICorner",vBox).CornerRadius=UDim.new(0,5*s)
    Instance.new("UIStroke",vBox).Color=G.BORD

    local sbg=Instance.new("Frame",row) sbg.Size=UDim2.new(1,-20*s,0,4*s) sbg.Position=UDim2.fromOffset(10*s,36*s)
    sbg.BackgroundColor3=G.CARD2 sbg.BorderSizePixel=0 Instance.new("UICorner",sbg).CornerRadius=UDim.new(1,0)

    local pct=math.clamp((defaultVal-minVal)/(maxVal-minVal),0,1)
    local fill=Instance.new("Frame",sbg) fill.Size=UDim2.new(pct,0,1,0) fill.BackgroundColor3=G.ACC fill.BorderSizePixel=0
    Instance.new("UICorner",fill).CornerRadius=UDim.new(1,0)
    local thumb=Instance.new("Frame",sbg) thumb.Size=UDim2.fromOffset(12*s,12*s)
    thumb.Position=UDim2.new(pct,-6*s,0.5,-6*s) thumb.BackgroundColor3=G.TXT thumb.BorderSizePixel=0
    Instance.new("UICorner",thumb).CornerRadius=UDim.new(1,0)

    local drag=Instance.new("TextButton",sbg) drag.Size=UDim2.new(1,0,4,0) drag.Position=UDim2.new(0,0,-1.5,0)
    drag.BackgroundTransparency=1 drag.Text=""

    local dragging=false
    local function upd(rel)
        rel=math.clamp(rel,0,1) fill.Size=UDim2.new(rel,0,1,0) thumb.Position=UDim2.new(rel,-6*s,0.5,-6*s)
        local v=minVal+(maxVal-minVal)*rel v=isFloat and (math.floor(v*100)/100) or math.floor(v)
        vBox.Text=tostring(v) Values[valueKey]=v callback(v)
    end
    drag.MouseButton1Down:Connect(function() dragging=true end)
    UserInputService.InputEnded:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then dragging=false end end)
    UserInputService.InputChanged:Connect(function(i)
        if dragging and (i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch) then
            upd((i.Position.X-sbg.AbsolutePosition.X)/sbg.AbsoluteSize.X)
        end
    end)
    vBox.FocusLost:Connect(function()
        local n=tonumber(vBox.Text) if n then
            n=math.clamp(n,minVal,maxVal) n=isFloat and (math.floor(n*100)/100) or math.floor(n)
            local r=(n-minVal)/(maxVal-minVal) fill.Size=UDim2.new(r,0,1,0) thumb.Position=UDim2.new(r,-6*s,0.5,-6*s)
            vBox.Text=tostring(n) Values[valueKey]=n callback(n)
        else vBox.Text=tostring(Values[valueKey]) end
    end)
    SliderSetters[valueKey]=function(v)
        local r=math.clamp((v-minVal)/(maxVal-minVal),0,1) fill.Size=UDim2.new(r,0,1,0) thumb.Position=UDim2.new(r,-6*s,0.5,-6*s) vBox.Text=tostring(v)
    end
end

-- Action button
local function createActionBtn(page,text,callback)
    local btn=Instance.new("TextButton",page)
    btn.Size=UDim2.new(1,0,0,32*s) btn.BackgroundColor3=G.CARD2 btn.BackgroundTransparency=0.1
    btn.Text=text btn.Font=Enum.Font.GothamBold btn.TextSize=11*s
    btn.TextColor3=G.TXT btn.BorderSizePixel=0
    Instance.new("UICorner",btn).CornerRadius=UDim.new(0,8*s)
    Instance.new("UIStroke",btn).Color=G.BORD
    btn.MouseEnter:Connect(function() TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundTransparency=0.03}):Play() end)
    btn.MouseLeave:Connect(function() TweenService:Create(btn,TweenInfo.new(0.1),{BackgroundTransparency=0.1}):Play() end)
    btn.MouseButton1Click:Connect(function() playSound("rbxassetid://6895079813",0.25) callback(btn) end)
    return btn
end

-- Section label
local function createSection(page,text)
    local lbl=Instance.new("TextLabel",page) lbl.Size=UDim2.new(1,0,0,18*s)
    lbl.BackgroundTransparency=1 lbl.Text=text lbl.Font=Enum.Font.GothamBlack
    lbl.TextSize=9*s lbl.TextColor3=G.DIM lbl.TextXAlignment=Enum.TextXAlignment.Left
end

-- Create all tabs
local pageMovement=createTab("Movement")
local pageCombat=createTab("Combat")
local pageAuto=createTab("Automation")
local pageVisuals=createTab("World & Visuals")
local pageSettings=createTab("Settings")
switchTab("Movement")

-- MOVEMENT
createToggle(pageMovement,"Speed Boost","SpeedBoost","SPEED",function(v) if v then startSpeedBoost() else stopSpeedBoost() end end)
createSlider(pageMovement,"Boost Speed",1,100,"BoostSpeed",true,function(v) Values.BoostSpeed=v end)
createToggle(pageMovement,"Anti Ragdoll","AntiRagdoll","ANTIRAGDOLL",function(v) if v then startAntiRagdoll() else stopAntiRagdoll() end end)
createToggle(pageMovement,"Player Hover","Hover","HOVER",function(v) ToggleHover(v) end)
createSlider(pageMovement,"Hover Height",0,100,"HoverHeight",true,function(v) Values.HoverHeight=v end)
createToggle(pageMovement,"Spin Bot","SpinBot","SPIN",function(v) if v then startSpinBot() else stopSpinBot() end end)
createSlider(pageMovement,"Spin Speed",5,100,"SpinSpeed",true,function(v) Values.SpinSpeed=v end)
createToggle(pageMovement,"Unwalk Animations","Unwalk","UNWALK",function(v) if v then startUnwalk() else stopUnwalk() end end)
createToggle(pageMovement,"Infinite Jump","InfJump","INFJUMP",function(v) end)
createActionBtn(pageMovement,"TP DOWN",function(btn)
    btn.Text="Teleporting..." btn.TextColor3=G.DIM
    doTPDown()
    task.delay(0.5,function() btn.Text="TP DOWN" btn.TextColor3=G.TXT end)
end)

-- COMBAT
createToggle(pageCombat,"Bat Aimbot","BatAimbot","BATAIMBOT",function(v)
    if v then startBatAimbot() else stopBatAimbot() end
    if isMobile and mobileShortcuts.bat then mobileShortcuts.bat.TextColor3=v and G.TXT or G.DIM end
end)
createToggle(pageCombat,"Spam Bat Swings","SpamBat","SPAMBAT",function(v) if v then startSpamBat() else stopSpamBat() end end)

-- AUTOMATION
createToggle(pageAuto,"Auto Steal","AutoSteal","AUTOSTEAL",function(v) if v then startAutoSteal() else stopAutoSteal() end end)
createSlider(pageAuto,"Grab Time (Secs)",0.0,5.0,"STEAL_DURATION",true,function(v) Values.STEAL_DURATION=v end)
createSlider(pageAuto,"Steal Radius",5,100,"STEAL_RADIUS",true,function(v) Values.STEAL_RADIUS=v end)
createToggle(pageAuto,"Speed While Stealing","SpeedWhileStealing","SPEEDSTEAL",function(v) if v then startSpeedWhileStealing() else stopSpeedWhileStealing() end end)
createSlider(pageAuto,"Stealing Speed",5,50,"StealingSpeedValue",true,function(v) Values.StealingSpeedValue=v end)
createSection(pageAuto,"AUTO WALK")
createToggle(pageAuto,"Auto Walk Left","AutoWalkEnabled","AUTOLEFT",function(v)
    AutoWalkEnabled,Enabled.AutoWalkEnabled=v,v if v then startAutoWalk() else stopAutoWalk() end
    if isMobile and mobileShortcuts.left then mobileShortcuts.left.TextColor3=v and G.TXT or G.DIM end
end)
createSlider(pageAuto,"Auto Left Speed",1,100,"AutoLeftSpeed",true,function(v) Values.AutoLeftSpeed=v end)
createToggle(pageAuto,"Auto Walk Right","AutoRightEnabled","AUTORIGHT",function(v)
    AutoRightEnabled,Enabled.AutoRightEnabled=v,v if v then startAutoRight() else stopAutoRight() end
    if isMobile and mobileShortcuts.right then mobileShortcuts.right.TextColor3=v and G.TXT or G.DIM end
end)
createSlider(pageAuto,"Auto Right Speed",1,100,"AutoRightSpeed",true,function(v) Values.AutoRightSpeed=v end)
createSection(pageAuto,"AUTO PLAY")
createToggle(pageAuto,"Auto Play Left","AutoPlayLeftEnabled","AUTOPLAYLEFT",function(v)
    AutoPlayLeftEnabled,Enabled.AutoPlayLeftEnabled=v,v if v then startAutoPlayLeft() else stopAutoPlayLeft() end
    if isMobile and mobileShortcuts.playLeft then mobileShortcuts.playLeft.TextColor3=v and G.TXT or G.DIM end
end)
createToggle(pageAuto,"Auto Play Right","AutoPlayRightEnabled","AUTOPLAYRIGHT",function(v)
    AutoPlayRightEnabled,Enabled.AutoPlayRightEnabled=v,v if v then startAutoPlayRight() else stopAutoPlayRight() end
    if isMobile and mobileShortcuts.playRight then mobileShortcuts.playRight.TextColor3=v and G.TXT or G.DIM end
end)
createSlider(pageAuto,"Base Exit Distance",0,30,"AutoPlayExitDist",false,function(v) Values.AutoPlayExitDist=v end)
createSlider(pageAuto,"Play Return Speed",1,100,"AutoPlayReturnSpeed",true,function(v) Values.AutoPlayReturnSpeed=v end)
createSlider(pageAuto,"Play Wait Time",0.0,10.0,"AutoPlayWaitTime",true,function(v) Values.AutoPlayWaitTime=v end)
createSlider(pageAuto,"Walk Return Speed",1,100,"AutoWalkReturnSpeed",true,function(v) Values.AutoWalkReturnSpeed=v end)
createSlider(pageAuto,"Walk Wait Time",0.0,10.0,"AutoWalkWaitTime",true,function(v) Values.AutoWalkWaitTime=v end)

-- VISUALS
createToggle(pageVisuals,"Speed Meter","SpeedMeter","SPEEDMETER",function(v) toggleSpeedMeter(v) end)
createToggle(pageVisuals,"ESP & Hitbox","ESP","ESP",function(v) toggleESP(v) end)
createToggle(pageVisuals,"Show FPS/Ping","Stats","STATS",function(v) StatsFrame.Visible=v end)
createSlider(pageVisuals,"Field of View",10,120,"FOV",true,function(v) Values.FOV=v updateFOV() end)
createToggle(pageVisuals,"Galaxy Mode","Galaxy","GALAXY",function(v) if v then startGalaxy() else stopGalaxy() end end)
createSlider(pageVisuals,"Gravity %",10,150,"GalaxyGravityPercent",false,function(v) Values.GalaxyGravityPercent=v if galaxyEnabled then adjustGalaxyJump() end end)
createSlider(pageVisuals,"Hop Power",10,100,"HOP_POWER",true,function(v) Values.HOP_POWER=v end)
createToggle(pageVisuals,"Galaxy Sky","GalaxySkyBright","GALAXY_SKY",function(v) if v then enableGalaxySkyBright() else disableGalaxySkyBright() end end)
createToggle(pageVisuals,"Optimizer","Optimizer","OPTIMIZER",function(v) if v then enableOptimizer() else disableOptimizer() end end)

-- SETTINGS
createActionBtn(pageSettings,"Save Config",function(btn)
    local ok=SaveConfig() btn.Text=ok and "Saved!" or "Failed"
    btn.TextColor3=ok and G.ON or G.RED
    task.delay(2,function() btn.Text="Save Config" btn.TextColor3=G.TXT end)
end)
createActionBtn(pageSettings,"Close GUI",function() guiVisible=false main.Visible=false end)
local infoLbl=Instance.new("TextLabel",pageSettings) infoLbl.Size=UDim2.new(1,0,0,30*s)
infoLbl.BackgroundTransparency=1 infoLbl.Text="[U] toggle menu  |  click keybind to rebind"
infoLbl.Font=Enum.Font.GothamMedium infoLbl.TextSize=9*s infoLbl.TextColor3=G.DIM infoLbl.TextWrapped=true

-- Mobile
local mobileShortcuts={}
if isMobile then
    local btns={
        {id="hover",txt="Hover",  pos=UDim2.new(1,-125,0.58,-120)},
        {id="bat",  txt="AutoBat",pos=UDim2.new(1,-65,0.58,-120)},
        {id="playLeft",txt="PlayL",pos=UDim2.new(1,-125,0.58,-60)},
        {id="playRight",txt="PlayR",pos=UDim2.new(1,-65,0.58,-60)},
        {id="left", txt="AutoL",  pos=UDim2.new(1,-125,0.58,0)},
        {id="right",txt="AutoR",  pos=UDim2.new(1,-65,0.58,0)},
    }
    for _,bd in ipairs(btns) do
        local btn=Instance.new("TextButton",sg)
        btn.Size=UDim2.fromOffset(52,52) btn.Position=bd.pos
        btn.BackgroundColor3=G.CARD btn.BackgroundTransparency=0.08
        btn.Text=bd.txt btn.Font=Enum.Font.GothamBold btn.TextSize=10
        btn.TextColor3=G.DIM btn.BorderSizePixel=0
        Instance.new("UICorner",btn).CornerRadius=UDim.new(0,10)
        Instance.new("UIStroke",btn).Color=G.BORD
        mobileShortcuts[bd.id]=btn
    end
    mobileShortcuts.hover.MouseButton1Click:Connect(function()
        local ns=not Enabled.Hover Enabled.Hover=ns if VisualSetters.Hover then VisualSetters.Hover(ns,true) end
        ToggleHover(ns) mobileShortcuts.hover.TextColor3=ns and G.TXT or G.DIM
    end)
    mobileShortcuts.bat.MouseButton1Click:Connect(function()
        local ns=not Enabled.BatAimbot Enabled.BatAimbot=ns if VisualSetters.BatAimbot then VisualSetters.BatAimbot(ns,true) end
        if ns then startBatAimbot() else stopBatAimbot() end mobileShortcuts.bat.TextColor3=ns and G.TXT or G.DIM
    end)
    mobileShortcuts.left.MouseButton1Click:Connect(function()
        local ns=not Enabled.AutoWalkEnabled AutoWalkEnabled,Enabled.AutoWalkEnabled=ns,ns
        if VisualSetters.AutoWalkEnabled then VisualSetters.AutoWalkEnabled(ns,true) end
        if ns then startAutoWalk() else stopAutoWalk() end mobileShortcuts.left.TextColor3=ns and G.TXT or G.DIM
    end)
    mobileShortcuts.right.MouseButton1Click:Connect(function()
        local ns=not Enabled.AutoRightEnabled AutoRightEnabled,Enabled.AutoRightEnabled=ns,ns
        if VisualSetters.AutoRightEnabled then VisualSetters.AutoRightEnabled(ns,true) end
        if ns then startAutoRight() else stopAutoRight() end mobileShortcuts.right.TextColor3=ns and G.TXT or G.DIM
    end)
    mobileShortcuts.playLeft.MouseButton1Click:Connect(function()
        local ns=not Enabled.AutoPlayLeftEnabled AutoPlayLeftEnabled,Enabled.AutoPlayLeftEnabled=ns,ns
        if VisualSetters.AutoPlayLeftEnabled then VisualSetters.AutoPlayLeftEnabled(ns,true) end
        if ns then startAutoPlayLeft() else stopAutoPlayLeft() end mobileShortcuts.playLeft.TextColor3=ns and G.TXT or G.DIM
    end)
    mobileShortcuts.playRight.MouseButton1Click:Connect(function()
        local ns=not Enabled.AutoPlayRightEnabled AutoPlayRightEnabled,Enabled.AutoPlayRightEnabled=ns,ns
        if VisualSetters.AutoPlayRightEnabled then VisualSetters.AutoPlayRightEnabled(ns,true) end
        if ns then startAutoPlayRight() else stopAutoPlayRight() end mobileShortcuts.playRight.TextColor3=ns and G.TXT or G.DIM
    end)
end

-- Startup
task.spawn(function()
    task.wait(2)
    for k,btn in pairs(KeyButtons) do if btn and KEYBINDS[k] then btn.Text=KEYBINDS[k].Name end end
    for key,setter in pairs(VisualSetters) do if Enabled[key] then setter(true,false) else setter(false,true) end end
    for key,setter in pairs(SliderSetters) do if Values[key] then setter(Values[key]) end end
end)

-- Input
UserInputService.InputBegan:Connect(function(input,gpe)
    if gpe then return end
    if UserInputService:GetFocusedTextBox() then return end
    if waitingForKeybind and input.KeyCode~=Enum.KeyCode.Unknown then
        local k=input.KeyCode KEYBINDS[waitingForKeybind]=k
        if KeyButtons[waitingForKeybind] then KeyButtons[waitingForKeybind].Text=k.Name end
        waitingForKeybind=nil return
    end
    if input.KeyCode==Enum.KeyCode.U then guiVisible=not guiVisible main.Visible=guiVisible return end
    if input.KeyCode==Enum.KeyCode.Space then spaceHeld=true return end
    local function tog(ek,sf,stf)
        Enabled[ek]=not Enabled[ek] if VisualSetters[ek] then VisualSetters[ek](Enabled[ek]) end
        if Enabled[ek] then if sf then sf() end else if stf then stf() end end
    end
    if input.KeyCode==KEYBINDS.SPEED then tog("SpeedBoost",startSpeedBoost,stopSpeedBoost) end
    if input.KeyCode==KEYBINDS.SPIN then tog("SpinBot",startSpinBot,stopSpinBot) end
    if input.KeyCode==KEYBINDS.GALAXY then tog("Galaxy",startGalaxy,stopGalaxy) end
    if input.KeyCode==KEYBINDS.BATAIMBOT then tog("BatAimbot",startBatAimbot,stopBatAimbot) end
    if input.KeyCode==KEYBINDS.NUKE then local n=getNearestPlayer() if n then INSTANT_NUKE(n) end end
    if input.KeyCode==KEYBINDS.AUTOLEFT then
        AutoWalkEnabled=not AutoWalkEnabled Enabled.AutoWalkEnabled=AutoWalkEnabled
        if VisualSetters.AutoWalkEnabled then VisualSetters.AutoWalkEnabled(AutoWalkEnabled) end
        if AutoWalkEnabled then startAutoWalk() else stopAutoWalk() end
    end
    if input.KeyCode==KEYBINDS.AUTORIGHT then
        AutoRightEnabled=not AutoRightEnabled Enabled.AutoRightEnabled=AutoRightEnabled
        if VisualSetters.AutoRightEnabled then VisualSetters.AutoRightEnabled(AutoRightEnabled) end
        if AutoRightEnabled then startAutoRight() else stopAutoRight() end
    end
    if input.KeyCode==KEYBINDS.AUTOPLAYLEFT then
        AutoPlayLeftEnabled=not AutoPlayLeftEnabled Enabled.AutoPlayLeftEnabled=AutoPlayLeftEnabled
        if VisualSetters.AutoPlayLeftEnabled then VisualSetters.AutoPlayLeftEnabled(AutoPlayLeftEnabled) end
        if AutoPlayLeftEnabled then startAutoPlayLeft() else stopAutoPlayLeft() end
    end
    if input.KeyCode==KEYBINDS.AUTOPLAYRIGHT then
        AutoPlayRightEnabled=not AutoPlayRightEnabled Enabled.AutoPlayRightEnabled=AutoPlayRightEnabled
        if VisualSetters.AutoPlayRightEnabled then VisualSetters.AutoPlayRightEnabled(AutoPlayRightEnabled) end
        if AutoPlayRightEnabled then startAutoPlayRight() else stopAutoPlayRight() end
    end
    if input.KeyCode==KEYBINDS.ANTIRAGDOLL then tog("AntiRagdoll",startAntiRagdoll,stopAntiRagdoll) end
    if input.KeyCode==KEYBINDS.SPEEDSTEAL then tog("SpeedWhileStealing",startSpeedWhileStealing,stopSpeedWhileStealing) end
    if input.KeyCode==KEYBINDS.AUTOSTEAL then tog("AutoSteal",startAutoSteal,stopAutoSteal) end
    if input.KeyCode==KEYBINDS.UNWALK then tog("Unwalk",startUnwalk,stopUnwalk) end
    if input.KeyCode==KEYBINDS.OPTIMIZER then tog("Optimizer",enableOptimizer,disableOptimizer) end
    if input.KeyCode==KEYBINDS.SPAMBAT then tog("SpamBat",startSpamBat,stopSpamBat) end
    if input.KeyCode==KEYBINDS.GALAXY_SKY then tog("GalaxySkyBright",enableGalaxySkyBright,disableGalaxySkyBright) end
    if input.KeyCode==KEYBINDS.INFJUMP then tog("InfJump") end
    if input.KeyCode==KEYBINDS.ESP then tog("ESP",function() toggleESP(true) end,function() toggleESP(false) end) end
    if input.KeyCode==KEYBINDS.HOVER then tog("Hover",function() ToggleHover(true) end,function() ToggleHover(false) end) end
    if input.KeyCode==KEYBINDS.SPEEDMETER then tog("SpeedMeter",function() toggleSpeedMeter(true) end,function() toggleSpeedMeter(false) end) end
    if input.KeyCode==KEYBINDS.STATS then tog("Stats") if StatsFrame then StatsFrame.Visible=Enabled.Stats end end
end)
UserInputService.InputEnded:Connect(function(input) if input.KeyCode==Enum.KeyCode.Space then spaceHeld=false end end)

Player.CharacterAdded:Connect(function(char)
    task.wait(1)
    if Enabled.SpinBot then stopSpinBot() task.wait(0.1) startSpinBot() end
    if Enabled.Galaxy then setupGalaxyForce() adjustGalaxyJump() end
    if Enabled.SpamBat then stopSpamBat() task.wait(0.1) startSpamBat() end
    if Enabled.BatAimbot then stopBatAimbot() task.wait(0.1) startBatAimbot() end
    if Enabled.Unwalk then startUnwalk() end
    if Enabled.Hover then ToggleHover(true) end
    if Enabled.SpeedMeter then toggleSpeedMeter(false) task.wait(0.1) toggleSpeedMeter(true) end
    if Enabled.SpeedBoost then stopSpeedBoost() task.wait(0.1) startSpeedBoost() end
    if Enabled.SpeedWhileStealing then stopSpeedWhileStealing() task.wait(0.1) startSpeedWhileStealing() end
    if Enabled.AntiRagdoll then stopAntiRagdoll() task.wait(0.1) startAntiRagdoll() end
    if Enabled.AutoWalkEnabled then stopAutoWalk() task.wait(0.1) startAutoWalk() end
    if Enabled.AutoRightEnabled then stopAutoRight() task.wait(0.1) startAutoRight() end
    if Enabled.AutoPlayLeftEnabled then stopAutoPlayLeft() task.wait(0.1) startAutoPlayLeft() end
    if Enabled.AutoPlayRightEnabled then stopAutoPlayRight() task.wait(0.1) startAutoPlayRight() end
end)
end)
