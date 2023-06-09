---
layout: docs
title: Розгортання TileServer GL
---

# {{ title }}

TileServer GL — це потужний інструмент із відкритими сирцями, який дозволяє обслуговувати векторні та растрові тайли створені з різних джерел даних. Він надає вам можливість створити власний тайловий сервер і працювати з тайлами мапи, які можна візуалізувати та налаштовувати в режимі реального часу. Ось покроковий посібник, який допоможе вам розпочати роботу з TileServer GL.

## Встановлення

Переконайтеся, що у вашій системі встановлено Node.js. Ви можете завантажити та встановити останню версію LTS з офіційного вебсайту Node.js (<https://nodejs.org>{: target=_blank}).

Відкрийте термінал або командний рядок і виконайте таку команду, щоб встановити TileServer GL глобально:

```sh
npm install -g tileserver-gl
```

або ж ви можете скористатись образом Docker

```sh
docker pull klokantech/tileserver-gl:latest
```

## Підготовка даних

Визначтесь з джерелом даних, яке ви хочете використовувати для вашої мапи. TileServer GL підтримує різні формати даних, зокрема векторні дані у форматах GeoJSON, Shapefile або Mapbox Vector Tiles (MVT), а також растрові дані у форматах MBTiles або GeoTIFF.

Якщо ваші дані ще не перетворено у сумісний формат, вам, можливо, знадобиться конвертувати їх за допомогою таких інструментів, як [ogr2ogr](https://gdal.org/programs/ogr2ogr.html){: target=_blank}, [Tippecanoe](https://github.com/felt/tippecanoe){: target=_blank} або [Tilemaker](https://tilemaker.org){: target=_blank}. Зверніться до документації TileServer GL, щоб отримати конкретні інструкції щодо перетворення даних у потрібний формат.

## Конфігурація TileServer GL

Створіть файл конфігурації JSON, який визначає параметри тайлового сервера. Цей файл визначає джерела даних, стилі мапи та інші параметри сервера. Ви можете звернутися до документації TileServer GL для детального пояснення параметрів конфігурації.

Визначте стилі мапи, які ви хочете використовувати для рендерингу тайлів. Ви можете використовувати Mapbox Studio, Maputnik або інші інструменти для створення стилів мап, щоб створювати власні стилі у форматі Mapbox GL Style JSON.

## Запуск TileServer GL

Відкрийте термінал або командний рядок і перейдіть до теки, де знаходиться файл конфігурації. Виконайте таку команду, щоб запустити TileServer GL:

```sh
tileserver-gl <config.json>
```

*Замініть `<config.json>` вашим шляхом до файлу конфігурації.*

Після запуску TileServer GL ви можете отримати доступ до нього у вебоглядачі або використовувати його як джерело тайлів у своєму застосунку, який використовує їх для показу та робот з мапою. Сервер надає кінцеві точки для отримання векторних і растрових тайлів, які ви можете використовувати відповідно до ваших конкретних вимог.

## Кастомізація та інтеграція

TileServer GL дає змогу налаштувати рендеринг тайлів вашої мапи, змінюючи стилі мапи та налаштування конфігурації. Ви можете експериментувати з різними стилями, композиціями шарів і взаємодіями, щоб досягти бажаного вигляду мапи.

Ви можете інтегрувати тайловий сервер у свій веб або мобільний застосунок за допомогою бібліотек, таких як Maplibre GL JS, Leaflet, OpenLayers або інших фреймворків, які підтримують Mapbox GL Style JSON або растрові тайли.

## Запуск сервера з допомогою Docker

Встановіть відповідну версію Tilemaker. У випадку Ubuntu 18.04 це буде так

```sh
cd /opt/osm/
wget https://github.com/systemed/tilemaker/releases/download/v2.2.0/tilemaker-ubuntu-18.04.zip
```

Для інших операційних систем ви можете знайти відповідну версію Tilemaker у репозиторії GitHub: <https://github.com/systemed/tilemaker/releases>{: target=_blank}.

Використовуйте, один з регіональних екстрактів для своїх даних, наприклад

```sh
wget https://download.openstreetmap.fr/extracts/europe/ukraine/kiev-latest.osm.pbf
```

Ви також можете створити власний екстракт у різних форматах за допомогою сервісів, подібних до <https://extract.bbbike.org>{: target=_blank}.

Скопіюйте завантажені дані OSM (`*.osm.pbf`) до теки `/opt/osm/tilemaker/build/`:

```sh
cp *.osm.pbf /opt/osm/tilemaker/build/
```

та конвертуйте `*.osm.pbf` у формат MBTiles з допомогою Tilemaker:

```sh
cd /opt/osm/tilemaker/build/
./tilemaker \
    --input /opt/osm/tilemaker/build/kiev.osm.pbf \
    --output kiev.mbtiles \
    --process ../resources/process-example.lua \
    --config ../resources/config-example.json
```

Перенесіть створений файл `*.mbtiles` до теки `/opt/osm/opentiles` folder:

```sh
mv kiev.mbtiles /opt/osm/opentiles/
```

та поверніться до теки `/opt/osm/`

```sh
cd /opt/osm/
```

Запустіть тайловий сервер за допомогою Docker і образу `klokantech/tileserver-gl`:

```sh
docker run -d --rm -it -v $(pwd)/opentiles:/data -p 8080:80 klokantech/tileserver-gl:latest
```

Ваш тайловий сервер тепер працює, і ви можете отримати доступ до нього за адресою `http://[server_address]:8080/`.

TileServer GL пропонує гнучке та масштабоване рішення для розповсюдження ваших власних тайлів. Виконуючи ці кроки, ви можете налаштувати тайловий сервер, налаштувати джерела даних і стилі мап, а також почати розповсюдження векторних або растрових тайлів для вдосконалення застосунків, які їх використовують. Зверніться до документації TileServer GL, щоб отримати докладніші інструкції та розширені функції для подальшого налаштування тайлового сервера.

!!! tip
    Також корисно згадати статтю ["How to deploy an OSM tile server"](https://www.blef.fr/how-to-deploy-tile-server/){: target=_blank}, Крістофа Блефарі. У статті наведено докладні інструкції та вказівки щодо розгортання власного тайлового сервера OpenStreetMap (OSM). Вона охоплює різні аспекти, включаючи встановлення програмного забезпечення, підготовку даних, стиль мапи й конфігурацію сервера.
