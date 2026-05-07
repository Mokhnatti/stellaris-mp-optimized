# Инструкция для ХОСТА Stellaris MP Optimized

**Твоё железо:** i7-9700K @ 3.6 GHz, 24 GB DDR4-2400, CBR 1TB SSD, Windows 10
**Твоя роль:** ХОСТ партии (lockstep MP правило — самый слабый CPU)
**Партия:** суббота, 3 игрока (хост в РФ + Рамиль в Тае + 2-й в РФ)

---

## ⚠️ ВАЖНО ПРОЧИТАТЬ

1. Не запускай Steam перед тем как сделать BIOS-настройки (пункт 1 ниже)
2. После установки модов — **НИЧЕГО не обновлять до конца кампании**
3. В субботу выходи в Discord минимум за 30 мин до партии — могут быть нюансы

---

## ШАГ 1 — BIOS (5 минут, требует ребут)

**Цель:** включить XMP для оперативки. Без этого RAM работает на 2133 МГц вместо 2400, теряем ~15% производительности.

1. Перезагрузка ПК
2. Заходи в BIOS (обычно **Del** или **F2** при загрузке)
3. Найди раздел **Memory / DRAM / OC / AI Tweaker** (зависит от мат.платы)
4. Найди **XMP Profile** или **DOCP** или **A-XMP**
5. Включи (Profile 1)
6. **Save & Exit** (F10)
7. Windows загрузится с RAM на 2400 MHz

**Проверка после ребута:**
- Win+R → `taskmgr` → Performance → Memory → Speed: должно быть **2400 MHz** (если 2133 — XMP не включился)

Если в BIOS нет XMP — твоя мат.плата может не поддерживать. Не критично, теряем ~15%, продолжаем.

---

## ШАГ 2 — Подписка на коллекцию модов (3 минуты)

1. Открой ссылку:
   **https://steamcommunity.com/sharedfiles/filedetails/?id=3719769152**

2. Залогинься в Steam если не залогинен

3. На странице коллекции найди кнопку **«Subscribe to all»** / **«Подписаться на все»** — нажми

4. Steam начнёт скачивать **25 модов** (~1.5 ГБ) в `Steam\steamapps\workshop\content\281990\`

5. Проверь что скачалось:
   - Steam → Library → Stellaris → правый клик → Properties → Local Files → **Verify integrity**

---

## ШАГ 3 — Tailscale (5 минут)

**Что это:** виртуальная локалка через WireGuard для стабильного MP без Steam SDR лагов.

1. Тебе придёт **invite от Рамиля** на email
2. Открой invite → **Accept** → залогинься на tailscale.com (Google login проще всего)
3. Скачай клиент: **https://tailscale.com/download/windows**
4. Установи, запусти
5. В клиенте залогинься тем же email что в invite
6. После логина в трее (рядом с часами) — иконка Tailscale зелёная = подключён
7. Узнай свой Tailnet IP:
   - Правый клик на иконку Tailscale → **This device** → копируй `100.x.x.x`
   - **Скинь IP в Discord** — Рамилю / 2-му другу

### Если в РФ Tailscale не пускает залогиниться

`login.tailscale.com` иногда блочит РКН. Решение:
- Включи **Cloudflare WARP** (бесплатно, скачай с **1.1.1.1**) — на 2 минуты для логина
- После логина WARP **выключи** — Tailscale работает напрямую через свои DERP-relays Singapore

---

## ШАГ 4 — Windows 10 настройки (10 минут)

### Game Mode выключить (Win 10)
- Settings → Gaming → **Game Mode** = OFF

### Game Bar / DVR выключить
- Settings → Gaming → Xbox Game Bar = OFF
- Settings → Gaming → Captures → Record what happened = OFF

### Power Plan: High Performance
- Открой Power Options (Win+R → `powercfg.cpl`)
- Выбери **High Performance** (не Balanced)
- Если нет High Performance:
  - Win+R → `cmd` (Run as admin) → `powercfg /duplicatescheme 8c5e7fda-e8bf-4a96-9a85-a6e23a8c635c`
  - Power Options обнови F5

### Defender exclusion (КРИТИЧНО)
- Settings → Update & Security → Windows Security → Virus & threat protection
- Manage settings → **Add or remove exclusions** → Add exclusion → Folder
- Добавь:
  1. `C:\Program Files (x86)\Steam\steamapps\common\Stellaris`
  2. `C:\Users\<твой_user>\Documents\Paradox Interactive\Stellaris`
- + Add exclusion → **Process** → `stellaris.exe`

Без этого Defender скание каждый autosave → freeze 1-2 секунды каждый раз.

### OneDrive — НЕ синкать Documents
- Открой OneDrive в трее → Settings → Backup → Manage Backup
- Если **Documents** включён в backup — выключи
- Если не пользуешься OneDrive — закрой его (правый клик в трее → Quit OneDrive)

---

## ШАГ 5 — Steam настройки (2 минуты)

1. Steam → Settings → **In-Game** → Steam Overlay:
   - Можно оставить ON, но при лагах попробуй OFF

2. Свойства Stellaris:
   - Steam Library → Stellaris → правый клик → Properties
   - **Launch Options** (внизу General tab):
     ```
     -dx11 -randomlog -randomlog_stack=5 -randomlog_frames=3 -gamestatetimer
     ```
   - Включаем DirectX 11 (быстрее) + диагностика OOS

3. Cloud Save выключить:
   - Properties Stellaris → General → **Keep games saves in the Steam Cloud** = **OFF**

---

## ШАГ 6 — Discord (1 минута)

В Discord: User Settings → **Game Activity** / **Игровой оверлей**:
- Найти Stellaris → переключатель Overlay = **OFF**

При лагах в late-game **полностью закрой Discord** на время игры (правый клик в трее → Quit Discord).

---

## ШАГ 7 — NVIDIA / AMD Control Panel (3 минуты)

### NVIDIA
1. Правый клик на рабочем столе → **NVIDIA Control Panel**
2. **Manage 3D settings** → **Program Settings** → Add → выбери `stellaris.exe`
3. Настройки:
   - **Power management mode** = **Prefer maximum performance**
   - **Low Latency Mode** = **Off** (это и есть Reflex/Anti-Lag)
   - **Threaded optimization** = **On**
   - **Vertical sync** = **Off**

### AMD
1. **AMD Software** → **Gaming** → Stellaris (если нет — Add Manually)
2. Настройки:
   - **Anti-Lag** = **Disabled**
   - **Radeon Boost** = **Disabled**
   - **Power efficiency** = **Disabled**

---

## ШАГ 8 — Stellaris settings.txt (2 минуты)

1. Запусти Stellaris хоть раз чтобы файл создался
2. Закрой игру
3. Открой блокнотом:
   ```
   C:\Users\<твой_user>\Documents\Paradox Interactive\Stellaris\settings.txt
   ```
4. Найди и поменяй:
   ```
   vsync=no
   multi_sampling=0
   shadowSize=0
   maxanisotropy=0
   autosave=4
   ```
5. Сохрани, закрой

`autosave=4` = раз в год (вместо раз в месяц = 12 фризов в год меньше).

---

## ШАГ 9 — Paradox Launcher (5 минут)

### Создать Playset

1. Запусти Stellaris (откроется Paradox Launcher)
2. Слева **Игровые наборы** (Playsets) → **+ Создать новый**
3. Имя: **MP Optimized** или **Synkarion Crew**

### Включить ВСЕ 25 модов

Нажми **Включить все** (или вручную проверь что все 25 модов из коллекции включены).

### Поставить ПРАВИЛЬНЫЙ порядок загрузки

Перетащи моды в этот порядок (мышка за левый край ☰):

```
1.  ~~Ariphaos Unofficial Patch (4.2)
2.  ~~Scripted Trigger Undercoat
3.  ~ StarNet AI
4.  ! Casako's Framework & Modmenu
5.  ! Better Performance & Utilities
6.  AI Game Performance Optimisation 4.3
7.  AI will not summon dimensional fleet
8.  Stellaris Stability Patch
9.  Disable Empire Focus
10. UI Overhaul Dynamic
11. Suppress Anomaly Pop-ups
12. StopSpam
13. Precursor Selection
14. No Shipyard Juggernauts
15. Total Assimilation
16. Ultimate Automation 4.3
17. Starbase Strong
18. Gray with animated portrait
19. Tasty Maid: Portraits
20. Tasty Maid: Events
21. Spacefleet Tactica
22. SFT ADD-ON: Marine Frigates for AI
23. SFT ADD-ON: Specialist Frigates
24. zz_ Tasty Maid Static (no animation)
25. zz_ Cetus MP Performance Patch  ← САМЫЙ ПОСЛЕДНИЙ
```

### Проверить checksum

⚠️ **ВНИМАНИЕ:** checksum смотрится **ВНУТРИ ИГРЫ** в главном меню Stellaris, а **НЕ в Paradox Launcher** (там checksum другой и неправильный для MP).

Запусти Stellaris с Playset «Opimizm». В **главном меню Stellaris** будет строка вида:
```
Версия: Cetus v4.3.5 (3e40)
```

Где `3e40` — наш текущий эталон.

У тебя **должен быть точно такой же** `3e40`. Если другой:
- Проверь версию игры (должна быть 4.3.5)
- Проверь что включены **все 29 модов** в правильном порядке
- Проверь набор DLC (должен совпадать с Рамилем)
- Verify Stellaris через Steam → Properties → Local Files

В субботу в лобби MP **снова сверим** — у всех 5 игроков должен быть одинаковый.

### Сохрани Playset

Сделай его активным (зелёная кнопка «Запустить с этим Playset»).

---

## ШАГ 10 — Тестовый запуск (10 минут)

**Цель:** убедиться что игра запускается без ошибок ДО партии в субботу.

1. **Запусти Stellaris** через лаунчер (Playset активен)
2. После загрузки главного меню — выйди в Windows
3. Открой:
   ```
   C:\Users\<твой_user>\Documents\Paradox Interactive\Stellaris\logs\error.log
   ```
4. Если файл маленький (<100 КБ) и нет красного спама про zz_perf_patch / zz_tasty_static / SFT — **отлично**
5. Если ошибки про конкретный мод — скинь Рамилю

**Опциональный тест-сейв:**
- New Game → Small galaxy / 4 AI / любая раса
- Запусти на speed 5 → 10 минут
- Если игра не падает и FPS норм — ты готов

---

## ШАГ 11 — Что делать в СУББОТУ (партия)

### За 30 минут до старта

1. **Закрой всё лишнее**:
   - Браузер (все вкладки)
   - Discord можно оставить, но **overlay для Stellaris OFF**
   - OBS / NVIDIA Shadow Play — закрыть
   - OneDrive — Quit
   - Тяжёлые процессы (видеоредактор, IDE)

2. **Проверь Tailscale**:
   - Иконка в трее зелёная = Connected
   - Если красная — `tailscale up` в PowerShell

3. **Запусти Stellaris** через лаунчер (Playset MP Optimized)

4. **Скинь свой Tailnet IP** в Discord:
   ```
   Мой Tailnet IP: 100.x.x.x
   ```

### Создание лобби (ты как хост)

1. Главное меню → **Multiplayer** → **Host Game**
2. Galaxy settings — **сообщи в Discord что выбираешь, согласуй**
3. Открой лобби, дождись захода Рамиля и 2-го

### В лобби

1. Сравни **checksum** в правом нижнем углу — у всех должен быть одинаковый
2. Если у кого-то другой:
   - Verify Stellaris в Steam Library
   - Проверить Playset Order — у всех одинаковый порядок?
3. Если всё ок — **Start Game**

### Во время игры

- Speed **3-4** в late-game (после 2300) — speed 5 на 9700K может тормозить
- **Не открывай тяжёлые UI окна** во время монтлый-tick (мигает в углу)
- При спайке лагов — пауза 30 сек, затем продолжайте
- Autosave раз в год — это норма (1-2 секунды фриз)

---

## ПРОБЛЕМЫ И РЕШЕНИЯ

### Игра не запускается / падает на загрузке
- Verify Stellaris в Steam
- Удалить Playset, создать заново
- Проверить error.log на конкретную ошибку

### Тормозит на speed 5
- Понизь до **speed 4** или **speed 3**
- Закрой Discord полностью
- Закрой лишние процессы

### Desync (OOS) у одного из игроков
1. Все → **Main Menu** (НЕ desktop)
2. Хост грузит **autosave** (не manual save)
3. Все возвращаются
4. Если повторяется — удалите `Documents\Paradox Interactive\Stellaris\oos\` у всех

### Не могу подключить друзей
- Через Steam Friends: убедись что они в Friends
- Через Tailscale Direct IP: дай им свой `100.x.x.x` → они вводят в "Direct connect by IP" в MP меню

### CTD (Crash To Desktop) посреди игры
- В late-game (после 2400 года) бывает у всех — Crisis spike
- Грузим autosave −1 год
- Если повторяется — `kill_country` мёртвых AI через консоль (Shift+Alt+C)

### Тики (debug_stats) > 300 ms в late-game
- Speed понизь до 3
- Через консоль:
  ```
  kill_country [id]
  ```
  Убей 2-3 weakest AI → −30-50% от тика

---

## КОНТАКТЫ

- **Рамиль (Тай)** — основной разработчик сборки, по любым проблемам пиши ему
- **Discord** — голос/чат во время партии
- **Tailnet** — все 3 в одной mesh-сети `100.x.x.x`

---

## ИТОГО ПОДГОТОВКА (1 раз перед субботой)

- [ ] BIOS XMP включён
- [ ] Steam коллекция подписана (29 модов)
- [ ] Tailscale установлен и подключён (опционально)
- [ ] Win 10: Game Mode OFF, Power Plan = High Performance, Defender exclusion, OneDrive не синкает
- [ ] Steam Launch Options для Stellaris с `-dx11 -randomlog ...`
- [ ] Discord overlay для Stellaris OFF
- [ ] NVIDIA/AMD: Maximum Performance для stellaris.exe
- [ ] settings.txt: vsync=no, autosave=4
- [ ] Playset «Opimizm» создан, 29 модов, порядок правильный, checksum в игре сверен
- [ ] Тестовый запуск прошёл без ошибок

После этого ты готов к партии. **В субботу — закрыл лишнее → запустил → создал лобби → погнали.**
