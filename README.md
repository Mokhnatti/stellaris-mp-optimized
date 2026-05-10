# Stellaris MP Optimized — Synkarion Crew

Приватная сборка модов для Stellaris 4.3 «Cetus» — субботняя партия 5 игроков RU↔TH.

🌐 **Сайт:** https://mokhnatti.github.io/stellaris-mp-optimized/

## Быстро

1. Подписаться на коллекцию: https://steamcommunity.com/sharedfiles/filedetails/?id=3719769152
2. Скачать [Opimizm.json](Opimizm.json) — импортировать в Paradox Launcher (27 модов в правильном порядке)
3. Сверить checksum **внутри игры** в главном меню Stellaris (НЕ в лаунчере!)
4. Подробный гайд для хоста: [HOST_GUIDE.md](HOST_GUIDE.md)

## Состав (27 модов)

- 11 геймплейных модов (UI Overhaul, Tasty Maid Portraits, SFT + 2 аддона, Starbase Strong, Total Assimilation и т.д.)
- 9 perf-модов (General Fixes, Stability Patch, Scripted Trigger Undercoat, AIGPO, Casako, BPU, StarNet AI, Disable Empire Focus, AI No Dim Fleet)
- 1 русификатор (Tasty Maid Portraits Russifier)
- 3 наших кастомных мода: zz_ Cetus MP Performance Patch, zz_ Tasty Maid Static, zz_ RU Pack - Synkarion Crew
- 2 утилиты: Precursor Selection, No Shipyard Juggernauts, StopSpam, Suppress Anomaly Pop-ups, Ultimate Automation

## Что изменилось 09.05 (за день до партии)

**Удалено 4 мода** на основе ночного теста + Research:
- ❌ Ariphaos Unofficial Patch (4.2) — конфликт с General Fixes (#0) который и есть его обновление
- ❌ Tasty Maid: Events — не обновлён под 4.3, **OOS-риск** (random_list блоки)
- ❌ Ariphaos UP RUS — русик удалённого мода
- ❌ Tasty Maid: Events RU — русик удалённого мода

**Изменён порядок загрузки** — Better Performance & Utilities теперь **в самом низу** (overrides 00_on_actions.txt, должен последний).

## House Rules — ОБЯЗАТЕЛЬНО

1. **Tutorial = NONE** у всех 5 игроков (без этого крах на First Contact)
2. **No uplift до 2300**
3. **Steam Cloud Sync OFF** для Stellaris
4. **Speed cap 4** для всей партии
5. **Hard pause при любом диалоге у любого**
6. Полный список — в [HOST_GUIDE.md](HOST_GUIDE.md)

## Launch Options

**Хост:**
```
-nolauncher -nakama -randomlog -randomlog_stack=5 -randomlog_frames=3
```

**Клиенты:**
```
-nolauncher -nakama
```

`-nakama` критичен с Tailscale (лечит "infinite connecting").

Версия Stellaris: **Cetus v4.3.5**, эталон checksum: **fd4d** (после удаления 4 модов checksum изменится — сверим в Discord перед партией).
