# websystemspl/wordpress-update-package

Paczka Composer odpowiedzialna za aktualizacje i licencjonowanie wtyczek oraz motywów WordPress, integrująca się z [UpdatePulse Server](https://github.com/Anyape/updatepulse-server).

Działa zarówno dla **wtyczek**, jak i **motywów** — klasa automatycznie wykrywa typ na podstawie ścieżki.

---

## Instalacja

```bash
composer require websystemspl/wordpress-update-package
```

---

## Użycie w wtyczce

**`my-plugin.php`:**

```php
<?php
/*
 * Plugin Name: My Plugin
 * Version: 1.0.0
 * Require License: yes
 */

use Anyape\UpdatePulse\Updater\v2_0\UpdatePulse_Updater;

require_once plugin_dir_path( __FILE__ ) . 'vendor/autoload.php';

$myplugin_updater = new UpdatePulse_Updater(
    wp_normalize_path( __FILE__ ),
    wp_normalize_path( __DIR__ )
);
```

---

## Użycie w motywie

**`functions.php`:**

```php
<?php
use Anyape\UpdatePulse\Updater\v2_0\UpdatePulse_Updater;

require_once get_stylesheet_directory() . '/vendor/autoload.php';

$mytheme_updater = new UpdatePulse_Updater(
    wp_normalize_path( __FILE__ ),
    get_stylesheet_directory()
);
```

> Zmienna `$mytheme_updater` (oraz `$myplugin_updater`) musi mieć unikalny prefix — różny dla każdej wtyczki/motywu.

---

## Konfiguracja — updatepulse.json

W katalogu głównym wtyczki lub motywu musi znajdować się plik `updatepulse.json` z adresem serwera UpdatePulse:

```json
{
    "server": "https://twoj-serwer.pl"
}
```

Plik ten jest aktualizowany automatycznie przez klasę podczas aktywacji/dezaktywacji licencji (klucz, sygnatura, itd.).

---

## Licencja wymagana / współdzielona

- Aby wtyczka/motyw wymagał licencji, dodaj nagłówek w głównym pliku:
  ```
  Require License: yes
  ```
- Aby używać licencji innej wtyczki/motywu, dodaj:
  ```
  Licensed With: slug-innej-wtyczki
  ```

---

## Jak działa wykrywanie typu

Klasa `UpdatePulse_Updater` automatycznie wykrywa czy package jest wtyczką czy motywem:
- **Wtyczka** — ścieżka leży w `WP_PLUGIN_DIR` lub `WPMU_PLUGIN_DIR`
- **Motyw** — w katalogu pakietu istnieje plik `style.css`

Nie trzeba tego konfigurować ręcznie.
