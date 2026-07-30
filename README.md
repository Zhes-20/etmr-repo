<p align="center">
  <img src="https://img.shields.io/badge/Platform-iOS%20%7C%20Android-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Language-Lua-2C2D72?style=for-the-badge&logo=lua&logoColor=white" />
  <img src="https://img.shields.io/badge/Engine-Unity-000000?style=for-the-badge&logo=unity&logoColor=white" />
  <img src="https://img.shields.io/badge/API-v1-brightgreen?style=for-the-badge" />
</p>

<h1 align="center">🥽 EnJoyTheMR — Developer Docs</h1>

<p align="center">
  <b>Создавай MR-приложения для смартфонов без пересборки платформы.</b><br/>
  Пиши на Lua, публикуй через Telegram-бота, пользователи скачивают из магазина.
</p>

<p align="center">
  <a href="#-быстрый-старт">Быстрый старт</a> •
  <a href="#-два-способа-разработки">Способы разработки</a> •
  <a href="#-emrapi-reference">API Reference</a> •
  <a href="#-публикация">Публикация</a> •
  <a href="#-faq">FAQ</a>
</p>

---

## ✨ Что такое EnJoyTheMR?

EnJoyTheMR — платформа смешанной реальности, работающая на обычных смартфонах. Сторонние разработчики создают приложения в виде `.emr`-пакетов — зашифрованных архивов с Lua-скриптами и (опционально) Unity AssetBundle.

Пользователи скачивают приложения прямо из встроенного магазина. **Не нужно собирать APK, не нужен доступ к основному проекту.**

```
📱 Пользователь открывает платформу
    └─ 🏪 Заходит в магазин приложений
        └─ ⬇️ Скачивает твоё приложение
            └─ 🚀 Запускает — готово!
```

---

## 🚀 Быстрый старт

Самое простое приложение — **3 файла, 5 минут:**

### 1. Создай папку проекта

```
my-first-app/
├── app.json
└── scripts/
    └── main.lua
```

### 2. Заполни `app.json`

```json
{
  "id": "com.myname.helloworld",
  "name": "Hello World",
  "version": "1.0.0",
  "author": "Твоё имя",
  "description": "Моё первое MR-приложение!",
  "entry": "scripts/main.lua",
  "display": "window",
  "minApiVersion": 1,
  "type": "lua",
  "permissions": ["ui"]
}
```

### 3. Напиши `scripts/main.lua`

```lua
local count = 0

function OnStart()
    local panel = EMRAPI.UI:CreatePanel(0.8, 0.5)
    EMRAPI.UI:SetColor(panel, 0.1, 0.1, 0.15, 0.95)

    local label = EMRAPI.UI:CreateText(panel, "Нажатий: 0")
    EMRAPI.UI:SetPosition(label, 0, 60, 0)
    EMRAPI.UI:SetSize(label, 300, 50)

    EMRAPI.UI:CreateButton(panel, "Нажми! 👆", function()
        count = count + 1
        EMRAPI.UI:SetText(label, "Нажатий: " .. count)
    end)
end
```

### 4. Опубликуй

```
📦 Запакуй папку в .zip
📤 Отправь Telegram-боту командой /upload
✅ Приложение в магазине!
```

**Вот и всё.** Дальше — детали.

---

## 🛠 Два способа разработки

<table>
<tr>
<td width="50%">

### 📝 Способ А — Чистый Lua

**Для кого:** быстрые прототипы, утилиты, 2D-игры.

**Что нужно:** текстовый редактор.

UI и 3D создаются кодом через `EMRAPI`.

```lua
function OnStart()
    local btn = EMRAPI.UI:CreateButton(
        nil, "Привет!", function()
            EMRAPI:Log("Нажато!")
        end
    )
end
```

</td>
<td width="50%">

### 🎨 Способ Б — Unity SDK

**Для кого:** красивый UI, сложные сцены, анимации.

**Что нужно:** Unity + ETMR SDK пакет.

UI собирается визуально, Lua — только логика.

```lua
function start()
    local tmp = valueText:GetComponent(
        "TextMeshProUGUI"
    )
    tmp.text = "Привет!"
end
```

</td>
</tr>
</table>

> [!IMPORTANT]
> У двух способов **разные имена lifecycle-функций!**
> - Чистый Lua: `OnStart()`, `OnUpdate(dt)`, `OnDestroy()`
> - Unity SDK: `start()`, `update()`, `onDestroy()`
> 
> Не путайте их — приложение просто не запустится.

---

## 📄 Манифест `app.json`

Каждое приложение начинается с `app.json` — это его паспорт:

| Поле | Обязательно | Описание |
|:-----|:---:|:----------|
| `id` | ✅ | Уникальный ID (`com.yourname.appname`) |
| `name` | ✅ | Название (видит пользователь) |
| `entry` | ✅* | Путь к главному Lua-скрипту |
| `entryPrefab` | ✅* | Unity: имя корневого префаба |
| `version` | | Версия (`1.0.0`) |
| `author` | | Имя автора |
| `description` | | Описание |
| `display` | | `"window"` *(по умол.)* или `"scene"` |
| `type` | | `"lua"` *(по умол.)* или `"bundle"` |
| `minApiVersion` | | Мин. версия API (сейчас `1`) |
| `windowWidth` | | Ширина окна (по умол. `500`) |
| `windowHeight` | | Высота окна (по умол. `400`) |
| `permissions` | | Массив прав (см. ниже) |

> **\*** Нужен хотя бы один из `entry` или `entryPrefab`.

### Режимы отображения

| Режим | `display` | Когда использовать |
|:------|:---------:|:-------------------|
| 🪟 **Окно** | `"window"` | Утилиты, 2D-игры, настройки — плашка перед камерой |
| 🌍 **Сцена** | `"scene"` | Иммерсивный 3D, AR-объекты, физика в пространстве |

---

## 🔐 Права (Permissions)

Приложение объявляет нужные права в `app.json`. Без права — нет доступа к модулю.

### Стандартные *(запрашиваются молча)*

| Право | Модуль | Что даёт |
|:------|:-------|:---------|
| `"ui"` | `EMRAPI.UI` | Кнопки, текст, панели, слайдеры, изображения |
| `"scene"` | `EMRAPI.Scene` | 3D-примитивы, физика, привязка к рукам |
| `"input"` | `EMRAPI.Input` | Положение головы/рук, жесты |
| `"audio"` | `EMRAPI.Audio` | Воспроизведение звуков |
| `"storage"` | `EMRAPI.Storage` | Локальное хранилище ключ-значение |

### Опасные *(пользователь увидит диалог)*

| Право | Что даёт |
|:------|:---------|
| `"network"` | HTTP-запросы (GET/POST) |
| `"unsafe_execution"` | Полный доступ к C# API, без сандбокса |

> `EMRAPI.Hands`, `EMRAPI.Camera`, `EMRAPI.Json`, `EMRAPI.Tween`, `EMRAPI.Timer` — прав не требуют, доступны всегда.

---

## 🔄 Жизненный цикл

### Чистый Lua-режим

```lua
function OnStart()          -- 🟢 Приложение запустилось
    -- Инициализация UI, переменных, таймеров
end

function OnUpdate(dt)       -- 🔄 Каждый кадр (dt = секунды между кадрами)
    -- Анимация, физика, обновление UI
end

function OnPause()          -- ⏸️ Приложение свёрнуто
function OnResume()         -- ▶️ Приложение развёрнуто

function OnDestroy()        -- 🔴 Приложение закрывается
    -- Отмена таймеров, освобождение ресурсов
end
```

### Unity SDK (LuaBehaviour)

```lua
function start()            -- 🟢 Компонент инициализирован
function update()           -- 🔄 Каждый кадр (без dt!)
function fixedUpdate()      -- ⚙️ Фиксированный шаг физики
function lateUpdate()       -- 🔚 После всех Update
function onEnable()         -- ✅ Объект активирован
function onDisable()        -- ❌ Объект деактивирован
function onDestroy()        -- 🔴 Объект уничтожен
```

> [!TIP]
> В LuaBehaviour `update()` **не получает dt**. Используй `EMRAPI.System:GetDeltaTime()`.

---

## 📘 EMRAPI Reference

### 📋 Базовые методы *(всегда доступны)*

```lua
EMRAPI:Log("сообщение")           -- Вывод в консоль
EMRAPI:LogWarning("предупреждение")
EMRAPI:LogError("ошибка")
```

### ℹ️ EMRAPI.App

```lua
EMRAPI.App.id                      -- "com.developer.myapp"
EMRAPI.App.displayMode             -- "window" или "scene"
EMRAPI.App:GetName()               -- название
EMRAPI.App:GetVersion()            -- версия
EMRAPI.App:GetAuthor()             -- автор
EMRAPI.App:GetDescription()        -- описание
EMRAPI.App:RequestExit()           -- закрыть приложение
```

### ⏱ EMRAPI.System

```lua
EMRAPI.System:GetTime()            -- секунды с запуска
EMRAPI.System:GetDeltaTime()       -- время между кадрами
EMRAPI.System:GetPlatform()        -- "ios" | "android" | "editor"
EMRAPI.System:GetTrackingMode()    -- "6dof" (позиция головы отслеживается)
                                   -- или "3dof" (только поворот)
```

### 🔢 EMRAPI.Math

```lua
EMRAPI.Math:Random(0, 100)         -- случайное число
EMRAPI.Math:Lerp(a, b, 0.5)        -- линейная интерполяция
EMRAPI.Math:Clamp(val, 0, 1)       -- ограничение диапазона
EMRAPI.Math:Distance(x1,y1,z1, x2,y2,z2)  -- расстояние
```

---

### 🖼 EMRAPI.UI *(право: `ui`)*

<details>
<summary><b>Создание элементов</b></summary>

```lua
-- Панель (ширина и высота в метрах мирового пространства)
local panel = EMRAPI.UI:CreatePanel(1.0, 0.6)

-- Кнопка
local btn = EMRAPI.UI:CreateButton(panel, "Текст кнопки", function()
    EMRAPI:Log("Нажата!")
end)

-- Текст
local label = EMRAPI.UI:CreateText(panel, "Hello!")

-- Изображение (из архива приложения)
local img = EMRAPI.UI:CreateImage(panel, "images/logo.png")

-- Поле ввода
local input = EMRAPI.UI:CreateInputField(panel, "Placeholder...", function(value)
    EMRAPI:Log("Ввод: " .. value)
end)

-- Слайдер
local slider = EMRAPI.UI:CreateSlider(panel, 0, 100, function(val)
    EMRAPI:Log("Значение: " .. val)
end)

-- Чекбокс
local toggle = EMRAPI.UI:CreateToggle(panel, "Опция", function(state)
    EMRAPI:Log("Включено: " .. tostring(state))
end)

-- Скроллируемая область
local scroll = EMRAPI.UI:CreateScrollView(panel, 1.0, 0.5)
```

</details>

<details>
<summary><b>Манипуляция элементами</b></summary>

```lua
EMRAPI.UI:SetPosition(el, x, y, z)     -- позиция
EMRAPI.UI:SetSize(el, width, height)    -- размер
EMRAPI.UI:SetColor(el, r, g, b, a)     -- цвет (0.0 – 1.0)
EMRAPI.UI:SetText(el, "новый текст")   -- изменить текст
EMRAPI.UI:SetActive(el, false)         -- скрыть элемент
EMRAPI.UI:Destroy(el)                  -- удалить

local text = EMRAPI.UI:GetText(el)     -- прочитать текст
```

</details>

---

### 🧊 EMRAPI.Scene *(право: `scene`)*

<details>
<summary><b>Создание 3D-объектов</b></summary>

```lua
-- Примитивы: "cube", "sphere", "capsule", "cylinder", "plane", "quad"
local cube = EMRAPI.Scene:CreatePrimitive("cube")
local ball = EMRAPI.Scene:CreatePrimitive("sphere")

-- Пустой объект
local empty = EMRAPI.Scene:CreateObject("marker")

-- Корень сцены
local root = EMRAPI.Scene:GetRoot()
```

</details>

<details>
<summary><b>Трансформации</b></summary>

```lua
EMRAPI.Scene:SetPosition(obj, 0, 1, 3)     -- позиция
EMRAPI.Scene:SetRotation(obj, 0, 45, 0)    -- поворот (углы Эйлера)
EMRAPI.Scene:SetScale(obj, 0.5, 0.5, 0.5)  -- масштаб
EMRAPI.Scene:SetColor(obj, 1, 0, 0, 1)     -- красный

EMRAPI.Scene:SetParent(child, parent)       -- иерархия

local x = EMRAPI.Scene:GetPositionX(obj)
local y = EMRAPI.Scene:GetPositionY(obj)
local z = EMRAPI.Scene:GetPositionZ(obj)

local found = EMRAPI.Scene:Find("ObjectName")  -- поиск по имени
EMRAPI.Scene:Destroy(obj)                       -- удалить
```

</details>

<details>
<summary><b>Физика</b></summary>

```lua
-- Rigidbody (масса, гравитация)
EMRAPI.Scene:AddRigidbody(obj, 1.0, true)

-- Коллайдеры
EMRAPI.Scene:AddBoxCollider(obj)
EMRAPI.Scene:AddSphereCollider(obj, 0.5)  -- радиус
```

</details>

<details>
<summary><b>Привязка к рукам 🤚</b></summary>

```lua
-- Жёсткая привязка
EMRAPI.Scene:AttachToHand(obj, "right", "IndexTip")

-- Плавная привязка (с интерполяцией)
EMRAPI.Scene:AttachToHandSmooth(obj, "right", "PalmCenter", 10.0, 8.0)
--                               объект, рука, точка, скорость позиции, скорость поворота

-- Отвязать
EMRAPI.Scene:DetachFromHand(obj)
```

**Точки привязки:**

```
PalmCenter · IndexTip · ThumbTip · MiddleTip · RingTip · PinkyTip
IndexKnuckle · MiddleKnuckle · RingKnuckle · PinkyKnuckle · Wrist
```

Или индекс лендмарка MediaPipe: `"0"` – `"20"`

</details>

<details>
<summary><b>Привязка к голове (HUD) 👁</b></summary>

```lua
-- Объект остаётся где стоит и дальше следует за головой
EMRAPI.Scene:AttachToHead(obj)

-- Явное смещение от камеры: x вправо, y вверх, z вперёд (метры)
EMRAPI.Scene:AttachToHeadAt(obj, 0.15, -0.1, 0.6)

EMRAPI.Scene:DetachFromHead(obj)   -- отвязать
```

</details>

---

### 🎯 EMRAPI.Input *(право: `input`)*

```lua
-- Отслеживается ли рука ("left" / "right")
local tracked = EMRAPI.Input:IsHandTracked("right")

-- Позиция руки — ТРИ возвращаемых значения (не Vector3!)
-- Точка: имя ("PalmCenter", "IndexTip", ...) или индекс лендмарка "0"–"20"
local x, y, z = EMRAPI.Input:GetHandPosition("right", "PalmCenter")

-- Поворот руки — четыре значения (кватернион)
local qx, qy, qz, qw = EMRAPI.Input:GetHandRotation("right", "PalmCenter")

-- Упрощённые геттеры (PalmCenter)
local px = EMRAPI.Input:GetHandPositionX("left")  -- также Y/Z

-- Жесты (строка: "Pinch", "Fist", "Five", "OK", "Point", "Like"...)
local gesture = EMRAPI.Input:GetHandGesture("right")
local pinch   = EMRAPI.Input:GetPinchStrength("right")  -- float
local grab    = EMRAPI.Input:IsGrabbing("left")          -- bool

-- Голова (основная камера)
local hx = EMRAPI.Input:GetHeadPositionX()  -- также Y/Z
local rw = EMRAPI.Input:GetHeadRotationW()  -- также X/Y/Z
```

> [!NOTE]
> Событий жестов нет — опрашивай `GetHandGesture` в `OnUpdate` или таймере. Вибрации и контроллеров нет — только руки.

---

### ⏲ EMRAPI.Timer

```lua
-- Повторяющийся таймер
local timerId = EMRAPI.Timer:Repeat(1.0, function()
    EMRAPI:Log("тик!")
end)

-- Одноразовый таймер
local delayId = EMRAPI.Timer:Delay(3.0, function()
    EMRAPI:Log("прошло 3 секунды")
end)

-- Отмена
EMRAPI.Timer:Cancel(timerId)
```

> [!WARNING]
> **Всегда отменяй таймеры в `OnDestroy` / `onDestroy`!**
> Иначе они продолжат работать после закрытия приложения.

---

### 🔊 EMRAPI.Audio *(право: `audio`)*

```lua
-- Одноразовые звуки
EMRAPI.Audio:Play("sounds/click.wav")
EMRAPI.Audio:PlayAtPoint("sounds/boom.wav", 0, 1, 3)  -- 3D-звук

-- Фоновая музыка: Play/PlayAtPoint/PlayLoop возвращают id источника
local music = EMRAPI.Audio:PlayLoop("sounds/bg.ogg")   -- зацикленный
EMRAPI.Audio:SetVolume(music, 0.4)   -- громкость источника
EMRAPI.Audio:SetPitch(music, 1.2)    -- тон/скорость (0.1–3.0)
EMRAPI.Audio:SetLoop(music, false)   -- доиграет и остановится
EMRAPI.Audio:IsPlaying(music)        -- bool
EMRAPI.Audio:Stop(music)             -- остановить источник

-- Глобально
EMRAPI.Audio:Stop()                  -- остановить всё
EMRAPI.Audio:SetVolume(0.8)          -- базовая громкость (0.0–1.0)
```

Пути — относительно корня архива приложения.

---

### 💾 EMRAPI.Storage *(право: `storage`)*

Данные изолированы по `id` приложения. Сохраняются между запусками.

```lua
-- Строки
EMRAPI.Storage:Set("username", "Player1")
local name = EMRAPI.Storage:GetString("username", "Guest")

-- Числа
EMRAPI.Storage:SetNumber("highscore", 9001)
local score = EMRAPI.Storage:GetNumber("highscore", 0)

-- Булевы
EMRAPI.Storage:SetBool("sound_on", true)
local sound = EMRAPI.Storage:GetBool("sound_on", true)

-- Удаление
EMRAPI.Storage:Remove("key")

-- ⚠️ ОБЯЗАТЕЛЬНО вызови после изменений!
EMRAPI.Storage:Save()
```

---

### 🌐 EMRAPI.Network *(право: `network` ⚠️ опасное)*

```lua
-- GET-запрос
EMRAPI.Network:Get("https://api.example.com/data", function(response, error)
    if error then
        EMRAPI:LogError("Ошибка: " .. error)
    else
        EMRAPI:Log("Ответ: " .. response)
    end
end)

-- POST-запрос
EMRAPI.Network:Post("https://api.example.com/submit",
    '{"score": 100}',
    function(response, error)
        -- ...
    end
)
```

---

### 🤚 EMRAPI.Hands *(права не нужны)*

Управление видом скелета рук. Всё откатывается при закрытии приложения.
`hand` — `"left"`, `"right"` или `"both"`; цвета — `0.0–1.0`.

```lua
EMRAPI.Hands:SetColor("both", 1, 0, 0)       -- весь скелет красный
EMRAPI.Hands:SetPointColor("left", 1, 1, 0)  -- только точки
EMRAPI.Hands:SetLinkColor("left", 0, 1, 1)   -- только связи
EMRAPI.Hands:SetAlpha("both", 0.3)           -- полупрозрачность
EMRAPI.Hands:SetVisible("right", false)      -- скрыть руку
EMRAPI.Hands:IsVisible("left")               -- bool
EMRAPI.Hands:Reset("both")                   -- вид по умолчанию
```

> Скрывается только визуализация — трекинг и `EMRAPI.Input` работают дальше.

---

### 📷 EMRAPI.Camera *(права не нужны)*

Фон с камеры телефона (passthrough). Откатывается при закрытии.

```lua
EMRAPI.Camera:SetBackground("hide")    -- "show" | "hide" | "blur"
EMRAPI.Camera:GetBackground()          -- текущий режим
EMRAPI.Camera:Reset()                  -- вернуть как было
```

`"hide"` — полное погружение (VR), `"blur"` — фокус на контенте.

---

### 📄 EMRAPI.Json *(права не нужны)*

В Lua нет встроенного JSON — используй этот модуль (особенно с `EMRAPI.Network`).

```lua
local s = EMRAPI.Json:Encode({ score = 100, tags = {"a", "b"} })
-- '{"score":100,"tags":["a","b"]}'

local t = EMRAPI.Json:Decode('{"items":[1,2,3]}')
if t then EMRAPI:Log(t.items[1]) end   -- ВСЕГДА проверяй на nil!
```

- Таблица с ключами `1..n` → массив, иначе → объект
- Ошибка → `nil` + причина в логе

---

### 🎬 EMRAPI.Tween *(права не нужны)*

Плавные анимации без ручной лерпы в `OnUpdate`. Локальные координаты, сглаживание smoothstep.

```lua
local id = EMRAPI.Tween:MoveTo(obj, 0, 1, 2, 0.5)   -- к точке за 0.5с
EMRAPI.Tween:RotateTo(obj, 0, 180, 0, 1.0)           -- углы Эйлера
EMRAPI.Tween:ScaleTo(obj, 2, 2, 2, 0.3)

-- Колбэк по завершении — последним аргументом
EMRAPI.Tween:MoveTo(obj, 0, 0, 1, 0.5, function()
    EMRAPI:Log("приехали")
end)

EMRAPI.Tween:Cancel(id)      -- объект замирает где был
EMRAPI.Tween:CancelAll()
EMRAPI.Tween:IsRunning(id)   -- bool
```

> Новый твин того же типа на объекте заменяет старый. Перед `AttachToHead`/`AttachToHand` отменяй твины объекта — подерутся с новым родителем.

---

## 🎨 Unity SDK — подробности

### Компоненты

| Компонент | Назначение |
|:----------|:-----------|
| `LuaBehaviour` | Мост Unity ↔ Lua. Вешается на GameObject, указываешь `.lua` файл |
| `LuaUIButton` | Кнопка → Lua. Без кода: задай имя функции и аргумент в инспекторе |
| `EtmrSimulatorHost` | Play-Mode тест: поднимает рантайм прямо в редакторе |
| `EtmrBundleBuilder` | Меню `ETMR → Build App Bundle` — собирает готовый пакет |

### LuaBehaviour — как MonoBehaviour, но на Lua

```lua
-- self = this (как в C#)
-- gameObject, transform — автоматически доступны
-- Инъекции — как public поля: перетащил в инспекторе → переменная в Lua

local tmp   -- кэш компонента

function start()
    -- GetComponent — как в C#
    tmp = valueText:GetComponent("TextMeshProUGUI")
    tmp.text = "Готово!"
    tmp.fontSize = 32
end

function update()
    -- Каждый кадр
    local dt = EMRAPI.System:GetDeltaTime()
end

function onDestroy()
    EMRAPI:Log("Закрыто")
end
```

### Доступные Unity-типы из Lua

Через xLua экспортировано **~120 типов**. Вот основные:

```
Unity Core     │ GameObject, Transform, RectTransform, Time
Physics        │ Rigidbody, BoxCollider, SphereCollider, CharacterController, Joint-ы...
Rendering      │ Camera, Light, MeshRenderer, SpriteRenderer, ParticleSystem, LineRenderer
UI             │ Image, Button, Slider, Toggle, ScrollRect, LayoutGroup-ы, Canvas
TextMeshPro    │ TextMeshProUGUI, TMP_InputField, TMP_Text
Animation      │ Animator, Animation, AnimationClip
Math           │ Vector2, Vector3, Quaternion, Color, Mathf, Physics (Raycast)
```

```lua
-- Примеры прямого доступа:
local rb = obj:GetComponent("Rigidbody")
rb.mass = 2.0
rb.useGravity = false

local img = bar:GetComponent("Image")
img.fillAmount = 0.75
img.color = CS.UnityEngine.Color(1, 0, 0, 1)
```

> [!NOTE]
> `CS.UnityEngine.*` работает **только в LuaBehaviour** (Unity SDK).
> В чистом Lua используй `EMRAPI.Scene` / `EMRAPI.UI`.

---

## 📦 Публикация

### Шаг за шагом

```
 ① Готовишь приложение
    ├── Чистый Lua → папка с app.json + scripts/
    └── Unity SDK  → ETMR → Build App Bundle

 ② Пакуешь в .zip

 ③ Отправляешь Telegram-боту
    └── /upload → прикрепляешь .zip

 ④ Бот спрашивает:
    ├── 🖼 Иконку (256×256 PNG)
    ├── 📸 Превью для магазина (16:9 PNG)
    └── 📱 Платформу (только для Unity-приложений)

 ⑤ Готово! Приложение в магазине ✅
```

### Команды бота

| Команда | Что делает |
|:--------|:-----------|
| `/upload` | Загрузить / обновить приложение |
| `/list` | Список всех приложений |
| `/remove <id>` | Удалить приложение |
| `/rebuild` | Пересобрать каталог |

### Lua vs Unity — что отправлять боту?

| | Чистый Lua | Unity SDK |
|:--|:-----------|:----------|
| Содержимое ZIP | Исходники | Собранный AssetBundle |
| Платформы | Автоматически обе | Бот спросит (iOS / Android / Обе) |
| Несколько файлов? | Нет, один | Да, если "Обе" — два `.emr` |

---

## 📚 Примеры

### 🎮 Dino Run — 2D-игра

<details>
<summary>Показать код</summary>

**`app.json`**
```json
{
  "id": "com.example.dinorun",
  "name": "Dino Run",
  "version": "1.0.0",
  "entry": "scripts/main.lua",
  "display": "window",
  "minApiVersion": 1,
  "type": "lua",
  "permissions": ["ui"]
}
```

**`scripts/main.lua`**
```lua
local score = 0
local isGameOver = false
local panel, dinoObj, cactusObj, scoreText

local GRAVITY = -1800
local JUMP_VEL = 650
local GROUND_Y = -80

local dinoY = GROUND_Y
local velY = 0
local cactusX = 250
local speed = 300

function OnStart()
    panel = EMRAPI.UI:CreatePanel(0.48, 0.32)

    scoreText = EMRAPI.UI:CreateText(panel, "Score: 0")
    EMRAPI.UI:SetPosition(scoreText, 150, 130, 0)

    dinoObj = EMRAPI.UI:CreatePanel(0.04, 0.04)
    EMRAPI.UI:SetColor(dinoObj, 0.2, 0.8, 0.2, 1)

    cactusObj = EMRAPI.UI:CreatePanel(0.02, 0.05)
    EMRAPI.UI:SetColor(cactusObj, 0.8, 0.2, 0.2, 1)

    -- Прозрачная кнопка на весь экран для прыжка
    local area = EMRAPI.UI:CreateButton(panel, "", function()
        if dinoY <= GROUND_Y then velY = JUMP_VEL end
    end)
    EMRAPI.UI:SetSize(area, 480, 320)
    EMRAPI.UI:SetColor(area, 0, 0, 0, 0)
end

function OnUpdate(dt)
    if isGameOver then return end

    velY = velY + GRAVITY * dt
    dinoY = dinoY + velY * dt
    if dinoY < GROUND_Y then dinoY = GROUND_Y; velY = 0 end

    EMRAPI.UI:SetPosition(dinoObj, -150, dinoY, 0)

    cactusX = cactusX - speed * dt
    if cactusX < -250 then
        cactusX = 250
        score = score + 10
        speed = speed + 15
        EMRAPI.UI:SetText(scoreText, "Score: " .. score)
    end
    EMRAPI.UI:SetPosition(cactusObj, cactusX, GROUND_Y + 5, 0)
end
```

</details>

### 🌐 IP Checker — сеть

<details>
<summary>Показать код</summary>

**`app.json`** — `permissions: ["ui", "network"]`

```lua
function OnStart()
    local panel = EMRAPI.UI:CreatePanel(0.8, 0.4)
    local label = EMRAPI.UI:CreateText(panel, "Загрузка...")
    EMRAPI.UI:SetSize(label, 350, 80)

    EMRAPI.Network:Get("https://api.ipify.org", function(resp, err)
        if err then
            EMRAPI.UI:SetText(label, "Ошибка: " .. err)
        else
            EMRAPI.UI:SetText(label, "Ваш IP: " .. resp)
        end
    end)
end
```

</details>

### 🤚 Объект на руке — AR

<details>
<summary>Показать код</summary>

**`app.json`** — `display: "scene"`, `permissions: ["scene", "input"]`

```lua
function OnStart()
    local cube = EMRAPI.Scene:CreatePrimitive("cube")
    EMRAPI.Scene:SetScale(cube, 0.05, 0.05, 0.05)
    EMRAPI.Scene:SetColor(cube, 0.3, 0.7, 1.0, 1.0)
    EMRAPI.Scene:AttachToHandSmooth(cube, "right", "PalmCenter", 12.0, 10.0)
end
```

</details>

---

## ⚠️ Частые ошибки

| Проблема | Причина | Решение |
|:---------|:--------|:--------|
| `attempt to call a nil value` | `.` вместо `:` | `EMRAPI.Timer:Repeat()`, не `EMRAPI.Timer.Repeat()` |
| `OnStart` не вызывается | Неправильный регистр | Чистый Lua: `OnStart`. LuaBehaviour: `start` |
| GetComponent → nil | Неверный регистр типа | `"TextMeshProUGUI"`, не `"textmeshprougui"` |
| Данные не сохраняются | Забыл `Save()` | `EMRAPI.Storage:Save()` после каждого изменения |
| Таймеры после закрытия | Не отменены | `EMRAPI.Timer:Cancel(id)` в `OnDestroy` |
| Приложение не в магазине | Не отправил боту | Запакуй в `.zip` → `/upload` |
| `App requires API vN` | Слишком высокий `minApiVersion` | Поставь `1` |
| Краш на iOS | Не сгенерированы обёртки | `XLua → Generate Code` в Unity |
| `attempt to index a nil value` после Decode | Битый JSON | Проверяй результат `EMRAPI.Json:Decode` на `nil` |
| Объект дёргается после AttachToHead | Живой твин в старых координатах | `EMRAPI.Tween:CancelAll()` перед привязкой |

---

## ❓ FAQ

<details>
<summary><b>Мне нужен Unity для разработки?</b></summary>

**Нет!** Для чистого Lua-режима достаточно любого текстового редактора. Unity нужен только если хочешь красивый визуальный UI или сложные 3D-сцены с анимациями.

</details>

<details>
<summary><b>Как тестировать без устройства?</b></summary>

**Чистый Lua:** положи папку приложения в `TestApps/` проекта и запусти в Unity Editor.

**Unity SDK:** добавь `EtmrSimulatorHost` на сцену → Play.

</details>

<details>
<summary><b>Безопасно ли публиковать приложение?</b></summary>

Да. Бот шифрует твой код. Ключи шифрования хранятся только на сервере — их нет ни в SDK, ни в APK.

</details>

<details>
<summary><b>Можно ли использовать несколько Lua-файлов?</b></summary>

Да! Используй `require("modulename")` — файлы ищутся в папке `scripts/`. Расширение `.lua` добавляется автоматически.

```lua
-- scripts/utils.lua
local M = {}
function M.clamp(v, lo, hi) return math.max(lo, math.min(hi, v)) end
return M

-- scripts/main.lua
local utils = require("utils")
local x = utils.clamp(150, 0, 100)  -- 100
```

</details>

<details>
<summary><b>Чем отличается `EMRAPI.Scene` от прямого Unity API?</b></summary>

`EMRAPI.Scene` — безопасная обёртка, работает в обоих режимах и автоматически привязывает объекты к корню приложения. Прямой Unity API (`GetComponent`, `CS.UnityEngine.*`) доступен только в LuaBehaviour и даёт полный контроль.

Для большинства задач хватает EMRAPI. Прямой API нужен для тонкой настройки (цвета материалов, анимации, UI-свойства вроде `fillAmount`).

</details>

---

<p align="center">
  <b>Остались вопросы? Пиши в Telegram-бота!</b><br/>
  <sub>Made with 🤍 by the Zhes</sub>
</p>
