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
EMRAPI.System:GetTime()          -- float, секунды с запуска приложения
EMRAPI.System:GetDeltaTime()     -- float, время между кадрами
EMRAPI.System:GetPlatform()      -- "ios" | "android" | "editor"
EMRAPI.System:GetTrackingMode()  -- "3dof" | "6dof"
```

> **GetTrackingMode:** `"6dof"` — позиция головы отслеживается (ARKit/ARCore), объекты можно расставлять по комнате. `"3dof"` — только поворот головы, контент держи перед пользователем.

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
-- Заменить картинку потом (в т.ч. на скачанную):
-- keepSize=false подгонит размер под пропорции новой картинки
EMRAPI.UI:SetImage(img, "data:cat.jpg", false)

-- Поле ввода (при фокусе автоматически появляется системная клавиатура
-- и печатает в поле; ручное управление — EMRAPI.Keyboard)
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

### Привязка объектов к голове (HUD)

```lua
-- Объект остаётся там, где стоит сейчас, и дальше едет вместе с головой
EMRAPI.Scene:AttachToHead(obj)

-- Явное смещение в локальных координатах камеры:
-- x — вправо, y — вверх, z — вперёд (в метрах)
EMRAPI.Scene:AttachToHeadAt(obj, 0.15, -0.1, 0.6)

EMRAPI.Scene:DetachFromHead(obj)  -- отвязать (остаётся где был)
```

### Текстуры и 3D-модели

```lua
-- Натянуть картинку на объект (лучше всего на "quad" или "plane").
-- Путь: из пакета ("images/cat.jpg") или скачанный ("data:cat.jpg").
local screen = EMRAPI.Scene:CreatePrimitive("quad")
EMRAPI.Scene:SetTexture(screen, "images/poster.jpg")   -- вернёт true/false

-- Загрузить 3D-модель GLB (асинхронно). Модель попадает в корень приложения.
EMRAPI.Scene:LoadModel("models/car.glb", function(obj, error)
    if error then
        EMRAPI:LogError("модель не загрузилась: " .. error)
    else
        EMRAPI.Scene:SetPosition(obj, 0, 0, 2)
        EMRAPI.Scene:SetScale(obj, 0.5, 0.5, 0.5)
    end
end)
```

> `LoadModel` понимает **GLB** (один файл со всем внутри). `.gltf` с отдельными
> текстурами не соберётся — конвертируй в `.glb`. Скачанные модели: сначала
> `Network:DownloadFile`, потом `LoadModel("data:car.glb", ...)`.

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
`Play`/`PlayAtPoint`/`PlayLoop` возвращают **id источника** — для одноразовых звуков его можно игнорировать.

```lua
-- Одноразовые звуки (id можно не сохранять)
EMRAPI.Audio:Play("sounds/click.wav")
EMRAPI.Audio:PlayAtPoint("sounds/boom.wav", 0, 1, 3)  -- позиционный 3D-звук

-- Фоновая музыка: зацикленный источник + управление по id
local music = EMRAPI.Audio:PlayLoop("sounds/bg.ogg")
EMRAPI.Audio:SetVolume(music, 0.4)   -- громкость этого источника
EMRAPI.Audio:SetPitch(music, 1.2)    -- скорость/тон (0.1–3.0)
EMRAPI.Audio:SetLoop(music, false)   -- доиграет и остановится
EMRAPI.Audio:IsPlaying(music)        -- bool
EMRAPI.Audio:Stop(music)             -- остановить этот источник

-- Глобальное управление
EMRAPI.Audio:Stop()                  -- остановить всё аудио приложения
EMRAPI.Audio:SetVolume(0.8)          -- базовая громкость всех источников (0.0–1.0)
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

### Скачивание файлов (картинки, звуки, 3D-модели)

```lua
-- Скачать в записываемую папку приложения.
-- В callback приходит путь вида "data:cat.jpg" — его понимают
-- UI:CreateImage / UI:SetImage / Scene:SetTexture / Scene:LoadModel / Audio:Play
EMRAPI.Network:DownloadFile("https://example.com/cat.jpg", "cat.jpg",
    function(path, error)
        if error then
            EMRAPI:LogError("не скачалось: " .. error)
        else
            local img = EMRAPI.UI:CreateImage(panel, path)   -- path == "data:cat.jpg"
        end
    end)

-- То же с прогрессом (0.0–1.0) — удобно показать полоску загрузки
EMRAPI.Network:DownloadFileWithProgress("https://example.com/model.glb", "model.glb",
    function(path, error) ... end,
    function(p) EMRAPI.UI:SetText(label, math.floor(p * 100) .. "%") end)
```

> Файлы качаются в отдельную папку приложения (пакет `.emr` — read-only) и переживают
> перезапуск. Потолок — 40 МБ на файл. Имя чистится от путей: только имя файла.

---

## EMRAPI.Multiplayer — онлайн-игры (Photon)
**Право:** `multiplayer` (опасное — пользователь подтверждает диалогом)

Онлайн через Photon Cloud. У каждой игры — СВОЙ Photon AppId (это бесплатно и
занимает пару минут; без него мультиплеер не подключится).

**Как получить AppId (инструкция для пользователя — объясняй по шагам):**
1. Зайти на **dashboard.photonengine.com** (регистрация бесплатная, почта+пароль).
2. Кнопка **Create a New App**.
3. Photon Type: выбрать **Realtime** (НЕ Fusion, НЕ PUN — именно Realtime; для
   старых дашбордов подходит и тип PUN).
4. Name — любое имя игры → Create.
5. На карточке приложения появится **App ID** (строка вида
   `a1b2c3d4-e5f6-...`) — скопировать её.
6. Вставить в `app.json` игры:

```json
{ "photonAppId": "a1b2c3d4-e5f6-...", "permissions": ["multiplayer"] }
```

Бесплатный тариф: **20 одновременных игроков (CCU)** на приложение — для мини-игр
хватает с запасом. Без `photonAppId` в манифесте можно передать id кодом:
`ConnectWithAppId(id, cb)` (удобно для быстрых тестов).

Регион: по умолчанию все игроки подключаются к **eu**. Другой — поле
`"photonRegion": "us"` в манифесте (`eu`,`us`,`asia`,`jp`,`sa`,`kr`,`in`).
Игроки в разных регионах НЕ видят друг друга — регион фиксируется, чтобы комната
с одним именем была одной комнатой у всех.

### Подключение и комнаты

```lua
local MP = EMRAPI.Multiplayer   -- nil, если права нет!

MP:Connect(function(ok, err) ... end)     -- AppId из манифеста
MP:Disconnect()
MP:IsConnected()                          -- true после Connect
MP:GetState()                             -- offline|connecting|connected|joining|inroom
MP:OnDisconnected(function(reason) ... end)

MP:SetNickname("Вася")
MP:CreateRoom("room1", 4, cb)             -- cb(ok, err)
MP:JoinRoom("room1", cb)
MP:JoinOrCreateRoom("room1", 4, cb)
MP:JoinRandomOrCreateRoom(4, cb)          -- самый простой старт
MP:LeaveRoom(cb)
MP:GetRoomList(function(json, err) ... end) -- [{name,players,maxPlayers}]; только ВНЕ комнаты
MP:IsInRoom(); MP:GetRoomName(); MP:GetPlayerCount(); MP:IsMaster()
```

### Игроки

```lua
local players = EMRAPI.Json:Decode(MP:GetPlayers()) -- {actor,nick,isMaster,isLocal}
MP:GetLocalActor()                        -- свой номер (или -1)
MP:OnPlayerJoined(function(actor, nick) ... end)
MP:OnPlayerLeft(function(actor, nick) ... end)
MP:OnMasterChanged(function(actor) ... end)
```

### События — главный инструмент геймплея

```lua
MP:SendEvent("shoot", EMRAPI.Json:Encode({x=1, y=2}))   -- всем кроме себя
MP:SendEventToAll("score", json)                        -- включая себя (локальное эхо)
MP:SendEventTo(actor, "secret", json)                   -- одному игроку
MP:OnEvent("shoot", function(senderActor, json)
    local data = EMRAPI.Json:Decode(json)
end)
MP:OffEvent("shoot")
```

### Сетевые объекты (чистый Lua, без префабов)

```lua
-- Спавн виден всем в комнате (и тем, кто зайдёт позже). Позиция в координатах сцены.
local ball = MP:SpawnSynced("sphere", 0, 1, 2, 0.12)  -- primitive, x,y,z, scale
MP:IsMine(ball)                 -- владелец = у кого физика
MP:TakeOwnership(ball)          -- поймал чужой мяч — забери владение, кидай
MP:DestroySynced(ball)
MP:OnSyncedSpawned(function(netId, ownerActor, primitive) ... end) -- чужой спавн
MP:SetSyncRate(10)              -- Гц синка трансформов (1..20)
```

> Физика (Rigidbody) работает только у владельца, остальные видят плавную
> интерполяцию. Бросок мяча: спавнишь → AddRigidbody → velocity. Ловля чужого:
> TakeOwnership → физика теперь у тебя.

### Свойства (персистентное состояние комнаты/игроков)

```lua
MP:SetRoomProperty("map", "arena2");   MP:GetRoomProperty("map")
MP:SetPlayerProperty("score", "15");   MP:GetPlayerProperty(actor, "score")
MP:OnRoomPropertyChanged(function(key, value) ... end)
MP:OnPlayerPropertyChanged(function(actor, key, value) ... end)
```

### Аватары игроков — два пути

**Путь 1 (по умолчанию): встроенные.** Ничего не делаешь — у всех в комнате видны
аватары друг друга: голова с направлением взгляда, все 21 сустав каждой кисти,
ник над головой, у каждого игрока свой цвет. Игроки автоматически разнесены по
кругу (~1.2 м), чтобы не спавниться друг в друге. Синк и отрисовку делает оболочка.

**Путь 2: своя система.** Хочешь кастомных аватаров (3D-модели, частицы, свой вид) —
выключи встроенные и собери свои из уже имеющихся кирпичей:

```lua
MP:SetAvatarsEnabled(false)          -- встроенные выключены

-- Отправка своей позы (10 Гц достаточно):
EMRAPI.Timer:Repeat(0.1, function()
    if not MP:IsInRoom() then return end
    local hx = EMRAPI.Input:GetHeadPositionX()
    local hy = EMRAPI.Input:GetHeadPositionY()
    local hz = EMRAPI.Input:GetHeadPositionZ()
    -- руки: любой из 21 сустава ("0".."20") или имена ("PalmCenter", "IndexTip"...)
    local rx, ry, rz = EMRAPI.Input:GetHandPosition("right", "PalmCenter")
    MP:SendEvent("pose", EMRAPI.Json:Encode({ h = {hx,hy,hz}, r = {rx,ry,rz} }))
end)

-- Приём и отрисовка чем угодно (примитивы, GLB-модель через Scene:LoadModel):
local avatars = {}
MP:OnEvent("pose", function(actor, json)
    local d = EMRAPI.Json:Decode(json)
    if d == nil then return end
    if avatars[actor] == nil then
        avatars[actor] = EMRAPI.Scene:CreatePrimitive("sphere")
        EMRAPI.Scene:SetScale(avatars[actor], 0.2, 0.2, 0.2)
    end
    EMRAPI.Scene:SetPosition(avatars[actor], d.h[1], d.h[2], d.h[3])
end)
MP:OnPlayerLeft(function(actor)
    if avatars[actor] then EMRAPI.Scene:Destroy(avatars[actor]); avatars[actor] = nil end
end)
```

> В кастомной системе разнос игроков (чтобы не спавнились в одной точке) делаешь
> сам: прибавь к отправляемым координатам смещение от своего номера
> (`MP:GetLocalActor()`), например по кругу.

### Unity-режим (SDK): сетевые префабы

```lua
-- Префаб с PhotonView из бандла (собирается в SDK-проекте с тем же PUN2):
MP:RegisterPrefab("NetBall", EMRAPI.Bundle:LoadAsset("NetBall"))
local ball = MP:Instantiate("NetBall", 0, 1, 2)  -- PhotonNetwork.Instantiate
```

### Голосовой чат (Photon Voice 2)

Голос встроен в `EMRAPI.Multiplayer` и работает через отдельный Voice-клиент Photon
(Opus-кодек, ~24 кГц). Отдельный Photon App **типа `Voice`**:

```json
{
  "photonAppId": "a1b2c3d4-...",
  "photonVoiceAppId": "v1-5e6f-...",
  "photonRegion": "eu",
  "permissions": ["multiplayer"]
}
```

**Как получить Voice AppId:** в дашборде Photon Engine создаётся **второе**
приложение (Create a New App → Photon Type: **Voice**) — это отдельный App ID от
Realtime. Голос ходит в отдельную комнату `<комната>_voice_` с **тем же регионом**,
что и Realtime (иначе игроки не услышат друг друга).

```lua
local MP = EMRAPI.Multiplayer

-- Включить микрофон (спросит разрешение на микрофон) — ТОЛЬКО внутри комнаты:
local res = MP:SetVoiceEnabled(true)
-- res — строка: "голос включён" / "голос уже включён" /
--              "голос недоступен: ... (причина)" — показывай её при необходимости

MP:IsVoiceEnabled()              -- bool — включён ли голос
MP:SetVoiceEnabled(false)        -- выключить (глушит микрофон)

-- VAD (подавление тишины, по умолчанию вкл — меньше трафика и фонового шума):
MP:SetVoiceVad(false)            -- отключить VAD
MP:SetVoiceVad(true)             -- включить обратно

-- Заглушить/разговорить конкретного игрока (по actor из GetPlayers()):
MP:MutePlayer(3, true)           -- не слышать игрока #3
MP:IsPlayerMuted(3)              -- bool
MP:MutePlayer(3, false)

-- Индикация «кто говорит» (событие при СМЕНЕ состояния, не тикающее):
MP:OnPlayerSpeaking(function(actor, isSpeaking)
    if isSpeaking then
        EMRAPI:Log("игрок #" .. actor .. " говорит")
        -- подсвети его аватар / покажи иконку микрофона
    else
        EMRAPI:Log("игрок #" .. actor .. " молчит")
    end
end)
```

**Особенности:**
- Голос автоматически выключается при выходе из комнаты / закрытии приложения —
  чистить руками не нужно.
- `SetVoiceEnabled(true)` до входа в комнату вернёт строку с причиной — вызови
  после `JoinOrCreateRoom`.
- Мут — только на твоём устройстве (другие игроки не узнают, что ты их заглушил).

### Ограничения

- `GetRoomList` работает только вне комнаты (ограничение Photon-лобби).
- Свернул приложение >60 сек → сервер отключит; лови `OnDisconnected` и предлагай
  `Connect` заново (авто-реконнекта нет).

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

## EMRAPI.Hands — вид скелета рук
**Право:** не требуется. Все изменения автоматически откатываются при закрытии приложения.

`hand` — `"left"`, `"right"` или `"both"`. Цвета — float `0.0–1.0`.

```lua
EMRAPI.Hands:SetColor("both", 1, 0, 0)       -- весь скелет красный
EMRAPI.Hands:SetPointColor("left", 1, 1, 0)  -- только точки (суставы)
EMRAPI.Hands:SetLinkColor("left", 0, 1, 1)   -- только связи (кости)
EMRAPI.Hands:SetAlpha("both", 0.3)           -- полупрозрачный скелет
EMRAPI.Hands:SetVisible("right", false)      -- скрыть правую руку
local vis = EMRAPI.Hands:IsVisible("left")   -- bool
EMRAPI.Hands:Reset("both")                   -- вернуть вид по умолчанию
```

> Скрытие руки прячет только визуализацию — трекинг и `EMRAPI.Input` продолжают работать.

---

## EMRAPI.Camera — фон с камеры (passthrough)
**Право:** не требуется. Откатывается при закрытии приложения.

```lua
EMRAPI.Camera:SetBackground("hide")   -- "show" | "hide" | "blur"
local mode = EMRAPI.Camera:GetBackground()  -- текущий режим строкой
EMRAPI.Camera:Reset()                 -- вернуть, как было до приложения
```

Полезно для иммерсивных scene-приложений: `"hide"` — полное VR-погружение, `"blur"` — фокус на контенте без потери ориентации.

---

## EMRAPI.Json — JSON
**Право:** не требуется.

В Lua нет встроенного `json`/`cjson` — используй этот модуль, особенно с `EMRAPI.Network`.

```lua
-- Таблица -> строка
local s = EMRAPI.Json:Encode({ name = "Player", score = 100, tags = {"a", "b"} })
-- '{"name":"Player","score":100,"tags":["a","b"]}'

-- Строка -> таблица
local t = EMRAPI.Json:Decode('{"items":[1,2,3],"ok":true}')
if t then
    EMRAPI:Log("первый: " .. t.items[1])
end
```

Правила:
- Таблица с целыми ключами `1..n` без пропусков → JSON-массив, иначе → объект.
- Ошибка (битый JSON, цикл в таблице) → возвращается `nil`, причина в логе. **Всегда проверяй результат на `nil`.**
- Целые числа декодируются как Lua integer, дробные — как float.

Типичная связка с сетью:

```lua
EMRAPI.Network:Get("https://api.example.com/user", function(resp, err)
    if err then return end
    local data = EMRAPI.Json:Decode(resp)
    if data then EMRAPI.UI:SetText(label, data.name) end
end)
```

---

## EMRAPI.Tween — плавные анимации
**Право:** не требуется.

Анимация трансформа без ручной лерпы в `OnUpdate`. Локальные координаты (та же система, что у `Scene:SetPosition`), сглаживание smoothstep.

```lua
-- Базовые твины (возвращают id)
local id = EMRAPI.Tween:MoveTo(obj, 0, 1, 2, 0.5)     -- к точке за 0.5 сек
EMRAPI.Tween:RotateTo(obj, 0, 180, 0, 1.0)             -- углы Эйлера
EMRAPI.Tween:ScaleTo(obj, 2, 2, 2, 0.3)

-- С колбэком по завершении (последним аргументом)
EMRAPI.Tween:MoveTo(obj, 0, 0, 1, 0.5, function()
    EMRAPI:Log("приехали")
end)

-- Управление
EMRAPI.Tween:Cancel(id)        -- остановить (объект замирает где был)
EMRAPI.Tween:CancelAll()
local run = EMRAPI.Tween:IsRunning(id)  -- bool
```

Правила:
- Новый твин **того же типа** на том же объекте заменяет старый (MoveTo поверх MoveTo). Разные типы (MoveTo + ScaleTo) работают параллельно.
- `duration <= 0` — применяется мгновенно, id не выдаётся (`-1`).
- Перед перепарентом объекта (например, `AttachToHead`) отмени его твины — они работают в локальных координатах и «подерутся» с новым родителем.

---

## EMRAPI.Keyboard — виртуальная клавиатура
**Право:** не требуется. Все изменения откатываются при закрытии приложения.

> **Для обычного ввода этот раздел не нужен.** Тап по полю из
> `EMRAPI.UI:CreateInputField` фокусирует его — клавиатура появляется сама
> и печатает прямо в поле. `Keyboard` нужен для ручного управления и
> кастомных сценариев ввода.

### Видимость

```lua
EMRAPI.Keyboard:Show()      -- принудительно показать
EMRAPI.Keyboard:Hide()      -- принудительно скрыть
EMRAPI.Keyboard:SetAuto()   -- вернуть авто-режим (по фокусу поля)
local vis = EMRAPI.Keyboard:IsVisible()
```

### Позиция / масштаб / язык

```lua
EMRAPI.Keyboard:SetPosition(x, y, z)  -- мировые координаты
EMRAPI.Keyboard:SetRotation(x, y, z)  -- углы Эйлера
EMRAPI.Keyboard:SetScale(1.5)         -- множитель ИСХОДНОГО размера (1 = как было)
local x = EMRAPI.Keyboard:GetPositionX()  -- также Y/Z

EMRAPI.Keyboard:SetLanguage("en")     -- "en" | "ru" (меняет и язык голосового ввода)
local lang = EMRAPI.Keyboard:GetLanguage()
```

### Свой приём ввода (без поля ввода)

Для игр/приложений с собственным текстовым UI. Колбэки получают ввод, **только
когда нет сфокусированного поля** — фокусное поле всегда в приоритете.

```lua
local typed = ""

EMRAPI.Keyboard:OnInput(function(text)
    typed = typed .. text
    EMRAPI.UI:SetText(label, typed)
end)
EMRAPI.Keyboard:OnBackspace(function()
    typed = typed:sub(1, -2)
    EMRAPI.UI:SetText(label, typed)
end)
EMRAPI.Keyboard:OnEnter(function()
    EMRAPI:Log("Ввод завершён: " .. typed)
end)

EMRAPI.Keyboard:Show()  -- при кастомном вводе показываем клавиатуру сами

-- Когда ввод больше не нужен:
EMRAPI.Keyboard:ClearCallbacks()
EMRAPI.Keyboard:SetAuto()
```

```lua
EMRAPI.Keyboard:Reset()  -- всё в исходное (позиция, масштаб, язык, авто-видимость, колбэки)
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
