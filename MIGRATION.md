# Как перенести новый дизайн на рабочий сайт Drive Phangan

Этот репозиторий — **макет** (статичный HTML). Рабочий сайт с базой, ценами, бронированием и админкой
живёт в репозитории `DanyWB/driverbot` (форк: `drivephangan2020/driverbot`).
Переносим **только внешний вид**. Логику (маршруты, база, расчёт цен, брони, CMS) не трогаем.

## 1. Как устроен рабочий сайт

Laravel + Blade-шаблоны. Всё, что относится к публичному сайту:

| Что | Файл в `backend/` |
|---|---|
| Шапка, меню, языки, подвал (общие для всех страниц) | `resources/views/website/layout.blade.php` |
| Главная | `resources/views/website/home.blade.php` |
| Каталог скутеров / авто | `resources/views/website/catalog.blade.php` + `partials/card.blade.php` |
| Страница машины / скутера | `resources/views/website/vehicle.blade.php` |
| Бронирование, статус заявки | `booking.blade.php`, `status.blade.php` |
| Условия, конфиденциальность, контакты | `policy.blade.php`, `partials/contacts.blade.php` |
| Стили сайта | `resources/css/website.css` |
| Скрипты сайта (даты, фильтры, меню) | `resources/js/website/app.ts` |
| Тексты EN / RU / UK | `lang/en/website.php`, `lang/ru/website.php`, `lang/uk/website.php` |
| Картинки | `public/website/images/` |
| Маршруты (`/en/scooters`, `/ru/policies/terms` …) | `routes/website.php` |

Соответствие страниц макета и маршрутов:

| Макет | Шаблон Laravel | Адрес |
|---|---|---|
| `index.html` | `home.blade.php` | `/{lang}` |
| `scooters.html`, `cars.html` | `catalog.blade.php` | `/{lang}/scooters`, `/{lang}/cars` |
| `terms.html` | `policy.blade.php` (terms) | `/{lang}/policies/terms` |
| `contacts.html` | `policy.blade.php` (contact) | `/{lang}/policies/contact` |
| `guide.html` | маршрута нет, нужно добавить | `/{lang}/island-guide` |

## 2. Порядок работ

Работать в отдельной ветке, например `design/v2`, и вливать её только после проверки.

1. **Картинки.** Скопировать `img/` из макета в `backend/public/website/images/v2/`.
2. **Стили.** Собрать CSS из блоков `<style>` всех страниц макета в `resources/css/website-v2.css`:
   - общая часть: токены `--tp-*`, шапка `.hdr`, меню `.mnav`, подвал `.ftr`, кнопки `.btn--*`;
   - затем стили конкретных страниц.

   В `layout.blade.php` заменить `resources/css/website.css` на новый файл в строке `@vite([...])`.
   Старый файл не удалять до конца переноса.
3. **Скрипты.** На сайте включён CSP с nonce, поэтому встроенные `<script>` из макета работать не будут.
   Их логику нужно перенести в `resources/js/website/` как модули:
   - меню;
   - выпадающий список языков;
   - круг с подсветкой;
   - слайдер отзывов;
   - `fitH1` / `fitFtr`;
   - фильтры каталога.

   **Не переносить** `setLang`, словарь `T` и `localStorage`: на сайте язык задаётся адресом (`/en`, `/ru`, `/uk`).
4. **Шапка и подвал.** В `layout.blade.php` заменить разметку `<header>` и `<footer>` на разметку из макета:
   - пункты меню берутся из `$navigation[...]`;
   - языки берутся из `$languages` (каждый пункт — обычная ссылка `href="{{ $language['url'] }}"`);
   - подпись языка `uk` — **UA**.

   Фото внизу страницы задаётся из каждой страницы через `@section('footer_image', '...')`.
5. **Тексты.** Перенести тексты из словарей `T` и блоков `[data-terms-lang]` в `lang/*/website.php`.
   В шаблонах выводить их через `$t('ключ')`.
6. **Главная** (`home.blade.php`):
   - верхний блок с фото;
   - блок «О нас»;
   - круг «Find your ride»: ссылки на `$navigation['cars']` и `$navigation['scooters']`;
   - отзывы.
7. **Каталоги** (`partials/card.blade.php`):
   - разметка карточки как `.sc-card` (скутеры) или `.cr-card` (авто), данные — из модели машины;
   - фильтры работают по `data-*` атрибутам карточки;
   - выбор дат и расчёт цены: оставить существующие `partials/dates`, `quote`, `availability` и только переоформить их.

   Расчёт «цена × дни» из макета не переносить: на сайте уже есть настоящий расчёт с сезонами.
8. **Условия, контакты, гид.**
   - Условия и контакты оформить по `terms.html` и `contacts.html`.
   - Для гида добавить маршрут в `routes/website.php`, метод в `WebsiteController` и новый шаблон.
9. **Проверка локально:**
   ```bash
   cd backend
   composer install && npm install
   php artisan serve
   npm run dev
   php artisan test
   ```
   Проверить все языки, ширину телефона (375px) и компьютера, весь путь: выбор → даты → бронь → статус.

## 3. Чтобы управлять сайтом самому (без Дани)

Нужно три вещи:

1. **Код.** У вас есть форк `drivephangan2020/driverbot`, в нём можно работать полностью самостоятельно.
2. **Сервер.** Сайт `drivephangan.com` работает на сервере (VPS). В репозитории нет автоматической выкладки,
   выкладывают скриптом `deploy/scripts/deploy-release.sh` прямо на сервере. Для этого нужен **SSH-доступ к серверу**
   и пароль/ключ от аккаунта хостинга. Попросите их у Дани один раз (лучше добавить ваш собственный SSH-ключ),
   плюс доступ к DNS домена `drivephangan.com` и секреты из `.env` (база, Telegram-бот).
3. **Знания.** Инструкции по выкладке и откату: `OPERATIONS_RUNBOOK.md`, `deploy/scripts/rollback-release.sh`.

Без доступа к серверу и домену изменения можно готовить в форке, но выложить их на рабочий сайт не получится.

## 4. Что заменить в макете перед переносом

- Заглушки отзывов → реальные отзывы Google.
- `scooter-placeholder.webp` → реальные фото скутеров (на сайте они уже есть в галереях машин).
- Плашка «Claude preview» (`.note`) — убрать.
- Тексты гида и переводы RU/UA — вычитать.
