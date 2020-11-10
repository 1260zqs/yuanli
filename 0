--@type UIAdventure
local UIAdventure = {}
extends(UIAdventure, UIBase)


function UIAdventure:OnInit()
  self.mTabs          = {}
  self.mOrderedList   = {}
  self.mDestinations  = {}
  self.mDropList      = {}
  self.mLevelsUI      = {}
  self.mMap           = {}
  self.mCombat        = Game:GetProcedure().mCombat
  self.mListener      = self.mCombat.mCookie:GetListener()

  self.sTexGrayColor  = Color(159/255, 130/255, 125/255, 1)
  self.sTexHotColor   = Color(255/255, 246/255, 217/255, 1)

  self.sLevelGrayImg = "UIImage/FrameAtlas/icon_map_point03.png"
  self.sLevelHotImg = "UIImage/FrameAtlas/icon_map_point02.png"

  self.mGiftImage ={
    "UIImage/FrameAtlas/icon_Ad_reward02.png",
    "UIImage/FrameAtlas/icon_Ad_reward01.png",
  }
  self.mLevelIcon = {
    [ExploreMgr.LevelType.Mob] = "UIImage/FrameAtlas/icon_map_boss.png",
    [ExploreMgr.LevelType.Boss]       = "UIImage/FrameAtlas/icon_map_boss.png",
    [ExploreMgr.LevelType.Gold]       = "UIImage/FrameAtlas/icon_map_award1.png",
    [ExploreMgr.LevelType.Gem]        = "UIImage/FrameAtlas/icon_map_award2.png",
    [ExploreMgr.LevelType.Box]        = "UIImage/FrameAtlas/icon_map_award3.png",
  }
  self.sLevelGrayTex = Color(239/255, 240/255, 248/255, 1)
  self.sLevelHotTex = Color(255/255, 241/255, 144/255, 1)

  for i = 1, 2 do
    local path  = "Panel/ListTab/"..i
    local tab   = {}
    tab.img     = self:GetUIImage(path)
    tab.btn     = self:GetUIButton(path)
    tab.text    = self:GetTextMeshPro(path.."/Text")
    path        = "Panel/Content/" .. i
    tab.content = self:GetGameObject(path)
    self.mTabs[i] = tab
  end

  local content1 = self.mTabs[1].content.transform
  local content2 = self.mTabs[2].content.transform

  local callbacks = {
  onContentActive = MAKE_CALLBACK(self.OnContentActive,self),
  onSetTabStyle = MAKE_CALLBACK(self.OnSetTabStyle,self)
  }
  self.mTabSwitcher = Switcher.new(self, self.mTabs,callbacks)--tab选项卡

  self.mTabSwitcher.sTabGrayColor  = nil
  self.mTabSwitcher.sTabHotColor   = nil

  local parent = self:GetTransform("Map/Charpters", content1)
  for i = 1, 5 do
    local item = {
      --text  = self:GetTextMeshPro(string.format("c%d/Process%d/Text", cId, i), parent),
      --name  = self:GetTextMeshPro(string.format("c%d/Destination%d/name", cId, i), parent),
      icon  = self:GetUIImage(string.format("c1/Destination%d",i),parent),
      gift  = self:GetUIImage(string.format("c1/Destination%d/award", i),parent),
      bar   = self:GetControl("CurvePath", string.format("c1/Process%d", i), parent),
      point = self:GetRectTransform(string.format("c1/Destination%d/point", i), parent),
    }
    table.insert(self.mMap,item)
  end
  parent     = self:GetTransform("Levels/Mask/Node",content1)
  local count = parent.childCount
  for i = 1, 10 do
    local item = {}
    local path = tostring(i)
    item.img = self:GetUIImage(path, parent)
    --item.obj = item.img.gameObject
    path = string.format("%d/Text", i)
    item.text = self:GetTextMeshPro(path, parent)
    path = string.format("%d/Boss", i)
    item.boss = self:GetGameObject(path, parent)
    item.anim = self:GetAnimator(path,parent)
    item.icon = self:GetUIImage(path,parent)
    path = string.format("%d/Point", i)
    item.point = self:GetTransform(path, parent)
    table.insert(self.mLevelsUI, item)
  end

  for i = 1, 4 do
    local path = string.format("Rewards/Items/%d", i)
    local item = {}
    local parent = self:GetTransform(path,content1)
    item = UIComponet:RegistEquipItem(self,parent)
    table.insert(self.mDropList, item)
  end

  --航行界面的内容
  self.mChapterText   = self:GetTextMeshPro("Map/Title/Text",content1) --章节名
  local actions       = self:GetTransform("Actions",content1)
  self.mExploreBtn    = self:GetUIButton("Go", actions)  --探索按钮
  self.mJourneyList   = self:GetUIListView("",content2)
  self.mBoatUI        = self:GetRectTransform("Map/boat",content1)
  --self.mFlagUI        = self:GetRectTransform("Flag",content1)
  self.mCharacter     = self:GetUIButton("Character",actions)
  self.mFlagUI = {
  rect   = self:GetRectTransform("Flag",content1),
  full  = self:GetGameObject("Flag/1",content1),
  half  = self:GetGameObject("Flag/2",content1),
  SetFull = function(isfull)
    self.mFlagUI.full:SetActive(isfull)
    self.mFlagUI.half:SetActive(not isfull)
  end
  }
  self.mLevelsUINode = self:GetRectTransform("Levels/Mask/Node",content1)
  self.mFlagJump = self:GetRectTransform("_FlagJump",content1)
end

function UIAdventure:OnOpen(anim)
  --刷新界面信息
  --self.mDoAnim = anim
  self:SetExploreProcess()
  self:SortList()
  self.mTabSwitcher:SetActiveTab(1)
  --self.mCharpterSwitcher:SetActiveTab(ExploreMgr.mCurChapter)
  self.mJourneyList:InitListView(0, function(listView, itemIndex)
    if itemIndex < 0 then return nil end
    local item = listView:NewListViewItem("JourneyItem")
    local parent = item.transform
    local itemnode = nil
    if item.IsInitHandlerCalled == false then
      item.IsInitHandlerCalled = true
      itemnode = self:RegistJourneyItem(parent)
      item.UserObjectData = itemnode
    end
    if itemnode == nil then itemnode = item.UserObjectData end
    local cfg = self.mOrderedList[itemIndex + 1]
    if cfg == nil then return nil end
    self:SetJourneyItemData(itemnode, cfg)
    return item
  end)
  self.mJourneyList:SetListItemCount(#ExploreMgr.mCharpters)
  self:OnClick(self:GetUIButton("Panel/Close"),function () self:Close() end)
  self:OnClick(self:GetUIButton("Panel/Content/1/Actions/Team"),function()
    UI.Open("CombatHeroSet")
  end)
  self.mCharacter.gameObject:SetActive(CookieMgr.Get("player"):IsUnlockFunc(FuncType.HangUp))
  self:OnClick(self.mCharacter,function()
    UI.Open("HeroBag")
  end)
  self:OnClick(self.mExploreBtn,function()
    --ExploreMgr:DoBattle({})
    if ExploreMgr:CanPVE() then
      local dataMgr = CookieMgr.Get("hero")
      --local dataLst = dataMgr:Get("list")
      local data_lst = dataMgr:GetCombatHeroLst()
      local lst = {}
      for k, _ in pairs(data_lst) do
        lst[#lst + 1] = k
      end
      if #lst > 0 then
          NetworkMgr:get_msg("player"):req_startPve(1, lst)
      else
          UI.Tip("请先选择上阵英雄")
      end
    end
  end)
  GuideMgr:OnUIOpen("Adventure",self)
end

--local time = 0.1
--function UIAdventure:OnUpdate()
--  time = time - Fuce.lockTime
--  if time < 0 then
--    time = 0.1
--    ExploreMgr:Init(self.mCombat,ExploreMgr.mCurLevel + 1)
--    self:SetExploreProcess()
--  end
--end

function UIAdventure:SetMapData()
  local cId = ExploreMgr.mCurChapter
  local charpter = ExploreMgr:GetCharpater()
  local cfg = charpter.cfg
  self.mChapterText.text = string.format("%d.%s",cId, cfg.Name)
  local sailConfig = cfg.SailConfig

  --for i, value in pairs(sailConfig) do
  --  local item = self.mMap[i]
  --  self:SetUIImage(item.gift,self.mGiftImage[value[3]])
  --  local levelId   = value[1]
  --  item.bar.background.fillAmount = levelId < ExploreMgr.mCurLevel and 1 or 0
  --end
  local index
  local s,e
  for i, item in pairs(self.mMap) do
    local cfg = sailConfig[i]
    if cfg then
      item.bar.gameObject:SetActive(true)
      item.icon.gameObject:SetActive(true)
      self:SetUIImage(item.gift,self.mGiftImage[cfg[3]])--gift
      self:SetUIImage(item.icon,cfg[4])--boss
      item.bar.background.fillAmount = cfg[1] < ExploreMgr.mCurLevel and 1 or 0
      if ExploreMgr.mCurLevel <= cfg[1] and not index then
        index = cfg[1]
        s = i - 1
        e = i
      end
    else
      item.bar.gameObject:SetActive(false)
      item.icon.gameObject:SetActive(false)
    end
  end

  local boatIndex = math.clamp(#charpter.unlockedBoss + 1,1,#self.mMap)
  local point = self.mMap[boatIndex].point
  local item = self.mMap[boatIndex]

  local S
  if s <= 0 then
    if cId == 1 then
      S = 0
    else
      local chap = ExploreMgr.mCharpters[cId - 1]
      S = chap.lastLevel.Id
    end
  else
    S = sailConfig[s][1]
  end
  local E = sailConfig[e][1]
  local M = ExploreMgr.mCurLevel - S
  local percent = M / (E - S)
  self:SetBarProcess(item.bar,percent)
  --local s = charpter.startLevel.Id
  --local e = charpter.lastLevel.Id
  --local l = ExploreMgr:GetRelativeLevelId(ExploreMgr.mCurLevel) - 1
  --self:SetBarProcess(item.bar,math.clamp01(l / (e - s + 1)))
end

function UIAdventure:SetBarProcess(bar,percent)
  local value = (bar.curve:Evaluate(percent) * 2 - 1) * bar.height
  bar:ReCalc()

  local pos = bar.background.rectTransform.position;
  pos.x = pos.x + bar.startOffsetX + percent * (bar.endOffsetX - bar.startOffsetX);
  pos.y = pos.y + value

  self.mBoatUI.position = pos
  bar.background.fillAmount = percent;
end

function UIAdventure:SetExploreProcess()
  self:SetMapData()
  --关卡进度条--------
  local levels = DATA.GetData("BattleLevel")
  local charpter = ExploreMgr:GetCharpater()

  local half = math.floor(9 / 2)
  local mid = math.ceil(9 / 2)
  local len = charpter.lastLevel.Id
  local start = charpter.startLevel.Id
  local startPoint = math.max(start,ExploreMgr.mCurLevel - half)
  local lastPoint  =  math.min(len,startPoint + half * 2)
  local isSub
  if ExploreMgr.mCurLevel + half >= len then
    startPoint = len - 9 + 1
    lastPoint = len
  else
    --local sv = ExploreMgr:GetRelativeLevelId(startPoint)
    --if math.ceil((lastPoint - startPoint + 1) / 2) == mid and sv > 1 then
    --  startPoint = startPoint - 1
    --  isSub = true
    --end
  end

  local index = 1
  for i = startPoint, lastPoint do
    local level = levels[i]
    local item = self.mLevelsUI[index]

    local show = level.Type > ExploreMgr.LevelType.Tiny
    show = show and level.Id >= ExploreMgr.mCurLevel
    item.boss:SetActive(show)
    item.anim.enabled = show
    if show then
      self:SetUIImage(item.icon,self.mLevelIcon[level.Type])
    end
    item.text.text = ExploreMgr:GetRelativeLevelId(level.Id)
    local color = level.Id <= ExploreMgr.mCurLevel and self.sLevelHotTex or self.sLevelGrayTex
    local img = level.Id <= ExploreMgr.mCurLevel and self.sLevelHotImg or self.sLevelGrayImg
    if level.Id == ExploreMgr.mCurLevel then
      local target = self.mLevelsUI[index]
      -----ARK-422-动画-----
      if self.mDoAnim then
        self.mFlagUI.SetFull(false)
        local done = function()
          self.mDoAnim = nil
          self.mFlagUI.SetFull(true)
        end
        local s = ExploreMgr:GetRelativeLevelId(startPoint)
        s = isSub and s + 1 or s
        local animType = self:GetAnimType(isSub and index - 1 or index,mid,s)
        if animType == -1 or animType == 1 then
          local prev = self.mLevelsUI[math.max(1,index - 1)]
          self.mFlagUI.rect.position = prev.point.position
          self.mFlagUI.rect:DOMoveX(target.point.position.x,1):SetEase(CS.DG.Tweening.Ease.Linear):OnComplete(done)
          local teen = self.mFlagUI.rect:DOMoveY(self.mFlagJump.position.y,0.5):OnComplete(function ()
            self.mFlagUI.rect:DOMoveY(prev.point.position.y,0.5):SetEase(CS.DG.Tweening.Ease.InQuad)
          end)
        elseif animType == 0 then
          local prev = self.mLevelsUI[isSub and index - 2 or index]
          local prevT = self.mLevelsUI[isSub and index - 1 or index]
          self.mFlagUI.rect.position = prevT.point.position
          FTools.SetAnchorPosition(self.mLevelsUINode,0,0)
          self.mLevelsUINode:DOMoveX(prev.img.transform.position.x,1):SetEase(CS.DG.Tweening.Ease.Linear):OnComplete(done)
          local teen = self.mFlagUI.rect:DOMoveY(self.mFlagJump.position.y,0.5)
          teen:OnComplete(function ()
            self.mFlagUI.rect:DOMoveY(prev.point.position.y,0.5):SetEase(CS.DG.Tweening.Ease.InQuad)
          end)
        else
          self.mFlagUI.rect.position = target.point.position
        end
      else
        self.mFlagUI.rect.position = target.point.position
      end
      -----ARK-422-动画-----
    end
    item.text.color = color
    self:SetUIImage(item.img,img)
    index = index + 1
  end
  --关卡进度条--end------
  local cfg = ExploreMgr:GetBattleCfg()
  self:SetDropItem(cfg)
end

function UIAdventure:GetAnimType(index,half,start)
  if index < half then
    return -1
  elseif index == half then -- and level.Id > 1 then
    if start > 1 then
      return 0
    else
      return -1
    end
  elseif index > half then
    return 1
  end
end

function UIAdventure:SetDropItem(cfg)
  for i = 1, 4 do
    local item = self.mDropList[i]
    item.obj:SetActive(true)
    if i == 1 then
      if cfg.PassRewardExp then
        local data = {}
        data.id = IDDefine.Exp
        data.count = cfg.PassRewardExp
        UIComponet:SetEquipItemData(self,item, data)
      else
        item.obj:SetActive(false)
      end
    elseif i == 2 then
      if cfg.PassRewardGold then
        local data = {}
        data.id = IDDefine.Gold
        data.count = cfg.PassRewardGold
        UIComponet:SetEquipItemData(self,item, data)
      else
        item.obj:SetActive(false)
      end
    elseif i == 3 then
      if cfg.PassRewardItem and cfg.PassRewardItem[2] then
        local data = {}
        data.id = cfg.PassRewardItem[1]
        data.count = cfg.PassRewardItem[2]
        UIComponet:SetEquipItemData(self,item, data)
      else
        item.obj:SetActive(false)
      end
    elseif i == 4 then
      item.obj:SetActive(false)
    end
  end
end

function UIAdventure:RegistJourneyItem(parent)
  local item = {}
  item.lock           = self:GetGameObject("Lock",parent)
  item.icon           = self:GetUIImage("Content/icon",parent)
  item.done           = self:GetGameObject("Content/Done",parent)
  item.doing          = self:GetGameObject("Content/Doing",parent)
  item.title          = self:GetTextMeshPro("Content/Name",parent)
  item.title2         = self:GetTextMeshPro("Lock/Name",parent)
  item.processBar     = self:GetControl("Slider","Content/Process",parent)
  item.processText    = self:GetTextMeshPro("Content/Process/Value",parent)
  item.content        = self:GetGameObject("Content",parent)
  return item
end

function UIAdventure: SetJourneyItemData(itemnode, charpter)

  local cfg = charpter.cfg
  local percent = 0
  local cId = ExploreMgr:GetCharpterIdByLevel(ExploreMgr.mCurLevel)

  if cId > charpter.cfg.Id then
    percent = 1
  elseif cId == charpter.cfg.Id then
    if charpter.complete then
      percent = 1
    else
      local s = charpter.startLevel.Id
      local e = charpter.lastLevel.Id
      local l = ExploreMgr:GetRelativeLevelId(ExploreMgr.mCurLevel) - 1
      percent = math.clamp01(l / (e - s + 1))
    end
  else
    percent = 0
  end

  itemnode.title.text = cfg.Name
  itemnode.title2.text = cfg.Name
  itemnode.processBar.value = percent
  local sailCfg = cfg.SailConfig
  local icon = sailCfg[#sailCfg][4]
  self:SetUIImage(itemnode.icon,icon)
  itemnode.processText.text = math.floor(percent * 100) .. "%"
  if charpter.complete then
    itemnode.done:SetActive(true)
    itemnode.content:SetActive(true)
    itemnode.lock:SetActive(false)
    itemnode.doing:SetActive(false)
    return
  end
  if cfg.Id > ExploreMgr.mCurChapter then
    itemnode.lock:SetActive(true)
    itemnode.done:SetActive(false)
    itemnode.doing:SetActive(false)
    itemnode.content:SetActive(false)
    return
  end
  itemnode.doing:SetActive(true)
  itemnode.lock:SetActive(false)
  itemnode.done:SetActive(false)
  itemnode.content:SetActive(true)
end

function UIAdventure:SortList()
  local levels = ExploreMgr.mCharpters
  local list = {}
  if levels == nil then
    self.mOrderedList = list
    return
  end
  for i, v in pairs(levels) do
    table.insert(list,v)
  end
  table.sort(list,function (a,b)
    local i1 = a.complete and -1 or 1
    i1 = a.cfg.Id == ExploreMgr.mCurChapter and 2 or i1
    local i2 = b.complete and -1 or 1
    i2 = b.cfg.Id == ExploreMgr.mCurChapter and 2 or i2
    if i1 == i2 then
      return a.cfg.Id < b.cfg.Id
    else
      return i1 > i2
    end
  end)
  self.mOrderedList = list
end

function UIAdventure:OnContentActive(index,switcher)
  if index == 1 then
    self:SetExploreProcess()
  elseif index == 2 then
    self:SortList()
    self.mJourneyList:SetListItemCount(#self.mOrderedList)
    self.mJourneyList:RefreshAllShownItem()
  end
end

function UIAdventure:OnSetTabStyle(tab, selected, switcher)
  local textColor = selected and self.sTexHotColor or self.sTexGrayColor
  tab.text.color = textColor

  if selected then
    local sprite = tab.btn.spriteState.selectedSprite
    tab.btn.image.sprite = sprite
  else
    local sprite = tab.btn.spriteState.highlightedSprite
    tab.btn.image.sprite = sprite
  end
end

function UIAdventure:clear()
  self.mListener:Release()
end
function UIAdventure:OnClose()
  self:clear()
  self.mJourneyList:Clear()
  self.mJourneyList   = nil
  self.mTabs          = nil
  self.mOrderedList   = nil
  self.mDestinations  = nil
  self.mDropList      = nil
  self.mLevelsUI      = nil
  self.mMap           = nil
  self.mCombat        = nil
  self.mListener      = nil
  self.mTabSwitcher   = nil
  self.sTexGrayColor  = nil
  self.sTexHotColor   = nil
  GuideMgr:OnUIClose("Adventure")
end
return UIAdventure
