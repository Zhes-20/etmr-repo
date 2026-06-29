# EMRAPI Reference Manual

Глобальный объект `EMRAPI` доступен в любом Lua-скрипте (и в чистом Lua, и в `LuaBehaviour`).

> **Когда использовать EMRAPI vs прямой Unity API:**
> - **Чистый Lua-режим:** используй `EMRAPI.UI`, `EMRAPI.Scene`, `EMRAPI.Input` — это твой единственный способ создавать объекты и UI.
> - **Unity SDK (LuaBehaviour):** можешь использовать и `EMRAPI`, и прямой доступ к Unity-компонентам через `GetComponent`. EMRAPI — удобные обёртки, прямой доступ — полный контроль.

---

## Базовые методы (всегда доступны)

```lua
EMRAPI:Log("msg")         -- print в Unity Console
EMRAPI:LogWarning("msg")  -- жёлтый warning
EMRAPI:LogError("msg")    -- красный error
EMRAPI.App:RequestExit()  -- закрыть приложение
```

---

## EMRAPI.App — мета-информация

```lua
EMRAPI.App.id               -- string (id из app.json)
EMRAPI.App.displayMode      -- "window" или "scene"
EMRAPI.App:GetName()        -- string
EMRAPI.App:GetVersion()     -- string
EMRAPI.App:GetAuthor()      -- string
EMRAPI.App:GetDescription() -- string
EMRAPI.App:RequestExit()    -- корректное закрытие приложения
```

---

## EMRAPI.System — системные данные

```lua
EMRAPI.System:GetTime()        -- float, секунды с запуска приложения
EMRAPI.System:GetDeltaTime()   -- float, время между кадрами
EMRAPI.System:GetPlatform()    -- "ios" | "android" | "editor"
```

---

## EMRAPI.Math — математика

```lua
EMRAPI.Math:Distance(x1,y1,z1, x2,y2,z2) -- float
EMRAPI.Math:Lerp(a, b, t)                 -- float (линейная интерполяция)
EMRAPI.Math:Random(min, max)              -- float (случайное число)
EMRAPI.Math:Clamp(val, min, max)          -- float (ограничение диапазона)
```

---

## EMRAPI.Timer — таймеры

Единственный способ создать «петлю Update» или отложенный вызов в чистом Lua без блокировки Unity.

```lua
-- Повторяющийся таймер (каждые ~16 мс ≈ 60 fps)
local id = EMRAPI.Timer:Repeat(0.016, function()
    -- каждый кадр
end)

-- Одноразовый таймер (через 2 секунды)
local id2 = EMRAPI.Timer:Delay(2.0, function()
    -- один раз через 2 секунды
end)

-- Остановить таймер
EMRAPI.Timer:Cancel(id)
```

> ⚠️ **Обязательно отменяйте таймеры в `OnDestroy` / `onDestroy`!** Иначе — утечки памяти и ошибки после закрытия приложения.

---

## EMRAPI.UI — 2D-интерфейс
**Право:** `ui`
**Возвращает:** `GameObject` — Unity-объект элемента.

> **В Unity-режиме (SDK):** UI создаётся мышкой в редакторе + `LuaBehaviour`.
> `EMRAPI.UI` нужен в чистом Lua-режиме или для динамического создания элементов.

### Создание

```lua
-- Полупрозрачная панель (ширина и высота в метрах мирового пространства)
local panel = EMRAPI.UI:CreatePanel(1.0, 1.0)

-- Кнопка с коллбэком
local btn = EMRAPI.UI:CreateButton(panel, "Нажми", function()
    EMRAPI:Log("нажато!")
end)

-- Текст
local label = EMRAPI.UI:CreateText(panel, "Привет!")

-- Изображение из архива (PNG/JPG, относительный путь внутри .emr)
local img = EMRAPI.UI:CreateImage(panel, "images/icon.png")

-- Поле ввода
local field = EMRAPI.UI:CreateInputField(panel, "введи текст...", function(val)
    EMRAPI:Log("введено: " .. val)
end)

-- Слайдер (min, max, callback)
local slider = EMRAPI.UI:CreateSlider(panel, 0.0, 100.0, function(val)
    EMRAPI:Log("слайдер: " .. val)
end)

-- Чекбокс / Toggle
local toggle = EMRAPI.UI:CreateToggle(panel, "Включить", function(state)
    EMRAPI:Log("toggle: " .. tostring(state))
end)

-- Скроллируемая область (ширина, высота — возвращает content-контейнер)
local scroll = EMRAPI.UI:CreateScrollView(panel, 1.0, 0.5)
```

### Манипуляция

```lua
EMRAPI.UI:SetColor(el, r, g, b, a)     -- 0.0–1.0
EMRAPI.UI:SetPosition(el, x, y, z)     -- позиция (в пикселях UI)
EMRAPI.UI:SetSize(el, w, h)            -- размер (в пикселях UI)
EMRAPI.UI:SetText(el, "текст")         -- изменить текст
local t = EMRAPI.UI:GetText(el)        -- прочитать текст
EMRAPI.UI:SetActive(el, true/false)    -- показать/скрыть
EMRAPI.UI:Destroy(el)                  -- удалить элемент
```

---

## EMRAPI.Scene — 3D-объекты и физика
**Право:** `scene`

### Создание объектов

```lua
-- Примитивы: "cube", "sphere", "capsule", "cylinder", "plane", "quad"
local cube   = EMRAPI.Scene:CreatePrimitive("cube")
local sphere = EMRAPI.Scene:CreatePrimitive("sphere")
local empty  = EMRAPI.Scene:CreateObject("MyObj")   -- пустой GameObject

local root = EMRAPI.Scene:GetRoot()                  -- корень сцены приложения
```

### Трансформации

```lua
EMRAPI.Scene:SetPosition(obj, x, y, z)
EMRAPI.Scene:SetRotation(obj, x, y, z)   -- углы Эйлера
EMRAPI.Scene:SetScale(obj, x, y, z)
EMRAPI.Scene:SetColor(obj, r, g, b, a)   -- работает с MeshRenderer
EMRAPI.Scene:SetParent(child, parent)

local x = EMRAPI.Scene:GetPositionX(obj)
local y = EMRAPI.Scene:GetPositionY(obj)
local z = EMRAPI.Scene:GetPositionZ(obj)

local found = EMRAPI.Scene:Find("ИмяОбъекта")  -- поиск по имени под корнем
EMRAPI.Scene:Destroy(obj)
```

### Физика

```lua
local rb  = EMRAPI.Scene:AddRigidbody(obj, mass, useGravity) -- возвращает Rigidbody
local col = EMRAPI.Scene:AddBoxCollider(obj)                  -- возвращает BoxCollider
local col = EMRAPI.Scene:AddSphereCollider(obj, radius)       -- возвращает SphereCollider (работает)
```

> [!NOTE]
> **Обработка столкновений (коллизий):**
> На данный момент встроенных событий (вроде `OnCollisionEnter` или `OnTriggerEnter`) в рантайме EMRAPI нет.
> - В **Pure Lua** режиме коллизии проверяются вручную в `OnUpdate(dt)` (например, через проверку расстояния `EMRAPI.Math:Distance(...)`).
> - В **Unity SDK (LuaBehaviour)** режиме можно повесить C# скрипт-посредник на объект, который перехватит событие `OnTriggerEnter` от Unity и вызовет Lua-метод: `targetBehaviour:Call("onTriggerEnter", other)`.


### Спаун готовых префабов (Unity-режим)

```lua
-- Клонировать Unity-префаб (GameObject из бандла)
local clone  = EMRAPI.Scene:Spawn(prefabGO)
local clone2 = EMRAPI.Scene:SpawnAt(prefabGO, 0, 1, 3)
```

### Привязка объектов к рукам

```lua
-- Немедленная привязка (жёсткая, объект становится дочерним)
EMRAPI.Scene:AttachToHand(obj, "left",  "IndexTip")   -- по имени точки
EMRAPI.Scene:AttachToHand(obj, "right", "4")           -- по индексу лендмарка (0-20)

-- Плавная привязка (компонент-следователь, сглаживание)
EMRAPI.Scene:AttachToHandSmooth(obj, "right", "PalmCenter", 10.0, 8.0)
-- positionLerp, rotationLerp — скорость следования

EMRAPI.Scene:DetachFromHand(obj)  -- отвязать
```

**Имена точек привязки (TrackedHandAnchorPoint):**
`PalmCenter`, `IndexTip`, `ThumbTip`, `MiddleTip`, `RingTip`, `PinkyTip`,
`IndexKnuckle`, `MiddleKnuckle`, `RingKnuckle`, `PinkyKnuckle`, `Wrist`

Или числовой индекс лендмарка: `"0"` – `"20"` (MediaPipe Hand Landmarks).

---

## EMRAPI.Input — VR/MR-трекинг
**Право:** `input`

```lua
-- Проверка отслеживания рук ("left" или "right")
local leftTracked = EMRAPI.Input:IsHandTracked("left")
local rightTracked = EMRAPI.Input:IsHandTracked("right")

-- Получение позиции руки (возвращает три float: x, y, z)
-- Второй параметр — имя точки привязки (например, "PalmCenter") или индекс 0-20
local lx, ly, lz = EMRAPI.Input:GetHandPosition("left", "PalmCenter")
local rx, ry, rz = EMRAPI.Input:GetHandPosition("right", "PalmCenter")

-- Получение поворота руки (возвращает четыре float: x, y, z, w)
local lx, ly, lz, lw = EMRAPI.Input:GetHandRotation("left", "PalmCenter")

-- Упрощенные методы получения координат PalmCenter для рук
local palmX = EMRAPI.Input:GetHandPositionX("left")
local palmY = EMRAPI.Input:GetHandPositionY("left")
local palmZ = EMRAPI.Input:GetHandPositionZ("left")

-- Жесты (возвращает строку: "Pinch", "Fist", "Five", "OK", "Point", "Like" или др. из Vive SDK)
local leftGesture = EMRAPI.Input:GetHandGesture("left")

-- Удобные хелперы для жестов
local isPinching = EMRAPI.Input:GetPinchStrength("left") > 0.5
local isGrabbing = EMRAPI.Input:IsGrabbing("left")

-- Трекинг головы (основной камеры)
local headX = EMRAPI.Input:GetHeadPositionX()
local headY = EMRAPI.Input:GetHeadPositionY()
local headZ = EMRAPI.Input:GetHeadPositionZ()

local headRotX = EMRAPI.Input:GetHeadRotationX()
local headRotY = EMRAPI.Input:GetHeadRotationY()
local headRotZ = EMRAPI.Input:GetHeadRotationZ()
local headRotW = EMRAPI.Input:GetHeadRotationW()
```

---

## EMRAPI.Audio — звук
**Право:** `audio`

Пути — относительно корня архива приложения (.emr).

```lua
EMRAPI.Audio:Play("sounds/click.wav")
EMRAPI.Audio:PlayAtPoint("sounds/boom.wav", 0, 1, 3)  -- позиционный 3D-звук
EMRAPI.Audio:Stop()                    -- остановить всё аудио приложения
EMRAPI.Audio:SetVolume(0.8)            -- 0.0–1.0
```

---

## EMRAPI.Storage — хранилище
**Право:** `storage`

Данные изолированы по `id` приложения. Используется `PlayerPrefs` под капотом.

```lua
EMRAPI.Storage:Set("key", "value")                    -- строка
EMRAPI.Storage:GetString("key", "default") --> string

EMRAPI.Storage:SetNumber("score", 42.0)
EMRAPI.Storage:GetNumber("score", 0.0)    --> float

EMRAPI.Storage:SetBool("enabled", true)
EMRAPI.Storage:GetBool("enabled", false)  --> bool

EMRAPI.Storage:Remove("key")
EMRAPI.Storage:Save()  -- ОБЯЗАТЕЛЬНО вызвать для записи на диск
```

---

## EMRAPI.Network — HTTP
**Право:** `network` *(опасное — показывает диалог пользователю)*

```lua
EMRAPI.Network:Get("https://api.example.com/data", function(response, error)
    if error then
        EMRAPI:LogError("Ошибка: " .. error)
    else
        EMRAPI:Log("Ответ: " .. response)
    end
end)

EMRAPI.Network:Post("https://api.example.com/submit", '{"key":"value"}',
    function(response, error)
        -- ...
    end
)
```

---

## EMRAPI.Bundle — работа с AssetBundle
**Только Unity-режим (`type: "bundle"`)**

```lua
EMRAPI.Bundle:Load()                   -- загрузить бандл (обычно автоматически)
EMRAPI.Bundle:IsLoaded()               -- bool
local go = EMRAPI.Bundle:LoadAsset("PrefabName")   -- загрузить как ассет
local go = EMRAPI.Bundle:Instantiate("PrefabName")  -- загрузить + создать экземпляр
```

---

## Соглашения о вызове (важно!)

1. **Двоеточие (`:`) vs точка (`.`)**: методы объектов EMRAPI всегда через `:`. Если написать через `.` и не передать `self` — будет `attempt to call nil`.
   ```lua
   -- Правильно:
   EMRAPI.Timer:Repeat(1.0, fn)
   EMRAPI.UI:SetText(el, "ok")
   EMRAPI:Log("msg")

   -- Неправильно:
   EMRAPI.Timer.Repeat(1.0, fn)   -- ОШИБКА
   ```

2. **`Vector3` / `Vector2`** — объекты C#. Доступны как `vec.x`, `vec.y`, `vec.z`. Возвращаются из `EMRAPI.Input:GetHeadPosition()` и подобных.

3. **Коллбэки из таймеров/кнопок** выполняются в главном Unity-потоке — Unity API из них вызывать безопасно.

4. **`require("module")`** — загружает скрипт из папки `scripts/` внутри архива. Расширение `.lua` добавляется автоматически.

---

## Доступ к Unity API из LuaBehaviour (xLua)

В Unity SDK (LuaBehaviour) кроме EMRAPI можно напрямую обращаться к Unity-компонентам:

```lua
-- Получить компонент по имени типа (строка)
local rb = myObj:GetComponent("Rigidbody")
rb.mass = 5.0
rb.useGravity = true

-- TMP-текст
local tmp = labelObj:GetComponent("TextMeshProUGUI")
tmp.text = "Hello"
tmp.fontSize = 24

-- Image (UI)
local img = healthBarObj:GetComponent("Image")
img.fillAmount = 0.75

-- Создание Unity-типов через CS.UnityEngine
local pos = CS.UnityEngine.Vector3(1, 2, 3)
local red = CS.UnityEngine.Color(1, 0, 0, 1)
```

**Полный список доступных типов** — см. раздел 8 в `EMR_Architecture_Guide.md`.
