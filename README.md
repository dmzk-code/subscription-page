## Remnawave Subscription Page
# Remnawave Subscription Page Fork

Learn more about Remnawave [here](https://remna.st/).
## Редактор конфигураций Remnawave
Проект ориентирован на простое и легкое редактирование конфигов, под mihomo, внутри remnawave sub-page 

# Contributors
### Основные функции: 
- Создание Авто-серверов(балансировка)
- Скрытие серверов из подписки
- Отказоустойчивость сервера
- Замена select'ов в proxy-groups
- Удаление select'ов в конфигурации

Check [open issues](https://github.com/remnawave/subscription-page/issues) to help the progress of this project.
### Добавлены/изменены файлы
- backend/src/modules/root/mihomo-layer.service.ts (новый)
- backend/src/modules/root/root.service.ts
- backend/src/modules/root/root.module.ts
- backend/src/common/config/app-config/config.schema.ts
- backend/src/common/constants/index.ts
- backend/src/common/constants/ignored-headers.constant.ts

<p align="center">
Thanks to the all contributors who have helped improve Remnawave:
</p>
<p align="center">
<a href="https://github.com/remnawave/subscription-page/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=remnawave/subscription-page" />
</a>
</p>
### Что вообще происходит?
- Когда ответ на подписку выглядит как Mihomo YAML (содержит прокси-серверы
  и/или прокси-групп), он анализируется и переписывается заново на основе правил:
  - применяются параметры баланс/переделка/скрытие/замена/удаление на основе конфига.
  - удаление удаляет элементы прокси и ссылки на группы повсюду.
  - новые группы вставляются перед ПРОКСИ и добавляются в ПРОКСИ, если они
не являются целью замены, скрыты или удалены.

### Куда поместить configMi.yaml?
- Поместите *configMi.yaml* на хост и смонтируйте его в контейнер.
- Установите для env **MIHOMO_LAYER_CONFIG_PATH** значение пути внутри контейнера.
- Если **MIHOMO_LAYER_CONFIG_PATH** не задан, серверная часть выполнит поиск
  *configMi.yaml* в рабочем каталоге серверной части (process.cwd()).

### Пример настройки (compose)
- Добавляем монтирование тома в контейнер серверной службы.
- Добавляем env *MIHOMO_LAYER_CONFIG_PATH*.

### Пример фрагмента (адаптируйте к вашему файлу compose):

```
services:
    remnawave-subscription-page:
        image: remnawave/sub-page-patched:7.0.5-patched
        container_name: remnawave-subscription-page
        hostname: remnawave-subscription-page
        restart: always
        env_file:
          - .env
        environment:
          - MIHOMO_LAYER_CONFIG_PATH=/app/configMi.yaml
        volumes:
          - /opt/remnawave/subscription/configMi.yaml:/app/configMi.yaml:ro
        ports:
          - '127.0.0.1:3010:3010'
        networks:
          - remnawave-network
```

## Полный цикл сборки
1. Клонировать оффициальный репозиторий *subscription-page* в папку *fork/sub-page-patched*
2. Скачать патч файлы из текущего репозитория *patch-files*
3. Поместить все файлы из *patch-files* в папку с официальным репозиторием *fork/sub-page-patched* с заменой файлов 
4. Выполнить команды внутри папки *fork* для сборки
```
docker build -t remnawave/sub-page-patched:7.0.5-patched -f sub-page-patched\Dockerfile sub-page-patched
```
```
docker save -o C:\Users\dmzk-code\fork\sub-page-patched.tar remnawave/sub-page-patched:7.0.5-patched
```
> [!IMPORTANT]
> Заменить на свой путь к папке *C:\Users\dmzk-code\fork*
> [!IMPORTANT]
> Если машина слабая, то лучше собирать на своём устройстве, Docker для Windows.
5. Загружаем образ на сервер *C:\Users\dmzk-code\fork\sub-page-patched.tar* в */opt/remnawave/subscription*
6. Загружаем файл *configMi.yaml* в */opt/remnawave/subscription*
7. Подгружаем слои

```
docker load -i /opt/remnawave/subscription/sub-page-patched.tar
```
8. Выполняем команды
```
cd /opt/remnawave/subscription && docker compose down && docker compose up -d && docker compose logs -f -t
```
> [!NOTE]
> Если нет файла *configMi.yaml* в */opt/remnawave/subscription* страница подписки будет работать как обычно
## Пример файла configMi.yaml
```
balance:
  - name: "📡 [AUTO] - Обычные"
    hidden: true
    type: load-balance
    strategy: sticky-sessions # или round-robin
    input-proxies: #тут мы перечесляем какие сервера будут в балансере для автовыбора
      - "🇫🇮 [B1] - Финляндия"
      - "🇪🇪 [B1] - Эстония"
      - "🇩🇪 [B1] - Германия"
      - "🇺🇸 [B1] - США"
      - "🇮🇹 [B1] - Италия"
  - name: "🔐 [AUTO] - Криптографические"
    hidden: true
    type: load-balance
    strategy: sticky-sessions # или consistent-hashing
    input-proxies:
      - "🇫🇮 [K1] - Финляндия"
      - "🇪🇪 [K1] - Эстония"
      - "🇩🇪 [K1] - Германия"
      - "🇺🇸 [K1] - США"
      - "🇮🇹 [K1] - Италия"
remake:
  - name: "🇫🇮 [B1] - Финляндия"
    hidden: true
    output-name: "🇫🇮 [B1-WTL] - Финляндия"
    type: fallback
    input-proxies: #тут мы перечесляем какая группа будет в отказоустойчивости
      - "🇫🇮 [B1] - Финляндия"
      - "🇫🇮 [WTL] - Финляндия"
  - name: "🇪🇪 [B1] - Эстония"
    hidden: true
    output-name: "🇪🇪 [B1-WTL] - Эстония"
    type: fallback 
    input-proxies:
      - "🇪🇪 [B1] - Эстония"
      - "🇪🇪 [WTL] - Эстония"
  - name: "🇩🇪 [B1] - Германия"
    hidden: true
    output-name: "🇩🇪 [B1-WTL] - Германия"
    type: fallback
    input-proxies:
      - "🇩🇪 [B1] - Германия"
      - "🇩🇪 [WTL] - Германия"
  - name: "🇺🇸 [B1] - США"
    hidden: true
    output-name: "🇺🇸 [B1-WTL] - США"
    type: fallback
    input-proxies:
      - "🇺🇸 [B1] - США"
      - "🇺🇸 [WTL] - США"
hidden:
  input-proxies: #тут мы перечесляем какие сервера не будут видны пользователю так как к ним можно будет подключиться только через fallback в случае если напрямую нет доступа к нужному серверу
    - "🇫🇮 [WTL] - Финляндия"
    - "🇪🇪 [WTL] - Эстония"
    - "🇩🇪 [WTL] - Германия"
    - "🇺🇸 [WTL] - США"
replace:
  proxies: #тут мы перечесляем какой select в proxy-groups заменяем на другой select
    "📡 [B1] - Обычные": "📡 [AUTO] - Обычные"
    "🔐 [K1] - Криптографические": "🔐 [AUTO] - Криптографические"
    "🇫🇮 [B1] - Финляндия": "🇫🇮 [B1-WTL] - Финляндия"
    "🇪🇪 [B1] - Эстония": "🇪🇪 [B1-WTL] - Эстония"
    "🇩🇪 [B1] - Германия": "🇩🇪 [B1-WTL] - Германия"
    "🇺🇸 [B1] - США": "🇺🇸 [B1-WTL] - США"
delete:
  proxies: #тут мы перечесляем что хотим удалить из конфигурации полностью
    - "📡 [B1] - Обычные"
    - "🔐 [K1] - Криптографические"
    - "🔆 [XT] - Стабильные день"
    - "🌗 [WTL] - Телефон ночь"
    
```
> [!TIP]
> Патч тестировался на версии 7.0.5.
