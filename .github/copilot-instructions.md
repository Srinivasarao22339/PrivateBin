# PrivateBin AI Coding Instructions

## Overview
PrivateBin is a zero-knowledge pastebin with client-side AES-256-GCM encryption. The server stores encrypted data without access to plaintext. Built with PHP (MVC architecture) and JavaScript for encryption/decryption.

## Architecture
- **Entry Point**: `index.php` instantiates `PrivateBin\Controller`
- **MVC Structure**: Controller handles requests (create/read/delete), Model factories Paste/Store objects, View renders via templates in `tpl/`
- **Data Flow**: Client encrypts data with SJCL, sends to server; server stores in configured backend (Filesystem default, supports Database/GCS/S3)
- **Key Components**:
  - `lib/Controller.php`: Routes operations (create/delete/read)
  - `lib/Model.php`: Factory for Paste and storage backends
  - `lib/Data/`: AbstractData subclasses for storage (e.g., `Filesystem.php`, `Database.php`)
  - `js/privatebin.js`: Client-side encryption, UI logic
  - `cfg/conf.sample.php`: INI config for features, model options

## Critical Workflows
- **Install Dependencies**: `composer install` (updates autoloader)
- **Run Tests**: `cd tst && phpunit` (PHP) or `cd js && mocha` (JS)
- **Migrate Data**: `php bin/migrate <src_conf> <dst_conf>` (e.g., filesystem to DB)
- **Configuration**: Copy `cfg/conf.sample.php` to `cfg/conf.php`, adjust model/storage
- **Version Bump**: `make increment VERSION=x.y.z` (updates files, commits)
- **Debug**: Use PHP error logs; client issues via browser console

## Project Conventions
- **Namespaces**: PSR-4, `PrivateBin\` in `lib/`
- **Config**: INI format with PHP header for security
- **Encryption Format**: FormatV2 JSON (iv, ct, etc.) for pastes/comments
- **Storage Abstraction**: Extend `Data\AbstractData` for new backends
- **I18n**: JSON files in `i18n/`, use `I18n::_()` for strings
- **Templates**: PHP files in `tpl/`, Bootstrap-based by default
- **Tests**: PHPUnit in `tst/`, mock configurations for isolation

## Specific Patterns
- **Paste Creation**: POST JSON with encrypted `data`/`attachment`, meta like `expire`, `formatter`
- **Error Handling**: Use `Controller::GENERIC_ERROR` for expired/missing pastes
- **Traffic Limiting**: `Persistence\TrafficLimiter` via config `[traffic]`
- **Purge Expired**: Automatic via `Model::purge()` on create, configurable batch size
- **File Uploads**: Encrypted `attachment`/`attachmentname`, preview via `kjua` QR lib

## Examples
- Add feature: Extend Controller with new operation, add route in `__construct()`
- New storage: Create `lib/Data/MyStorage.php` extending AbstractData, configure in `[model]`
- Client change: Modify `js/privatebin.js`, test with `make test-js`
- Config option: Add to `conf.sample.php`, read via `$this->_conf->getKey()` in Controller/Model