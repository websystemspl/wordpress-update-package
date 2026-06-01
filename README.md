# websystemspl/wordpress-update-package

Paczka Composer odpowiedzialna za aktualizacje i licencjonowanie wtyczek oraz motywów WordPress, integrująca się z [UpdatePulse Server](https://github.com/Anyape/updatepulse-server).

Bazuje na bibliotece [updatepulse-updater](https://github.com/Anyape/updatepulse-server-integration) z modyfikacjami opisanymi w sekcji [Zmiany względem oryginału](#zmiany-względem-oryginału).

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

Plik zawiera **tylko** adres serwera. Dane licencji (klucz, sygnatura, itd.) są przechowywane w bazie danych WordPress (`wp_options`) i nie są zapisywane do pliku — dzięki temu aktualizacja wtyczki/motywu nie kasuje licencji.

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

---

## Zmiany względem oryginału

Oryginalna biblioteka pochodzi z [Anyape/updatepulse-server-integration](https://github.com/Anyape/updatepulse-server-integration) i była przeznaczona do ręcznego kopiowania do folderu `lib/` wtyczki/motywu. Poniżej lista zmian wprowadzonych w tej paczce:

### 1. Integracja z Composerem

**Oryginał:** biblioteka ładowana przez `require_once` wskazujący na `lib/updatepulse-updater/class-updatepulse-updater.php`.

**Zmiana:** paczka rejestruje klasę przez autoload Composera (`autoload.files`). W projekcie wystarczy:
```php
require_once plugin_dir_path( __FILE__ ) . 'vendor/autoload.php';
```

### 2. Usunięcie bundlowanego plugin-update-checker

**Oryginał:** biblioteka zawierała kopię `yahnis-elsts/plugin-update-checker` w podfolderze `lib/plugin-update-checker/` i ładowała ją ręcznie przez `require`.

**Zmiana:** `yahnis-elsts/plugin-update-checker` jest zadeklarowany jako zależność Composera (`^5.7`). Fallback `require` wskazuje na `vendor/yahnis-elsts/plugin-update-checker/` zamiast na lokalny `lib/`.

### 3. Ścieżki do assets przeniesione na __DIR__

**Oryginał:** ścieżki do plików JS, szablonów i tłumaczeń były budowane względem `$this->package_path` (katalogu wtyczki/motywu) z dołączonym prefiksem `lib/updatepulse-updater/`:
```php
$this->package_path . 'lib/updatepulse-updater/js/main.js'
$this->package_url  . '/lib/updatepulse-updater/js/main.js'
```

**Zmiana:** ścieżki oparte o `__DIR__` klasy (katalog paczki w vendorze):
```php
__DIR__ . '/js/main.js'
plugin_dir_url( __FILE__ ) . 'js/main.js'
__DIR__ . '/templates/'
__DIR__ . '/languages/'
```

### 4. Przechowywanie danych licencji w bazie danych

**Oryginał:** wszystkie opcje (licenseKey, licenseSignature, licenseNextDeactivate, itd.) były zapisywane do pliku `updatepulse.json` w katalogu wtyczki/motywu. Przed aktualizacją klasa kopiowała je do `wp_options` jako backup, a po aktualizacji — z powrotem do pliku.

**Zmiana:** dane licencji są **zawsze** przechowywane w `wp_options` pod kluczem `updatepulse_{slug}_options`. Plik `updatepulse.json` przechowuje wyłącznie `server` URL (config deploy-time, commitowany do repozytorium). Metody `save_updatepulse_options()` i `restore_updatepulse_options()` są no-op — backup przed aktualizacją nie jest potrzebny.

### 5. Kompatybilność z plugin-update-checker v5.7

**Oryginał:** używał namespace `YahnisElsts\PluginUpdateChecker\v5p3\PucFactory`.

**Zmiana:** zaktualizowany do `v5p7` zgodnie z aktualną wersją biblioteki (`^5.7`).
