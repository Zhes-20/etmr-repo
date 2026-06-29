# EnJoyTheMR (EMR) — Архитектура и руководство разработчика

## 1. Философия платформы

EnJoyTheMR — Unity-платформа смешанной реальности для смартфонов.
Она позволяет сторонним разработчикам создавать приложения **без пересборки основного APK/IPA**.

Приложение — это `.emr` пакет (зашифрованный ZIP-архив), содержащий `app.json`, Lua-скрипты и опционально AssetBundle. Пользователь загружает `.emr` из магазина приложений (App Store) прямо в платформу.

Поддерживаются два способа создания приложений — они могут использоваться вместе:

| Режим | Описание |
|---|---|
| **Lua-режим** | Логика и UI на чистом xLua. Не нужен Unity-проект у разработчика. Всё в текстовых файлах. UI создаётся кодом через `EMRAPI.UI`, 3D — через `EMRAPI.Scene`. |
| **Unity-режим (SDK)** | Разработчик открывает Unity, строит UI/сцену визуально, прикрепляет `LuaBehaviour`, пакует в AssetBundle. Lua — только бизнес-логика, Unity — визуальный дизайн. |

Оба режима запускаются одним и тем же `LuaAppRuntime` и используют один `EMRAPI`.

---

## 2. Визуальные режимы приложения

Задаются полем `"display"` в `app.json`:

### Window Mode (`"display": "window"`)
Приложение появляется как 2D-плашка перед камерой (с заголовком и кнопкой ×). Всё внутри плашки — UI-элементы. Подходит для утилит, 2D-игр, настроек.

Размер окна настраивается полями `windowWidth` / `windowHeight` в `app.json` (умолчание: 500×400 UI-единиц).

### Scene Mode (`"display": "scene"`)
Приложение захватывает всю MR-среду: 3D-объекты, физика, привязка к рукам. Подходит для иммерсивного контента, AR-приложений.

---

## 3. Манифест `app.json`

Каждое приложение содержит файл `app.json` в корне архива. Это его «паспорт»:

```json
{
  "id": "com.developer.myapp",
  "name": "My Application",
  "version": "1.0.0",
  "author": "Developer Name",
  "description": "Описание приложения",

  "entry": "scripts/main.lua",
  "entryPrefab": "MyAppRoot",

  "display": "window",
  "minApiVersion": 1,
  "type": "lua",

  "windowWidth": 500,
  "windowHeight": 400,

  "permissions": ["ui", "audio"]
}
```

### Поля

| Поле | Обяз. | Описание |
|---|---|---|
| `id` | ✅ | Уникальный идентификатор (reverse-domain: `com.yourname.appname`). Используется для изоляции хранилища и прав. |
| `name` | ✅ | Отображаемое имя приложения. |
| `version` | — | Версия (семантическая, например `1.0.0`). |
| `author` | — | Имя разработчика. |
| `description` | — | Описание приложения. |
| `entry` | ✅* | Путь к главному Lua-скрипту (относительно корня архива). Например: `scripts/main.lua`. |
| `entryPrefab` | ✅* | **Unity-режим:** имя корневого префаба в AssetBundle. Будет автоматически инстанцирован при запуске. |
| `type` | — | `"lua"` — только скрипты (по умолчанию); `"bundle"` — есть AssetBundle (Unity-режим). |
| `display` | — | `"window"` (по умолчанию) или `"scene"`. |
| `windowWidth` | — | Ширина окна в UI-единицах (умолчание: 500). Только для `window`. |
| `windowHeight` | — | Высота окна в UI-единицах (умолчание: 400). Только для `window`. |
| `minApiVersion` | — | Минимальная версия EMRAPI (сейчас: 1). |
| `permissions` | — | Массив строк — запрашиваемые разрешения (см. ниже). |

> **✅* Важно:** приложение валидно если задан хотя бы один из `entry` **или** `entryPrefab` (или оба).

---

## 4. Система прав (Permissions)

Lua-скрипт может обращаться к EMRAPI-подсистемам, если соответствующее право объявлено в `app.json`.

**Стандартные права** (запрашиваются молча):

| Право | Что открывает |
|---|---|
| `ui` | `EMRAPI.UI` — 2D-элементы (панели, кнопки, текст, изображения, слайдеры, скролл…) |
| `audio` | `EMRAPI.Audio` — воспроизведение WAV/OGG файлов из архива |
| `storage` | `EMRAPI.Storage` — сохранение пар ключ-значение (изолировано по `id`) |
| `scene` | `EMRAPI.Scene` — 3D-примитивы, физика, привязка к рукам |
| `input` | `EMRAPI.Input` — положение головы/рук, жесты |

**Опасные права** (показывают пользователю диалог с подтверждением):

| Право | Что открывает |
|---|---|
| `network` | `EMRAPI.Network` — HTTP-запросы GET/POST |
| `unsafe_execution` | Отключает xLua-сандбокс, открывает `io`, `os`, `CS.System.IO`, рефлексию C# |

После однократного подтверждения решение сохраняется за конкретной версией приложения.

---

## 5. Безопасность и сандбокс

По умолчанию xLua-среда ограничена:
- Удалены: `io`, `os.execute`, `os.remove`, `os.exit`, `loadfile`, `dofile`
- Заблокированы: `CS.System.IO`, `CS.System.Net`, `CS.System.Reflection`, `CS.System.Diagnostics`, `CS.System.Threading`

Для доверенных приложений можно запросить `unsafe_execution` — пользователь увидит предупреждение.

---

## 6. Два способа создания приложений

### Способ А — Чистый Lua (без Unity у разработчика)

Самый простой путь. Разработчик пишет `app.json` и `.lua`-файлы в любом текстовом редакторе.

**Структура папки:**
```
my-app/
├── app.json
├── icon.png              ← иконка (опционально, для магазина)
├── sounds/               ← звуки (если используется EMRAPI.Audio)
│   └── click.wav
├── images/               ← картинки (если используется EMRAPI.UI:CreateImage)
│   └── logo.png
└── scripts/
    └── main.lua          ← точка входа (указана в entry)
```

**Жизненный цикл (глобальные функции с заглавной буквы):**

```lua
function OnStart()
    -- Вызывается один раз при запуске приложения.
    -- Здесь инициализация: создание UI, начальное состояние.
end

function OnUpdate(dt)
    -- Вызывается каждый кадр. dt = Time.deltaTime (число секунд).
    -- Используй для анимаций, физики, обновления UI.
end

function OnPause()   end  -- Приложение свернулось
function OnResume()  end  -- Приложение развернулось

function OnDestroy()
    -- Вызывается при закрытии приложения.
    -- Здесь освобождение ресурсов (отмена таймеров и т.д.)
end
```

**Важно:**
- Имена функций — **с заглавной буквы** (`OnStart`, не `start`).
- `OnUpdate` получает один параметр `dt` (дельта времени).
- UI и 3D-объекты создаются через `EMRAPI.UI` и `EMRAPI.Scene`.
- Можно использовать `require("modulename")` — файлы ищутся в `scripts/`.

**Цикл разработки (от идеи до магазина):**
1. Написал `scripts/main.lua` и `app.json`
2. Протестировал (при желании — через TestApps/ в Unity-проекте)
3. Упаковал папку в `.zip` → отправил Telegram-боту платформы
4. Бот собирает зашифрованный `.emr` пакет
5. Бот публикует в магазин (App Store) — доступен для загрузки всем пользователям

---

### Способ Б — Unity SDK (LuaBehaviour + AssetBundle)

Разработчик строит UI и сцену **визуально в Unity**, а Lua управляет логикой.
Не нужно создавать элементы кодом через `EMRAPI.UI` — всё расставляется мышкой.

**Что входит в SDK (`Assets/ETMR/AppSDK`):**
- `LuaBehaviour` — компонент, крепится на любой GameObject. Указываешь `.lua`-файл и инъекции (Unity-объекты, видимые в Lua по именам).
- `LuaUIButton` — компонент на кнопках: вызывает Lua-функцию при нажатии (без написания C#).
- `EtmrSimulatorHost` — Play-Mode симулятор: поднимает настоящий `LuaEnv`, позволяет тестировать без сборки бандла.
- `EtmrBundleBuilder` — меню `ETMR → Build App Bundle`: собирает AssetBundle + `app.json`.

**Структура выходной папки:**
```
com.dev.myapp/
├── app.json       ← display, type:bundle, entryPrefab
└── bundle         ← AssetBundle с префабом и .lua-файлами
```

**Жизненный цикл LuaBehaviour (нижний регистр):**

```lua
function start()       -- аналог Start() в C#
function update()      -- аналог Update() — каждый кадр (если callUpdate = true)
function fixedUpdate() -- аналог FixedUpdate()
function lateUpdate()  -- аналог LateUpdate()
function onEnable()    -- аналог OnEnable()
function onDisable()   -- аналог OnDisable()
function onDestroy()   -- аналог OnDestroy()
```

**Важно: отличие от Lua-режима!**
- LuaBehaviour: **нижний регистр** (`start`, `update`, `onDestroy`)
- Чистый Lua: **заглавная буква** (`OnStart`, `OnUpdate`, `OnDestroy`)
- LuaBehaviour `update()` не получает `dt` — используй `EMRAPI.System:GetDeltaTime()`

**Цикл разработки (от идеи до магазина):**
1. Собрал UI-префаб в Unity, навесил `LuaBehaviour` и `LuaUIButton`
2. **Тест:** добавил `EtmrSimulatorHost` на сцену → Play → видишь живой результат
3. **Сборка:** меню `ETMR → Build App Bundle`
4. Упаковал папку в `.zip` → отправил Telegram-боту
5. **ВАЖНО:** для Unity-приложений бот спросит платформу (iOS / Android / Обе), т.к. AssetBundle платформо-зависим
6. Бот публикует в магазин

---

## 7. LuaBehaviour — как писать скрипты (C#-стиль)

### Концепция

Lua-скрипт на `LuaBehaviour` пишется **как C#-скрипт на MonoBehaviour**:

| C# (MonoBehaviour) | Lua (LuaBehaviour) |
|---|---|
| `this` | `self` |
| `this.gameObject` | `self.gameObject` / `gameObject` |
| `this.transform` | `self.transform` / `transform` |
| `GetComponent<Rigidbody>()` | `obj:GetComponent("Rigidbody")` |
| `public TMP_Text label;` | Инъекция `label` (перетащил в инспекторе → переменная в Lua) |
| `label.text = "hi"` | `label:GetComponent("TextMeshProUGUI").text = "hi"` |
| `Start()` | `start()` |
| `Update()` | `update()` |
| `OnDestroy()` | `onDestroy()` |

### Автоматические переменные (доступны без объявления)

```lua
self       -- текущий LuaBehaviour (как this в C#)
gameObject -- self.gameObject
transform  -- self.transform
```

### `self` = `this`

`self` — ссылка на текущий компонент `LuaBehaviour` (наследует `MonoBehaviour`).
Через `self` доступны все стандартные Unity-свойства и методы:

```lua
self.gameObject              -- текущий GameObject
self.transform               -- Transform
self.transform.position      -- Vector3 позиция в мире
self:GetComponent("Rigidbody")   -- как GetComponent<T> в C#
self.gameObject:SetActive(false) -- выключить объект
```

### Инъекции = public поля

Во вкладке **Injections** на компоненте `LuaBehaviour` ты перетаскиваешь Unity-объекты — точно как public поля в C#-скрипте.

Каждая инъекция — это пара **имя → объект**. В Lua объект доступен как глобальная переменная по имени:

```lua
-- В инспекторе: name="myLabel", value=(перетащил GameObject с TMP)
-- В Lua:
local tmp = myLabel:GetComponent("TextMeshProUGUI")
tmp.text = "Привет!"
tmp.fontSize = 24
```

### Хелперы-шорткаты (необязательно)

Для частых операций есть удобные методы на `self` — чтобы не писать `GetComponent` каждый раз:

```lua
self:SetText(myLabel, "текст")    -- установить .text (TMP/UI.Text)
self:GetText(myLabel)             -- прочитать .text
self:SetActive(myLabel, false)    -- показать/скрыть объект
```

Аргумент — GameObject напрямую (инъекция) или строка (имя инъекции, backward-compat).

### Кнопки → Lua (LuaUIButton)

На кнопку вешается компонент `LuaUIButton`. Без кода — всё в инспекторе:
- **Function** = имя Lua-функции (`"onChange"`)
- **Argument** = строковый аргумент (`"1"`)
- **Target** = какой `LuaBehaviour` дёрнуть (пусто → ищет в родителях)

```lua
function onChange(arg)
    count = count + (tonumber(arg) or 0)
end
```

---

## 8. Доступ к Unity API из Lua (xLua)

Платформа открывает через xLua доступ к ~120 Unity-типам. Из Lua можно работать с ними **как в C#** — через `GetComponent`, прямой доступ к свойствам, вызов методов.

### Как это работает

```lua
-- Получить компонент по имени типа (строка):
local rb = myObj:GetComponent("Rigidbody")
rb.mass = 5.0
rb.useGravity = true
rb:AddForce(CS.UnityEngine.Vector3(0, 10, 0))

-- TMP-текст:
local tmp = labelObj:GetComponent("TextMeshProUGUI")
tmp.text = "Hello"
tmp.fontSize = 24

-- Image (UI):
local img = healthBarObj:GetComponent("Image")
img.fillAmount = 0.75
```

### Доступные типы

**Unity Core:** `GameObject`, `Component`, `Transform`, `RectTransform`, `Time`

**Physics:** `Rigidbody`, `Rigidbody2D`, `BoxCollider`, `SphereCollider`, `CapsuleCollider`, `MeshCollider`, `CharacterController`, `HingeJoint`, `FixedJoint`, `SpringJoint`, `ConfigurableJoint`, `PhysicMaterial`, а также 2D-физика (`BoxCollider2D`, `CircleCollider2D` и т.д.)

**Rendering:** `Camera`, `Light`, `Renderer`, `MeshRenderer`, `SkinnedMeshRenderer`, `SpriteRenderer`, `MeshFilter`, `LineRenderer`, `TrailRenderer`, `ParticleSystem`, `Shader`, `Texture2D`, `Sprite`

**UI (UnityEngine.UI):** `Canvas`, `CanvasGroup`, `Image`, `RawImage`, `Button`, `Toggle`, `Slider`, `Scrollbar`, `ScrollRect`, `InputField`, `Text`, `LayoutGroup`, `HorizontalLayoutGroup`, `VerticalLayoutGroup`, `GridLayoutGroup`, `ContentSizeFitter`, `LayoutElement`, `Mask`, `RectMask2D`

**TextMeshPro:** `TMP_Text`, `TextMeshPro`, `TextMeshProUGUI`, `TMP_InputField`

**Audio:** `AudioClip`, `AudioListener` (для звука используй `EMRAPI.Audio`)

**Animation:** `Animator`, `Animation`, `AnimationClip`, `RuntimeAnimatorController`

**Math / Structs:** `Vector2`, `Vector3`, `Vector4`, `Quaternion`, `Color`, `Color32`, `Rect`, `Bounds`, `Ray`, `RaycastHit`, `Matrix4x4`, `Mathf`, `Physics` (для Raycast и т.д.)

**Другое:** `Screen`, `Application`, `PlayerPrefs`, `Resources`

### Важные ограничения

- **`Material` не экспортирован** напрямую (слишком много editor-only API). Для цвета используй `EMRAPI.Scene:SetColor()` или доступ через `renderer.sharedMaterial`.
- **`AudioSource` не экспортирован** (проблемы с перегрузками на iOS). Для звука используй `EMRAPI.Audio`.
- Для создания Unity-объектов из Lua есть `CS.UnityEngine.Vector3(x,y,z)`, `CS.UnityEngine.Color(r,g,b,a)` и т.д.
- После любых изменений в списке типов необходимо запустить `XLua → Generate Code`.

### Примечание по `CS.UnityEngine`

Для создания экземпляров Unity-структур из Lua используется синтаксис xLua:

```lua
local pos = CS.UnityEngine.Vector3(1, 2, 3)
local red = CS.UnityEngine.Color(1, 0, 0, 1)
local rot = CS.UnityEngine.Quaternion.Euler(0, 90, 0)
```

Это работает **только в Unity-режиме (LuaBehaviour)**, где xLua предоставляет полный доступ к C#-типам. В чистом Lua-режиме используй `EMRAPI.Scene` / `EMRAPI.UI` — они принимают числа (float), а не Vector3.

---

## 9. Два жизненных цикла — сводная таблица

| | Чистый Lua (`type: "lua"`) | Unity SDK (`LuaBehaviour`) |
|---|---|---|
| **Инициализация** | `function OnStart()` | `function start()` |
| **Каждый кадр** | `function OnUpdate(dt)` | `function update()` — без `dt`, используй `EMRAPI.System:GetDeltaTime()` |
| **Закрытие** | `function OnDestroy()` | `function onDestroy()` |
| **Пауза** | `function OnPause()` / `OnResume()` | `function onDisable()` / `onEnable()` |
| **Физика** | — (используй `EMRAPI.Timer:Repeat`) | `function fixedUpdate()` |
| **UI** | `EMRAPI.UI:CreateButton(...)` | `LuaUIButton` компонент в Unity |
| **3D объекты** | `EMRAPI.Scene:CreatePrimitive(...)` | Визуально в Unity + `EMRAPI.Scene` для динамики |
| **Переменные** | Глобальные | `self`, `gameObject`, `transform` + инъекции |

---

## 10. Упаковка и публикация (Telegram-бот)

Весь процесс сборки `.emr` пакетов и публикации в магазин выполняется через **официального Telegram-бота**.

### Процесс публикации

```
1. Разработчик упаковывает папку приложения в .zip
2. Отправляет .zip боту командой /upload
3. Бот расшифровывает, валидирует app.json
4. Для type:"bundle" — спрашивает платформу (iOS / Android / Обе)
   Для type:"lua" — автоматически ставит для обеих платформ
5. Бот просит иконку (PNG 256×256) и превью (PNG, 16:9)
6. Бот собирает зашифрованный .emr, загружает на хостинг
7. Обновляет repo.json — приложение появляется в магазине
```

### Команды бота

| Команда | Описание |
|---|---|
| `/upload` | Загрузить новое или обновить существующее приложение |
| `/remove <app_id>` | Удалить приложение из каталога |
| `/list` | Показать все приложения в репозитории |
| `/rebuild` | Пересобрать `repo.json` |

### Для Lua-приложений vs Unity-приложений

| | Lua-режим | Unity SDK |
|---|---|---|
| **Что отправляешь** | ZIP с исходниками | ZIP со сбилженным AssetBundle |
| **Платформы** | Автоматически для обеих | Бот спросит: iOS / Android / Обе |
| **Если "Обе"** | Один файл | Два `.emr` (по одному на платформу) |

### Безопасность

Unity SDK **намеренно не содержит** ключи шифрования и схему шифрования. Все криптографические операции выполняются на сервере Telegram-бота. Это исключает утечку ключей через SDK.

---

## 11. Экспорт SDK другим разработчикам

`Assets → Export Package…` → выбери `Assets/ETMR/AppSDK`
(папку `DevApp` — с примерами — можно включить опционально).

Полученный `.unitypackage` импортируется в любой Unity-проект с XLua.

**После импорта** обязательно запусти `XLua → Generate Code` — иначе `GetComponent` и прямой доступ к свойствам не будут работать на устройстве (IL2CPP/AOT ограничения).
