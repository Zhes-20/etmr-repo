# EnJoyTheMR — Примеры приложений

Платформа поддерживает два пути создания приложений: **чистый Lua** и **Unity SDK** (LuaBehaviour + AssetBundle).
Примеры ниже показывают оба варианта с рабочим кодом.

---

## 1. Hello World — Чистый Lua (Window Mode)

Минимальное приложение: одна кнопка, один текст.

**`app.json`**
```json
{
  "id": "com.example.hello",
  "name": "Hello World",
  "version": "1.0.0",
  "author": "Developer",
  "entry": "scripts/main.lua",
  "display": "window",
  "minApiVersion": 1,
  "type": "lua",
  "permissions": ["ui"]
}
```

**`scripts/main.lua`**
```lua
local clicks = 0

local panel = EMRAPI.UI:CreatePanel(1.0, 0.6)
EMRAPI.UI:SetColor(panel, 0.1, 0.1, 0.15, 0.9)

local label = EMRAPI.UI:CreateText(panel, "Кликов: 0")
EMRAPI.UI:SetPosition(label, 0, 80, 0)
EMRAPI.UI:SetSize(label, 400, 60)

local btn = EMRAPI.UI:CreateButton(panel, "Кликни!", function()
    clicks = clicks + 1
    EMRAPI.UI:SetText(label, "Кликов: " .. clicks)
end)
EMRAPI.UI:SetPosition(btn, 0, -30, 0)
EMRAPI.UI:SetSize(btn, 250, 70)

EMRAPI:Log("Hello World запущен")
```

> **Примечание:** В чистом Lua-режиме можно инициализировать UI прямо в теле скрипта (вне OnStart) — код выполняется при загрузке. Но рекомендуется использовать `OnStart()` для порядка.

---

## 2. Dino Run — Чистый Lua-игра (Window Mode)

2D-игра с прыжками, физикой, счётом и столкновениями. Демонстрирует `OnUpdate(dt)` для покадровой логики.

**`app.json`**
```json
{
  "id": "com.example.dinorun",
  "name": "Dino Run",
  "version": "1.0.0",
  "author": "Developer",
  "entry": "scripts/main.lua",
  "display": "window",
  "minApiVersion": 1,
  "type": "lua",
  "permissions": ["ui"]
}
```

**`scripts/main.lua`**
```lua
local isGameOver = false
local score = 0

-- UI элементы
local panel, dinoObj, cactusObj, scoreText, gameOverText, clickArea

-- Физика
local GRAVITY = -1800
local JUMP_VELOCITY = 650
local GROUND_Y = -100
local DINO_START_Y = GROUND_Y + 20
local CACTUS_Y = GROUND_Y + 25

local dinoY = DINO_START_Y
local velocityY = 0
local cactusX = 250
local CACTUS_SPEED = 300

function OnStart()
    EMRAPI:Log("Dino Run started!")

    -- Панель 480x320
    panel = EMRAPI.UI:CreatePanel(0.48, 0.32)

    -- Линия земли
    local groundLine = EMRAPI.UI:CreatePanel(0.48, 0.002)
    EMRAPI.UI:SetColor(groundLine, 1, 1, 1, 1)
    EMRAPI.UI:SetPosition(groundLine, 0, GROUND_Y, 0)

    -- Счётчик
    scoreText = EMRAPI.UI:CreateText(panel, "Score: 0")
    EMRAPI.UI:SetSize(scoreText, 150, 30)
    EMRAPI.UI:SetPosition(scoreText, 150, 130, 0)

    -- Дино (зелёный квадрат 40x40)
    dinoObj = EMRAPI.UI:CreatePanel(0.04, 0.04)
    EMRAPI.UI:SetColor(dinoObj, 0.2, 0.8, 0.2, 1)

    -- Кактус (красный прямоугольник 20x50)
    cactusObj = EMRAPI.UI:CreatePanel(0.02, 0.05)
    EMRAPI.UI:SetColor(cactusObj, 0.8, 0.2, 0.2, 1)

    -- Текст проигрыша
    gameOverText = EMRAPI.UI:CreateText(panel, "GAME OVER\nTap to restart")
    EMRAPI.UI:SetSize(gameOverText, 300, 60)
    EMRAPI.UI:SetPosition(gameOverText, 50, 0, 0)
    EMRAPI.UI:SetActive(gameOverText, false)

    -- Кнопка на весь экран (прозрачная)
    clickArea = EMRAPI.UI:CreateButton(panel, "", function()
        if isGameOver then
            RestartGame()
        else
            Jump()
        end
    end)
    EMRAPI.UI:SetSize(clickArea, 480, 320)
    EMRAPI.UI:SetColor(clickArea, 0, 0, 0, 0)

    -- Начальные позиции
    EMRAPI.UI:SetPosition(dinoObj, -150, dinoY, 0)
    EMRAPI.UI:SetPosition(cactusObj, cactusX, CACTUS_Y, 0)
end

function RestartGame()
    isGameOver = false
    score = 0
    dinoY = DINO_START_Y
    velocityY = 0
    cactusX = 250
    CACTUS_SPEED = 300
    EMRAPI.UI:SetText(scoreText, "Score: 0")
    EMRAPI.UI:SetActive(gameOverText, false)
end

function Jump()
    if dinoY <= DINO_START_Y then
        velocityY = JUMP_VELOCITY
    end
end

function OnUpdate(dt)
    if isGameOver then return end

    -- Физика прыжка
    velocityY = velocityY + GRAVITY * dt
    dinoY = dinoY + velocityY * dt
    if dinoY <= DINO_START_Y then
        dinoY = DINO_START_Y
        velocityY = 0
    end
    EMRAPI.UI:SetPosition(dinoObj, -150, dinoY, 0)

    -- Движение кактуса
    cactusX = cactusX - CACTUS_SPEED * dt
    if cactusX < -250 then
        cactusX = 250
        score = score + 10
        EMRAPI.UI:SetText(scoreText, "Score: " .. score)
        CACTUS_SPEED = CACTUS_SPEED + 15
    end
    EMRAPI.UI:SetPosition(cactusObj, cactusX, CACTUS_Y, 0)

    -- AABB-столкновения
    if (-150 + 15) > (cactusX - 8)
       and (-150 - 15) < (cactusX + 8)
       and (dinoY + 15) > (CACTUS_Y - 23)
       and (dinoY - 15) < (CACTUS_Y + 23)
    then
        isGameOver = true
        EMRAPI.UI:SetActive(gameOverText, true)
    end
end

function OnDestroy()
    EMRAPI:Log("Dino Run closed. Score: " .. score)
end
```

---

## 3. Физика в 3D — Чистый Lua (Scene Mode)

Спавн физических шаров при щипке правой рукой. Демонстрирует `EMRAPI.Scene` и `EMRAPI.Input`.

**`app.json`**
```json
{
  "id": "com.example.physics",
  "name": "Physics Playground",
  "version": "1.0.0",
  "author": "Developer",
  "entry": "scripts/main.lua",
  "display": "scene",
  "minApiVersion": 1,
  "type": "lua",
  "permissions": ["scene", "input"]
}
```

**`scripts/main.lua`**
```lua
-- Пол
local floor = EMRAPI.Scene:CreatePrimitive("plane")
EMRAPI.Scene:SetScale(floor, 5, 1, 5)
EMRAPI.Scene:SetColor(floor, 0.2, 0.2, 0.25, 1.0)
EMRAPI.Scene:AddBoxCollider(floor)

EMRAPI.Timer:Repeat(0.05, function()
    if EMRAPI.Input:IsHandTracked("right") and EMRAPI.Input:GetHandGesture("right") == "Pinch" then
        local rx, ry, rz = EMRAPI.Input:GetHandPosition("right", "PalmCenter")

        local ball = EMRAPI.Scene:CreatePrimitive("sphere")
        EMRAPI.Scene:SetPosition(ball, rx, ry, rz)
        EMRAPI.Scene:SetScale(ball, 0.08, 0.08, 0.08)
        EMRAPI.Scene:SetColor(ball,
            EMRAPI.Math:Random(0,1), EMRAPI.Math:Random(0,1), EMRAPI.Math:Random(0,1), 1.0)
        EMRAPI.Scene:AddSphereCollider(ball, 0.04)
        EMRAPI.Scene:AddRigidbody(ball, 0.5, true)
    end
end)
```

---

## 4. Объект на руке — Чистый Lua (Scene Mode)

Куб, плавно следующий за центром правой ладони.

**`app.json`** — `permissions: ["scene", "input"]`, `display: "scene"`

**`scripts/main.lua`**
```lua
local cube = EMRAPI.Scene:CreatePrimitive("cube")
EMRAPI.Scene:SetScale(cube, 0.05, 0.05, 0.05)
EMRAPI.Scene:SetColor(cube, 0.3, 0.7, 1.0, 1.0)

-- Плавно следует за центром правой ладони
EMRAPI.Scene:AttachToHandSmooth(cube, "right", "PalmCenter", 12.0, 10.0)

function OnDestroy()
    EMRAPI:Log("Hand cube closed")
end
```

---

## 5. Сохранение данных — Чистый Lua

Счётчик, который не сбрасывается при перезапуске.

**`app.json`** — `permissions: ["ui", "storage"]`

**`scripts/main.lua`**
```lua
local count = EMRAPI.Storage:GetNumber("count", 0)

local panel = EMRAPI.UI:CreatePanel(0.8, 0.5)
local label = EMRAPI.UI:CreateText(panel, "Счёт: " .. count)
EMRAPI.UI:SetPosition(label, 0, 60, 0)
EMRAPI.UI:SetSize(label, 350, 50)

local btn = EMRAPI.UI:CreateButton(panel, "+1", function()
    count = count + 1
    EMRAPI.UI:SetText(label, "Счёт: " .. count)
    EMRAPI.Storage:SetNumber("count", count)
    EMRAPI.Storage:Save()  -- ОБЯЗАТЕЛЬНО вызвать для записи на диск
end)
EMRAPI.UI:SetPosition(btn, 0, -20, 0)
EMRAPI.UI:SetSize(btn, 200, 60)
```

---

## 6. Счётчик — Unity SDK (Window Mode)

Тот же счётчик, но UI собран мышкой в Unity. Lua — только логика.
Находится в `Assets/ETMR/DevApp/WindowApp/`.

### Структура UI-префаба (CounterRoot)

```
CounterRoot [LuaBehaviour, luaScript=counter.lua]
 ├─ Value           [TextMeshPro]   ← инъекция "valueText"
 ├─ PlusBtn  [Button + LuaUIButton] Function="onChange"  Argument="1"
 ├─ MinusBtn [Button + LuaUIButton] Function="onChange"  Argument="-1"
 ├─ ResetBtn [Button + LuaUIButton] Function="onReset"
 └─ AutoBtn  [Button + LuaUIButton] Function="onToggleAuto"
      └─ AutoLabel [TextMeshPro]    ← инъекция "autoText"
```

**`counter.lua`**
```lua
-- Инъекции = public поля: valueText и autoText — это GameObject'ы.
-- Кнопки: LuaUIButton → вызывает Lua-функции (onChange, onReset, onToggleAuto).

local count   = 0
local autoOn  = false
local timerId = nil

-- Кэш TMP-компонентов (как GetComponent<T> в Start())
local valueTMP
local autoTMP

local function render()
    valueTMP.text = tostring(count)
    autoTMP.text = autoOn and "Auto: ON" or "Auto: OFF"
end

function start()  -- аналог Start() в C#
    -- GetComponent — как в C#
    valueTMP = valueText:GetComponent("TextMeshProUGUI")
    autoTMP  = autoText:GetComponent("TextMeshProUGUI")

    count, autoOn = 0, false
    render()
end

function onChange(arg)
    count = count + (tonumber(arg) or 0)
    render()
end

function onReset()
    count = 0; render()
end

function onToggleAuto()
    autoOn = not autoOn
    if autoOn then
        timerId = EMRAPI.Timer:Repeat(1.0, function()
            count = count + 1; render()
        end)
    elseif timerId then
        EMRAPI.Timer:Cancel(timerId); timerId = nil
    end
    render()
end

function onDestroy()  -- аналог OnDestroy() в C#
    if timerId then EMRAPI.Timer:Cancel(timerId) end
end
```

**Как запустить в Play-Mode:**
1. Добавь на сцену пустой GameObject → Add Component → `EtmrSimulatorHost`
2. `Window Mode = ✓`, `Preview Entry Prefab = CounterRoot.prefab`
3. Play → счётчик открывается прямо в редакторе

---

## 7. Unity SDK — работа с GetComponent (как в C#)

Пример показывает прямой доступ к Unity-компонентам из Lua в LuaBehaviour.

```lua
-- ИНЪЕКЦИИ: healthBar (GameObject с Image), label (GameObject с TMP)
-- self = this (LuaBehaviour)

local img
local tmp
local hp = 100

function start()
    -- GetComponent — как в C#
    img = healthBar:GetComponent("Image")
    tmp = label:GetComponent("TextMeshProUGUI")

    -- Прямой доступ к свойствам, как в C#
    img.fillAmount = 1.0
    tmp.text = "HP: 100"
    tmp.fontSize = 24
end

function takeDamage(amount)
    hp = hp - (tonumber(amount) or 10)
    if hp < 0 then hp = 0 end

    img.fillAmount = hp / 100
    tmp.text = "HP: " .. hp

    -- Цвет меняется от зелёного к красному
    local t = hp / 100
    img.color = CS.UnityEngine.Color(1 - t, t, 0, 1)
end
```

---

## 8. HTTP-запросы — Чистый Lua

**`app.json`** — `permissions: ["ui", "network"]`  
⚠️ Пользователю будет показан диалог подтверждения.

```lua
local panel = EMRAPI.UI:CreatePanel(1.0, 0.5)
local label = EMRAPI.UI:CreateText(panel, "Загрузка...")
EMRAPI.UI:SetPosition(label, 0, 0, 0)
EMRAPI.UI:SetSize(label, 400, 100)

function OnStart()
    EMRAPI.Network:Get("https://api.ipify.org?format=json", function(response, error)
        if error then
            EMRAPI.UI:SetText(label, "Ошибка: " .. error)
        else
            EMRAPI.UI:SetText(label, "Ваш IP: " .. response)
        end
    end)
end
```

---

## 9. Вращающийся куб — смешанный режим (Scene + UI)

Куб в 3D + кнопка для смены цвета. Демонстрирует совместное использование `EMRAPI.Scene` и `EMRAPI.UI`.

```lua
local cube
local t = 0

function OnStart()
    cube = EMRAPI.Scene:CreatePrimitive("cube")
    EMRAPI.Scene:SetPosition(cube, 0, 1, 3)
    EMRAPI.Scene:SetColor(cube, 0.2, 0.8, 0.4, 1)

    EMRAPI.UI:CreateButton(nil, "Random Color", function()
        EMRAPI.Scene:SetColor(cube,
            math.random(), math.random(), math.random(), 1)
    end)
end

function OnUpdate(dt)
    t = t + dt
    if cube then
        EMRAPI.Scene:SetRotation(cube, 0, t * 30, 0)
    end
end

function OnDestroy()
    EMRAPI:Log("Cube demo closed")
end
```

---

## Частые ошибки

| Ошибка | Причина | Решение |
|---|---|---|
| `attempt to call a nil value` | `.` вместо `:` при вызове EMRAPI, или не сгенерированы обёртки xLua | Проверь синтаксис (`EMRAPI.Timer:Repeat`, не `EMRAPI.Timer.Repeat`). Запусти XLua → Generate Code. |
| `GetComponent` возвращает nil | Неверное имя типа (регистр!) или компонента нет на объекте | `"TextMeshProUGUI"`, не `"textmeshprougui"`. Проверь объект в инспекторе. |
| Свойство недоступно из Lua | Тип не добавлен в `EMRAPIXLuaConfig` | Добавь тип → XLua → Generate Code |
| Текст не меняется | Инъекция указывает не на тот объект | Проверь Injections в инспекторе |
| `Invalid manifest` | Невалидный `app.json` (нет `id`, `name`, или нет `entry`/`entryPrefab`) | Проверь все обязательные поля |
| `LuaBehaviour: Хост недоступен` | Нет `EtmrSimulatorHost` на сцене (Play-Mode тест) | Добавь компонент на любой объект |
| Краш на iOS / IL2CPP | XLua → Generate Code не запускали | Обязательно запусти перед билдом |
| `OnStart` не вызывается в LuaBehaviour | Используется неправильный регистр | LuaBehaviour: `start()`. Чистый Lua: `OnStart()`. |
| App requires API vN | `minApiVersion` выше текущей платформы | Понизь `minApiVersion` в `app.json` или обнови платформу |
