# Cloud Stellaris из Тайланда — итоговый гайд

**Цель:** играть с друзьями в РФ из Бангкока без проблемы пинга. Решение — запустить Stellaris на сервере, к которому **друзья из РФ имеют пинг 5-30мс**, а ты стримишь к себе в Тай (видеопоток 200-280мс — для пошаговой стратегии терпимо).

---

## 🎯 TL;DR — что брать

| Приоритет | Решение | Цена/мес | Когда выбирать |
|-----------|---------|----------|----------------|
| 🥇 **Лучший** | **immers.cloud (Москва) + Apollo** | $30-37 | Контроль + лучший пинг друзьям (5-30мс) |
| 🥈 **Простой** | **TensorDock (Frankfurt) + Sunshine** | $15-18 | Дёшево, без RU-привязки. Друзьям 30-50мс |
| 🥉 **Готовое** | **VK Play Cloud Comfort + ВМ** | $27 | Без настройки, но нужен RU-VPN для оплаты |
| 💡 **Каталожное** | **GFN.AM Performance** | $13 | Дёшево но риск с кастомными империями/модами |

---

## Полное сравнение всех вариантов

### ✅ Работающие

| Сервис | Цена 30ч/мес | Сервер | Моды | Пинг друзьям | Удобство |
|--------|--------------|--------|------|--------------|----------|
| **immers.cloud RTX 3080** | ~$37 | Москва | ✅ полный Windows | **5-30мс** | Средне |
| **immers.cloud T4** | ~$28 | Москва | ✅ полный Windows | **5-30мс** | Средне |
| **VK Play Cloud Comfort + Хранилище** | $27 | Москва/СПб/Екб | ✅ через ВМ | **5-30мс** | Высокое |
| **МТС Fog Play** | $8-15 | Москва/СПб/Новосиб/Владивосток | ✅ свой Steam | **5-30мс** | Высокое |
| **TensorDock RTX 4000** | $15-18 | Frankfurt/Helsinki/Stockholm | ✅ полный Windows | 30-50мс | Средне |
| **airgpu** | $26.50 | Frankfurt/Amsterdam/Stockholm | ✅ полный Windows | 30-50мс | **Очень высокое** |
| **GFN.AM Performance** | $13 | Хельсинки/Стокгольм/Frankfurt | ⚠️ Workshop ОК, империи спорно | 30-80мс | Высокое |
| **Hetzner GEX44** | $200 (€184/мес flat) | Helsinki | ✅ BYOL Windows | 15-25мс | Низкое |
| **Shadow PC Neo** | $33-45 | Париж/Амстердам | ✅ полный Windows | 50-70мс | Высокое |
| **Selectel T4 spot** | $8-13 | Москва/СПб | ✅ BYOL Windows | 5-30мс | Низкое |
| **Loudplay** | $11-22 | Москва | ✅ свой Steam | 5-30мс | Среднее |

### ❌ НЕ работают

| Сервис | Почему не подходит |
|--------|-------------------|
| **Boosteroid** | Моды стираются между сессиями (Help Center подтверждает) |
| **GeForce NOW** | Нет Workshop модов в полном виде, кастомные империи теряются |
| **Paperspace** | С июля 2024 закрыли Windows-templates для новых аккаунтов |
| **Vultr Cloud GPU** | Остановленный инстанс продолжает биллиться + RU-карты блочат |
| **Yandex Cloud GPU** | TCC compute-режим, десктопный рендер не работает |
| **RunPod / Lambda / Genesis** | Linux-only, нет /dev/kvm для Windows VM |
| **AWS** | Требует юрлицо, дорогой Win Spot |
| **GFN.RU** | Закрылся 1 октября 2023 |

---

## 🏆 Топ-3 решения с пошаговыми инструкциями

### 🥇 immers.cloud + Apollo (рекомендуется)

**Цена: $30-37/мес** при 32 часах игры. Сервер в Москве = пинг друзьям 5-30мс.

**Шаги:**

1. **Регистрация:** https://en.immers.cloud/
2. **Пополнить баланс на 3000₽** через USDT-мост (Bitkub Тай → USDT TRC20 → Bybit P2P → МИР через СБП)
3. **Deploy VM:**
   - Образ: **«Для игр» Windows Server 2022** (есть в маркетплейсе)
   - Конфиг: `rtx3080-1.8.16.40` (8 vCPU, 16 ГБ RAM, RTX 3080) — **38₽/час** при работе
   - Дополнительный Volume SSD 100 ГБ — **792₽/мес** (в режиме shelve)
4. **Открыть порты в Security Group:**
   - TCP: 47984, 47989, 48010
   - UDP: 47998-48010
   - RDP 3389 (только на этап настройки — потом закрыть!)
5. **Зайти по RDP, обновить драйвер NVIDIA Game Ready**
6. **Установить:**
   - **VB-Audio Virtual Cable** (https://vb-audio.com/Cable/) — обязательно, у VPS нет аудио
   - **Apollo** (https://github.com/ClassicOldSong/Apollo) — встроенный virtual display
   - **Steam → Stellaris → 26 модов из Opimizm плейсета**
   - Перенести Steam-библиотеку на диск D:
7. **На клиенте в Тае: Moonlight Qt** (https://moonlight-stream.org/)
8. **Между сессиями:** `Shelve` VM в панели immers.cloud → платится только диск (~26₽/сутки)

**Apollo конфиг:**
```
encoder = nvenc
nvenc_preset = p1 (lowest latency)
nvenc_tune = ll
rc = cbr
fec_percentage = 30
nvenc_latency_over_power = enabled
```

**Moonlight настройки:**
- 1920×1080 @ 60 FPS
- HEVC, 20 Mbps
- 4:4:4 chroma (важно для UI текста)
- V-Sync OFF, frame pacing OFF
- Reduce frame buffering ON
- HDR OFF

---

### 🥈 TensorDock + Sunshine (бюджетный без RU)

**Цена: $15-18/мес** при 30 часах. Сервер в Европе = пинг друзьям 30-50мс.

**Шаги:**

1. **Регистрация:** https://dashboard.tensordock.com/deploy
2. **Депозит $20** через USDT TRC20 (или Visa)
3. **Deploy VM:**
   - Фильтр: Europe (Frankfurt / Amsterdam / Helsinki / Stockholm)
   - GPU: **RTX 4000 SFF Ada** или **A4000**
   - ОС: Windows 10 / Server 2022
   - Disk: 100 GB SSD
   - Цена: ~$0.30-0.40/час running, **$0.005/час stopped** (хранение!)
4. **Установить тот же стек** что и в immers.cloud (Apollo/Sunshine + Steam + моды)
5. **API-скрипт автостопа** (после 30 мин idle → Stop VM → платится только диск)

**Расчёт:** 30ч × $0.40 + 700ч × $0.005 = $12 + $3.50 ≈ **$15.50/мес**

---

### 🥉 VK Play Cloud Comfort (готовое РФ-решение)

**Цена: $27/мес** (1999₽ Comfort + 499₽ Хранилище ВМ).

**Шаги:**

1. **Включить VPN с РФ-IP** (поднять AmneziaWG на собственном RU-VPS за 200₽/мес)
2. **VK ID на email** → пополнить МИР через USDT-мост
3. **Оплатить тариф Comfort + Хранилище** на vkplay.ru
4. **Запустить «Виртуальный компьютер»** (выбрать ДЦ Москва)
5. **Войти в Steam → установить Stellaris + моды** (Хранилище ВМ сохранит после сессии)
6. Подключение через **десктопный клиент VK Play или браузер**

**Лимиты Comfort:**
- 60 часов днём (доплата 60₽/час за переработку)
- Безлимит ночью 00:00-08:00 МСК (= 04:00-12:00 в Бангкоке — удачное окно!)

**Минусы:**
- Task Manager заблокирован
- VPN внутри ВМ запрещён
- В 2026 VK Play режет доступ из-под зарубежных VPN — нужен **реальный РФ IP**

---

## 💳 Платежи из Тайланда

**Wise → РФ закрыт навсегда.** Стандарт для русских в Тае:

### Главный путь — USDT мост

```
1. Тайская биржа (Bitkub / Bitazza / Binance TH) — покупаешь USDT за тайские
   баты с банка Bangkok Bank/SCB/Kasikorn
2. Перевод USDT TRC20 (~$1 комиссия) или TON (~$0.03) на Bybit/HTX
3. P2P за RUB через СБП на МИР-карту
4. Из МИР оплачиваешь иностранный/российский сервис

Суммарная комиссия: 1-3% + спред 1-2%
```

### По провайдерам:

| Провайдер | Принимает |
|-----------|-----------|
| **immers.cloud, Selectel, Timeweb, Yandex Cloud, VK Play, GFN.RU/AM** | МИР (с РФ-VPN при оплате) |
| **TensorDock** | Visa + USDT прямо |
| **airgpu, Shadow** | Visa международная |
| **МТС Fog Play** | СБП + МИР, **требует RU-номер** для регистрации |
| **Hetzner** | Visa международная (но сложный KYC с РФ-паспортом) |

---

## 🛠️ Технический стек стрима

### Что устанавливать на сервер:

1. **Apollo** (https://github.com/ClassicOldSong/Apollo) — лучше Sunshine для VPS
   - Встроенный virtual display (без HDMI-dongle)
   - Поддержка HDR
2. **VB-Audio Virtual Cable** (https://vb-audio.com/Cable/) — обязательно для звука
3. **Stellaris в режиме Borderless Windowed** (не Fullscreen Exclusive — лучше захват)
4. **RTSS framerate cap 60 FPS**

### Что устанавливать на клиент в Тае:

1. **Moonlight Qt** (https://moonlight-stream.org/) — основной клиент
2. **Подключение по публичному IP сервера** (не через Tailscale/VPN)

### НЕ использовать:

- ❌ **Parsec** в РФ нестабилен в 2025-2026 (ошибки -6101/-6023)
- ❌ **Steam Remote Play** — баг с Paradox Launcher (чёрный экран)
- ❌ **RDP во время стрима** — отбирает NVENC сессию у Apollo
- ❌ **Tailscale через DERP** — добавляет +30-80мс
- ❌ **Bluetooth наушники** — добавляют 80-250мс

### Сетевая обвязка:

- **Прямой UDP по публичному IP** + Windows Firewall whitelist твоего тайского IP
- **На роутере в Тае: SQM/fq_codel** (ASUS Merlin или OpenWrt) — главное против bufferbloat
- **Ethernet, не Wi-Fi**
- **Проводные/USB наушники**, не Bluetooth
- **Провайдер в Тае: AIS Fibre / 3BB Fibre3** (лучше для международного трафика)

---

## ⚠️ Подводные камни

### Технические:

| Проблема | Решение |
|----------|---------|
| **A100/H100 без NVENC** | Брать только T4, A2, L4, A10, RTX 3060+ (для гейминга) |
| **Yandex Cloud TCC mode** | Не использовать для гейминга |
| **Cloud-VM без аудио** | Установить VB-Audio Virtual Cable |
| **Linux VPS не запустит Stellaris** | Проверять что есть Windows в маркетплейсе |
| **Vultr биллит остановленный инстанс** | Делать `destroy + snapshot` после сессии |
| **RDP отбирает NVENC сессию** | Админить через VNC или web-UI Apollo |

### Геймплейные:

| Проблема | Решение |
|----------|---------|
| Все игроки должны иметь **одинаковый набор и порядок модов** | Использовать наш плейсет Opimizm 26 модов |
| **Stability Patch + Disable Empire Focus обязательны** | Уже в нашем плейсете |
| Cloud-CPU слабее десктопного X3D | Снизить размер галактики до Medium 600 stars |
| Custom empires/конфиги в `Documents\Paradox Interactive` | На GFN — нужен Persistent Storage. На полном Windows VM — работает само |

### Регистрация/оплата:

| Сервис | Подводный камень |
|--------|------------------|
| **МТС Fog Play** | Требует РФ-номер для СМС, в роуминге СМС не всегда доходят |
| **VK Play Cloud** | В 2026 блокирует доступ из-под зарубежных VPN — нужен реальный РФ-IP |
| **GFN.AM** | При первом входе нужен VPN на Армению/Казахстан |
| **Steam антифрод** | Настроить Steam Guard Mobile Authenticator перед сменой региона |

---

## 🚀 Что делать прямо сейчас (3 шага)

### Шаг 1 — тест маршрута (15 минут, ~150₽)

Возьми тестовый VPS на immers.cloud на 2-3 часа. Прогоняешь:

```
mtr -u -c 100 <твой_тайский_IP>  # с сервера до тебя
mtr -u -c 100 <IP_сервера>        # с домашнего компа до сервера
```

Проверяешь что маршрут идёт через **RETN/TransTelecom** (быстро), а не через Cogent/европейский транзит.

### Шаг 2 — выбор сервиса

- Если маршрут хорош (RTT < 220мс) → **immers.cloud** = лучший по пингу
- Если маршрут плох → **TensorDock Frankfurt** (без RU-привязки)
- Если хочешь zero-setup → **airgpu** (Parsec/Moonlight предустановлены)
- Если совсем дёшево → **GFN.AM** (но риск с кастомными империями)

### Шаг 3 — настройка (2 часа, один раз)

1. Регистрация выбранного провайдера
2. Пополнение баланса (USDT-мост или Visa)
3. Deploy VM с Windows + GPU + Frankfurt/Москва
4. Установка: Apollo + VB-Cable + Steam + Stellaris + 26 модов из Opimizm
5. Snapshot после установки (на случай сбоев)
6. Тестовая партия 30 минут одиночная — проверить FPS, задержку
7. Готовы к субботней партии

---

## 📊 Ожидаемый результат

### Сейчас (через Tailscale напрямую):
- Пинг хосту в РФ: **228мс**
- Lockstep MP скорость: ограничена пингом → 10-15 лет/час в late game
- Партия до 2400: 8-12 часов

### С cloud-сервером в Москве:
- Стрим до меня в Тае: 200-260мс motion-to-photon (не мешает stratagey)
- Lockstep MP: **5-30мс между всеми** (как локалка!)
- Скорость: 30-40 лет/час даже в late crisis
- Партия до 2400: **5-7 часов**

**Архитектурный сдвиг:** ты перестал бороться с lockstep через сетевую оптимизацию и перенёс «настоящего игрока» в Москву. Канал Тай↔Москва несёт только видео, а игровая симуляция живёт в правильной сети.

---

## 🔗 Ключевые ссылки

- **immers.cloud:** https://en.immers.cloud/prices/
- **TensorDock:** https://dashboard.tensordock.com/
- **VK Play Cloud:** https://cloud.vkplay.ru/
- **Apollo (рек):** https://github.com/ClassicOldSong/Apollo
- **Sunshine:** https://github.com/LizardByte/Sunshine
- **Moonlight:** https://moonlight-stream.org/
- **VB-Audio Cable:** https://vb-audio.com/Cable/
- **NVENC matrix (без A100/H100!):** https://developer.nvidia.com/video-encode-and-decode-gpu-support-matrix-new
- **Bufferbloat тест:** https://www.waveform.com/tools/bufferbloat
- **Stellaris Stability Patch:** https://steamcommunity.com/sharedfiles/filedetails/?id=3479537780
- **Disable Empire Focus:** https://steamcommunity.com/sharedfiles/filedetails/?id=3489022357

---

## Финальная рекомендация

**Бери immers.cloud + Apollo**, плати ~$30-37/мес.

**Почему:**
1. Сервер в Москве = пинг друзьям 5-30мс (lockstep как локалка)
2. Полный Windows = моды/сейвы работают
3. NVENC доступен (RTX 3080)
4. Биллинг посекундный + бесплатный shelve между сессиями
5. Платёж через USDT TRC20 / МИР (есть рабочая схема)
6. Готовый образ «Для игр» в маркетплейсе

**Время на настройку:** 2 часа разово.
**После:** каждую субботу — Start VM (1 минута) → играем → Shelve (платится только диск).

К следующей субботе будет работающий сетап. Партия будет в 2-3 раза быстрее текущей.
