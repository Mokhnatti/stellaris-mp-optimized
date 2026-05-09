# Stellaris cloud-VM tuning — выжать максимум из cloud-Xeon

Полный research-отчёт по оптимизации single-thread производительности cloud-Xeon Cascadelake для Stellaris MP.

## TL;DR

**Cloud-Xeon Gold 6240R (Cascadelake) ≈ 1000-1100 Cinebench R23 ST.** Десктопный 9700K ≈ 1300-1500. Разница 30-50% = в lockstep MP cloud-клиент тормозит партию.

**Tweak-ами реально выжать +15-30%** = паритет с 9700K. Дальше — миграция на Hetzner AX102 (Ryzen 7950X3D) = +2.7-3x.

---

## A. Быстрые твики (5-10 минут, без reboot)

### A.1. Зафиксировать baseline

```powershell
Get-WmiObject Win32_Processor | Format-List Name,NumberOfCores,NumberOfLogicalProcessors,MaxClockSpeed,Stepping,L3CacheSize
wmic cpu get Name,ProcessorId,NumberOfCores,NumberOfLogicalProcessors
```

Stepping 4 = Skylake-SP (mitigations off даёт МНОГО)
Stepping 6/7 = Cascadelake (mitigations уже в железе)

Скачать **Coreinfo** (Sysinternals): `Coreinfo64.exe -c -accepteula`
Если HT виден как `**------ / --**---- / ----**-- / ------**` — affinity на чётные ядра даёт +5-15%.

### A.2. Ultimate Performance + High Priority + Affinity

```powershell
$out = powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61
$guid = ([regex]::Match($out,'[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}')).Value
powercfg /setactive $guid
powercfg /setacvalueindex SCHEME_CURRENT SUB_PROCESSOR PROCTHROTTLEMIN 100
powercfg /setacvalueindex SCHEME_CURRENT SUB_PROCESSOR PROCTHROTTLEMAX 100
powercfg /setacvalueindex SCHEME_CURRENT SUB_PROCESSOR IDLEDISABLE 1
powercfg /setactive SCHEME_CURRENT

$p = Get-Process stellaris -ErrorAction Stop
$p.PriorityClass = 'High'
$p.ProcessorAffinity = [IntPtr]0x55  # vCPU 0,2,4,6 — физические если HT виден
```

⚠️ НЕ Realtime — может зависнуть Windows.

### A.3. Defender exclusions

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableScriptScanning $true
Set-MpPreference -DisableBehaviorMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Add-MpPreference -ExclusionProcess "stellaris.exe","steam.exe","steamwebhelper.exe"
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam","$env:USERPROFILE\Documents\Paradox Interactive\Stellaris"
Add-MpPreference -ExclusionExtension ".sav",".mod",".pdx"
```

### A.4. Закрыть фоновые сервисы

```powershell
$svcs = 'WSearch','SysMain','BITS','wuauserv','DiagTrack','WerSvc','MapsBroker','PcaSvc','DPS'
foreach ($s in $svcs) {
    Stop-Service $s -Force -ErrorAction SilentlyContinue
    Set-Service  $s -StartupType Disabled -ErrorAction SilentlyContinue
}
```

⚠️ НЕ трогать TermService, RemoteRegistry, WinRM, NlaSvc, Dnscache (RDP отвалится).

### A.5. Steam Launch Options

```
-dx11 -threads=4 --skip-intro
```

`-threads=` = число **физических** ядер (не vCPU).

### A.6. settings.txt — `refreshCap=30` (самый недооценённый фикс)

`%USERPROFILE%\Documents\Paradox Interactive\Stellaris\settings.txt`:

```
shadowSize=0
maxanisotropy=0
multi_sampling=0
vsync=no
refreshCap=30
```

Paradox dev diary 4.3.6: *"increasing refreshCap leaves less time for the simulation loop"*. **Меньше FPS = быстрее тики.**

### A.7. In-game graphics на минимум

Multisampling 0, Shadow Low, Galaxy Low, Bloom Off, Lens flare Off.

### A.8. Steam settings

- Library → Stellaris → Properties → uncheck Steam Overlay
- Settings → Downloads → Region: Russia/Moscow
- Settings → In-Game → Server Browser Pings: 250

### A.9. После рестарта Stellaris — повторить A.2

Priority/affinity сбрасываются при перезапуске процесса.

**Ожидаемый эффект A.1-A.9: +10-20%**

---

## B. Среднесрочные твики (1-2 часа, требуют reboot)

### B.1. Отключение CPU mitigations

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" /v FeatureSettingsOverride     /t REG_DWORD /d 3 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" /v FeatureSettingsOverrideMask /t REG_DWORD /d 3 /f
shutdown /r /t 0
```

После reboot проверить:

```powershell
Set-ExecutionPolicy -Scope Process RemoteSigned -Force
Install-Module SpeculationControl -Force -Scope CurrentUser
Import-Module SpeculationControl
Get-SpeculationControlSettings
```

Прирост: Cascadelake +3-10%, Skylake-SP +8-25%.

Альтернатива GUI: **InSpectre.exe** (https://www.grc.com/inspectre.htm).

### B.2. Core Parking + dynamic tick

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\54533251-82be-4824-96c1-47b60b740d00\0cc5b647-c1df-4637-891a-dec35c318583" /v Attributes /t REG_DWORD /d 0 /f
bcdedit /set disabledynamictick yes
```

```powershell
powercfg /setacvalueindex SCHEME_CURRENT SUB_PROCESSOR 0cc5b647-c1df-4637-891a-dec35c318583 100
powercfg /setacvalueindex SCHEME_CURRENT SUB_PROCESSOR bc5038f7-23e0-4960-96da-33abaf5935ec 100
powercfg /setactive SCHEME_CURRENT
```

Прирост: +2-5%.

### B.3. Полное удаление Defender (Server 2022)

```powershell
Uninstall-WindowsFeature -Name Windows-Defender
Restart-Computer
```

Прирост: +5-10% поверх A.3.

### B.4. Process Lasso

https://bitsum.com/ (trial 30 дней = full)
- Affinity → Disable SMT/Hyper-Threading
- Priority Class = High, sticky
- Bitsum Highest Performance plan = ON
- ProBalance = ON

ParkControl: Core Parking 0%, Frequency Scaling 100%.

### B.5. Network adapter tuning

```powershell
$nic = (Get-NetAdapter | Where-Object Status -eq 'Up' | Select-Object -First 1).Name
Set-NetAdapterAdvancedProperty -Name $nic -DisplayName "Interrupt Moderation" -DisplayValue "Disabled" -EA SilentlyContinue
Set-NetAdapterAdvancedProperty -Name $nic -DisplayName "Recv Segment Coalescing (IPv4)" -DisplayValue "Disabled" -EA SilentlyContinue
Set-NetAdapterAdvancedProperty -Name $nic -DisplayName "Energy Efficient Ethernet" -DisplayValue "Disabled" -EA SilentlyContinue
Enable-NetAdapterRss -Name $nic -EA SilentlyContinue
Disable-NetAdapterRsc -Name $nic -EA SilentlyContinue

netsh int tcp set supplemental Internet congestionprovider=ctcp
netsh int tcp set global rss=enabled
netsh int tcp set global rsc=disabled
```

**Network для Stellaris не критичен** — это UDP через Steam SDR, на скорость симуляции не влияет. Делать «для гигиены».

### B.7. Диагностика virt overhead

- **LatencyMon** (https://www.resplendence.com/latencymon) — Highest DPC > 5000µs = есть virt overhead
- **Cinebench R23** SC (https://www.maxon.net/en/cinebench)
  - Bare-metal Xeon 6240R = ~1000-1100 ST
  - Если у тебя <700 — соседи на хосте, писать в support
- **ClockRes** во время игры — должен быть Current = 1.000ms

**Кумулятивный эффект B: +10-15% поверх A.**

---

## C. Stellaris модификации

### C.1. Performance моды для 4.3 Cetus

- **AI Game Performance Optimisation 4.3** (id 2184737698)
- **Better Performance & Utilities** (Kasako, id 2475302050) — +75% game fluency
- **Full Tiny Outliner** (id 2948301103) — снижает UI render
- **Universal Resource Patch** (id 1595876588) — внизу load order

**AI выбрать ОДИН:** StarTech AI > StarNet AI > Glavius

**НЕ ставить:** Gigastructures, NSC2/NSC3, ACOT, Real Space, More Events.

### C.2. Galaxy Setup для performance

| Параметр | Рекомендация |
|----------|--------------|
| Galaxy size | **600 stars** (не 1000) |
| Habitable Worlds | **0.25x** (главное) |
| AI Empires | **6** (5p + 6 AI) |
| Fallen Empires | 1 |
| Marauders | 0 |
| Hyperlane Density | 0.5-0.75x |
| L-Gates / Wormholes / Caravaneers | **OFF** |
| Xeno-Compatibility | OFF |
| Pre-FTL civs | 0.5x |
| Crisis Strength | 1-5x (не 25x!) |
| **Logistic Growth Ceiling** | **1x (min)** |
| **Growth Required Scaling** | **1x (max)** |
| Cosmic Storms | OFF |
| Voidworms / Cuthuloids | 0x |

### C.3. DLC отключаемые

Cosmic Storms, Astral Planes, Grand Archive, опц. First Contact, Galactic Paragons.

---

## D. Когда оптимизация недостаточна — миграция

**Если Cinebench R23 ST < 800** = virt overhead, оптимизация не поможет.

**Сравнение вариантов:**

| Вариант | CPU | R23 ST | Цена/мес | Stellaris speed |
|---------|-----|--------|----------|-----------------|
| immers.cloud Cascadelake (now) | 6240R | ~1000 | текущая | 1.0× |
| immers.cloud RTX 4090 нода | 6336Y Ice Lake | ~1100 | +доплата | ~1.1-1.2× |
| **immers.cloud Emerald Rapids** | 5th gen Xeon | ~1500-1700 | +$30-50 | **~1.5-1.7×** |
| Aeza VPS Москва | Ryzen 9 9950X | ~2200 | $30-60 | ~2.0-2.2× |
| **Hetzner AX102** | **Ryzen 7950X3D** | ~2050 | €148 (~$160) | **~2.7-3.0×** ⭐ |

### D.1. X3D эффект для Stellaris

Документированный xxEzri/Vermeer guide: **+37% на 5800X3D vs 5900X**. Stellaris упирается в L3 cache hit-rate (branchy script-VM, мелкие аллокации). 96 MB L3 на CCD0 у 7950X3D = **лучшее железо для Stellaris в облаке**.

### D.2. Hetzner AX102 — план

1. hetzner.com/dedicated-rootserver/ax102 (Falkenstein или Helsinki)
2. Windows Server 2022 install
3. AMD chipset drivers + Xbox Game Bar + Game Mode (для preferred cores)
4. Принудительно прибить Stellaris к CCD0 (ядра 0-7 = 3D V-Cache):
   `start /affinity FF stellaris.exe`
5. Применить все A+B твики
6. Графика через iGPU 7950X3D или WARP. Stellaris CPU-bound, UI пойдёт.

### D.3. Тикет в immers.cloud (промежуточный)

Telegram @immerscloudsupport:

```
Здравствуйте. Текущая нода (Xeon Gold 6240R Cascadelake) даёт
single-thread производительность ниже наших требований.

1. Возможен ли перевод на ноду с CPU поколения Ice Lake (6336Y, как
   на RTX 4090 нодах) или 5th gen Emerald Rapids?
2. Есть ли тариф с гарантированным dedicated CPU без shared overcommit?
3. Возможен ли NUMA pinning (все 8 vCPU на одном физ. сокете)?
4. Можете прислать host-side метрики "% Total Run Time" для нашего
   VM за последний час, чтобы оценить CPU contention?
```

---

## Дорожная карта

**Сегодня (15 минут):** A.1, A.2, A.3, A.5, A.6, A.7, A.8

**На неделе (2 часа):** B.1, B.3, B.4, B.7 + тикет в support

**Через 2-4 недели:** Если R23 ST < 1000 после всех твиков — мигрировать на Hetzner AX102 или Aeza Ryzen.

---

## Источники

- Phoronix mitigations cost: https://www.phoronix.com/review/zombieload-v2-taa
- InSpectre: https://www.grc.com/inspectre.htm
- LatencyMon: https://www.resplendence.com/latencymon
- Process Lasso: https://bitsum.com/
- Cinebench R23: https://www.maxon.net/en/cinebench
- Coreinfo: https://learn.microsoft.com/sysinternals/downloads/coreinfo
- AMD X3D Stellaris guide: xxEzri/Vermeer
- Paradox dev diary 4.3.6 (refreshCap)
- Hetzner AX102: https://www.hetzner.com/dedicated-rootserver/ax102
