# Набор скриптов и веб-интерфейс для правки границ

В этих каталогах лежит набор инструментов для редактирования набора границ
в формате [Osmosis Poly](http://wiki.openstreetmap.org/wiki/Osmosis/Polygon_Filter_File_Format).

## Развёртывание в Docker

Самое простое — запустить систему в Docker-контейнерах.

#### Предварительные требования
* Должен быть установлен docker https://docs.docker.com/engine/install/
* и docker-compose https://docs.docker.com/compose/install/
* Для всей планеты во время сборки необходимо ~200 Гб дискового пространства
(после сборки — 30 Гб), разворачивание может длиться около суток.

#### Настройка сборки
В файле docker-compose.yaml нужно выставить желаемый порт, на котором будет
работать веб-интерфейс (сейчас это число 8081 в строке "8081:80"),
и URL с файлом планеты в переменной PLANET_URL. Переменные PLANET_URL_&lt;suffix&gt;
не используются, это просто примеры. Для тестирования советуется подставить
в PLANET_URL небольшой файл, тогда вся сборка займёт несколько минут.
 

## Развёртывание вручную
Для работы требуется база данных PostgreSQL + PostGIS, инициализированная
скриптами из каталога `db`. Последовательность выполнение скриптов и необходимые
переменные окружения см. в `db/Dockerfile.db` и `docker-compose.yaml`.
Для оценки размера файла MWM нужно заполнить
таблицу `tiles` из файла планеты (см. скрипты `db/*tiles*.sh`).

Также для обновления и замены границ из OpenStreetMap желательно импортировать
таблицу `osm_borders` — см. `db/prepare_borders.sh` и `db/load_borders.sh`.
Начальный набор границ для редактирования можно либо загрузить скриптом
`scripts/poly2postgis.py`, либо скопировать из таблицы `osm_borders` по,
например, `admin_level=2`.

После редактирования набор файлов `poly` создаст скрипт `scripts/export_poly.py`
или ссылка *Скачать poly - всё* в веб-интерфейсе.

#### Серверная часть

Два скрипта в каталоге `server` должны работать постоянно на фоне.

* `borders-api.py` — сервер на Flask, работает на порту 5000. К нему обращается
    веб-интерфейс через XHR-запросы. В начале скрипта проверьте названия таблиц
    и флаг READONLY.

* `borders-daemon.py` — непрерывно проверяет таблицу `borders` на пустые значения
    в столбце количества данных, и найдя их, пересчитывает. Запустите, если нужна
    оценка размера MWM.

#### Веб-интерфейс

Файлы в каталоге `web/app/static` не требуют каких-либо интерпретаторов или
выделенных серверов: просто откройте `index.html` в браузере. На карте
нарисованы границы, по клику на границу панель справа наполнится кнопками.
Оттуда можно разрезать и склеивать границы, переименовывать их, заменять и
дополнять из таблицы `osm_borders`, а также экспортировать в JOSM для сложных
модификаций.

## Автор и лицензия

Написал Илья Зверев для MAPS.ME, опубликовано под лицензией MIT.
