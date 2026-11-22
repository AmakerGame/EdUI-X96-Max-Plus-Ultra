# Архітектура системи оновлення EdUI

## Компоненти

### EdUIInstaller.apk (`com.edui.installer`)

- Звичайний користувацький застосунок для **первинного** встановлення EdUI на стокову SlimBOXTV 49.
- Має іконку в лаунчері, запускається користувачем вручну.
- Потрібен **Root (Magisk)** — запит `su -c id` на старті встановлення.
- Містить ту саму логіку Engine, що й EdUIUpdate, але прошиває виключно через `su`.

### EdUIUpdate.apk (`com.edui.update`)

- **Системний компонент** EdUI: `android:sharedUserId="android.uid.system"`, підписаний платформним тестовим ключем → працює з uid 1000 без Magisk.
- Розташований у `/system_ext/priv-app/EdUIUpdate/` прошивки.
- **Без іконки лаунчера**, без LAUNCHER- категорії — не відображається серед звичайних програм.
- Має дозвіл `DEVICE_POWER` (reboot) і `INTERNET`.
- Запуск **тільки** через: *Налаштування → Про пристрій → Перевірити оновлення* (Intent з TvSettings).
- **Не працює у фоні**: перевірка GitHub виконується один раз при відкритті Activity після натискання користувача.

### Привілеї прошивки для EdUIUpdate

- `/system/etc/init/edui.rc` містить `on post-fs-data → /system/bin/sh /system/bin/edui_dm.sh`.
- `edui_dm.sh` виконує `chown 1000:1000 /dev/block/dm-*` + `chmod 0660` → uid 1000 (system) отримує право запису на логічні розділи super (system_a, system_ext_a тощо) **без root**.
- Якщо прямий запис недоступний — EdUIUpdate автоматично перемикається на `su` (fallback).

## Механізм прошивки

1. Завантаження `update.zip` з GitHub Release (файл `update.zip` відповідного tag).
2. Перевірки:
   - ZIP відкривається, є `manifest.json` і всі файли зі списку;
   - `manifest.version` == версія Release (Git tag без `v`);
   - SHA-256 кожного образу збігається з manifest.
3. Прошивка кожного розділу:
   - **EdUIUpdate**: `RandomAccessFile` на `/dev/block/mapper/<partition>` (прямий запис, uid 1000), з fallback на `su`;
   - **EdUIInstaller**: `su -c 'exec <toybox> dd if=… of=… bs=4M …'` — `toybox` попередньо копіюється у приватний каталог застосунку, щоб процес прошивки не залежав від `/system`, який у цей момент перезаписується.
   - Розділ визначається з manifest (`system_a`, `system_ext_a`); шлях — `/dev/block/mapper/<name>` (fallback `/dev/block/by-name/`).
4. `fsync` після кожного образу, `sync` у кінці.
5. Автоматичне `reboot` (`PowerManager.reboot` для EdUIUpdate; `su -c reboot` для Installer).

## Чого НЕ робиться (гарантії)

- Немає wipe/factory reset — `userdata`, `cache`, `metadata` не чіпаються.
- Прошиваються тільки розділи зі списку `manifest.json` — жодних зайвих записів.
- EdUIUpdate не має фонових сервісів, receivers, boot-колбеків — існує лише Activity.

## Захист під час прошивки

- `FLAG_KEEP_SCREEN_ON` на час операції.
- UI блокує Back під час прошивки (toast «не переривайте»).
- Велике попередження «Не вимикайте приставку!».
- Прошивка виконується послідовно, з fsync — при перериванні живлення можливий період відновлення через повний Amlogic-образ з того ж Release.

## Версіонування

- `versionName` EdUIUpdate.apk = версія EdUI (напр. `1.0.0`).
- Git tag `v1.0.0` → Release «EdUI v1.0.0» → `manifest.version = "1.0.0"`.
- Порівняння — семантичне (x.y.z), не за датою/іменем файлу.
