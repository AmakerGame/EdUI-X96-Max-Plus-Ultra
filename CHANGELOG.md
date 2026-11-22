# Що змінено в EdUI v1.0.0

Перший реліз EdUI для X96 Max Plus Ultra на базі SlimBOXTV 49.

## Система

- Україська локалізація: налаштування (TvSettings, SBXsettings), лаунчер Google TV (повний переклад видимих рядків, вкл. «Прискорення»/«Фон»).
- Видалено російськомовні/непотрібні застосунки: Zona, Xplore, RuStore/VK, Yandex Browser TV, FMPLAY, KinoTrend, PowerMenu, TorrServe, VimuMediaPlayer, AirReceiver, ByeByeDPI, SoundAmplifier.
- `ro.product.locale=uk-UA`, `persist.sys.locale=uk-UA`, `Europe/Kyiv`.

## Налаштування → Про пристрій

Новий розділ **«EdUI — X96MPU»**:

- **Стан системи та пристрою** — модель X96MPU v1, Android, білд, fingerprint, плата, версія/lock/verified/слот/serial завантажувача, локаль, dm-verity.
- **Керування завантажувачем** — backup/restore U-Boot env на /sdcard, reboot, reboot→update.
- **«Перевірити оновлення»** — запуск EdUIUpdate (системний компонент оновлення).

## Компоненти

- **EdUIUpdate.apk** — системний компонент оновлення (`com.edui.update`, uid system, без іконки лаунчера).
- **EdUIInstaller.apk** — інсталятор первинного встановлення (у Release; на стокову прошивку).
- **LeanKeyboardF** — українська клавіатура за замовчуванням (autodefault через init.rc).

## Завантажувач (U-Boot env)

- AVB вимкнено (`avb2=0`), `get_avb_mode` прибрано з boot-ланцюжка.
- `androidboot.flash.locked=0`, `verifiedbootstate=orange`.
- Максимальне логування ядра: `ignore_loglevel`, `initcall_debug`, `log_buf_len=4M`, `loglevel=7`.

## Виправлені помилки

- `SBXcustomizer`: усунуто `Resources.NotFoundException` (`ConfigurationProvider.query` → неіснуючий string id) — зник періодичний краш-кожні-15-секунд.
- Settings: усунуто `ActivityNotFoundException` для пунктів EdUI/оновлення — тепер рідні PreferenceFragment + прямий Intent на EdUIUpdate.

## Відомі проблеми

- Wi-Fi: періодичні `wifi_mac_scan_timeout` / `WifiVendorHal` помилки статистики — обмеження драйвера W150S1.
- `tee_rpmb_init Verify key failed` — відсутнє factory-провіжнування RPMB (не впливає на роботу системи).
