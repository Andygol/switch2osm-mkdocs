---
layout: docs
title: Використання контейнера Docker
lang: uk
---

# {{ title}}

Якщо ви тільки бажаєте потестувати або ви використовуєте іншу операційну систему (відмінну від Ubuntu), або ви вже використовуєте Docker для контейнеризації&nbsp;– ви можете спробувати [цю інструкцію](https://github.com/Overv/openstreetmap-tile-server/blob/master/README.md){: target=_blank}. Вона базується на інструкції [звідси](/serving-tiles/manually-building-a-tile-server-ubuntu-18-04-lts/) і вже має готовий до використання контейнер.

## Docker

Якщо у вас ще не встановлено Docker, ви можете скористатись різноманітними порадами щодо його встановлення, наприклад, [одна з них в щоденниках OSM](https://www.openstreetmap.org/user/SomeoneElse/diary/45070){: target=_blank}, що посилається на це керівництво.

## Дані OpenStreetmap

Для демонстрації тут, завантажимо дані для Замбії та імпортуємо їх, але має спрацювати будь-який файл OSM `.pbf`. Для тесту спробуйте якийсь невеликий файл `.pbf`, для початку. Увійдіть з правами звичайного користувача до вашої системи та завантажте дані для Замбії:

```sh
cd ~
wget https://download.geofabrik.de/africa/zambia-latest.osm.pbf
```

Створіть docker volume для даних:

```sh
docker volume create openstreetmap-data
```

Встановіть контейнер та імпортуйте дані:

```sh 
time \
    docker run \
    -v /home/renderaccount/zambia-latest.osm.pbf:/data.osm.pbf \
    -v openstreetmap-data:/var/lib/postgresql/12/main \
    overv/openstreetmap-tile-server:1.3.10 import
```

Шлях до файлу даних має бути абсолютним шляхом. В цьому прикладі це домашня тека нашого користувача `renderaccount`.

Як довго це триватиме&nbsp;– залежить від швидкості вашого зʼєднання з мережею та розміру ділянки для завантаження. Замбія, яку ми обрали для прикладу, має досить не багато даних.

Зауважте, якщо щось піде не так, повідомлення про помилки можуть бути трохи загадковими; ви можете побачити “… is a directory” якщо файл з даними не буде знайдений. Команда `time` на початку рядка не є обовʼязковою, вона лише повідомить вам як довго тривав процес, щоб ви мали уявлення про це на майбутнє. Також, версію сервера `postgres`, що використовується в контейнері, було змінено з 1.3.5 на 1.3.6 (Див [докладніше тут](https://github.com/Overv/openstreetmap-tile-server/releases/tag/v1.3.6){: target=_blank}). Це означає, що "openstreetmap-data" створена у попередній версії не буде працювати, вам потрібно вилучити її за допомогою команди `#!sh docker volume rm openstreetmap-data` та створити її заново.

Для отримання докладної інформації про те, що на справді відбуватиметься, ознайомтесь з файлом [опису контейнера](https://github.com/Overv/openstreetmap-tile-server/blob/master/Dockerfile){: target=_blank}. Ви побачите, що він дуже схожий на інструкцію з “[розгортання тайлового сервера вручну](/serving-tiles/manually-building-a-tile-server-ubuntu-18-04-lts/)”, з незначними змінами, такими як URL для тайлів та використання внутрішнього облікового запису. Ви можете побачити що в середині контейнера використовується Ubuntu 18.04. Хоча вам і не треба взаємодіяти з системою безпосередньо ви можете зробити це за допомогою команди `#!sh docker exec -it mycontainernumber bash`.

Після завершення імпорту ви маєте побачити повідомлення подібне до цього:

```sh
Osm2pgsql took 568s overall

real    9m34.378s
user    0m0.030s
sys     0m0.060s
```

Тут повідомляється скільки часу тривав імпорт (в цьому випадку 9 з половиною хвилин).

Для запуску тайлового сервера скористайтесь командою:

```sh
docker run \
    -p 80:80 \
    -v openstreetmap-data:/var/lib/postgresql/12/main \
    -d overv/openstreetmap-tile-server:1.3.10 \
    run
```

та перевірте чи він працює відкривши посилання:

`http://адреса.вашого.сервера/tile/0/0/0.png`

Ви маєте побачити мапу світу у вашому оглядачі.

### Додаткова інформація

Цей докер-контейнер насправді підтримує набагато більше можливостей ніж наведено тут в прикладі, дивіться [readme](https://github.com/Overv/openstreetmap-tile-server/blob/master/README.md){: target=_blank}, щоб дізнатись деталі про оновлення, налаштування та таке інше.

### Перегляд тайлів

Для перегляду простої “рухомої мапи” скористаємось файлом [`sample_leaflet.html`](https://github.com/SomeoneElseOSM/mod_tile/blob/switch2osm/extra/sample_leaflet.html){: target=_blank}, який знаходиться в теці “extra” mod_tile’а. Змініть `hot` в URL у файлі на `tile`, і відкрийте його в оглядачі на компʼютері де встановлено контейнер docker. Якщо це не можливо через відсутність встановленого вебоглядача, вам доведеться замінити у файлі `127.0.0.1` на IP адресу сервера та скопіювати його в `/var/www/html` на цьому сервері.

У разі потреби завантаження іншої території, повторіть процес від завантаження даних з допомогою `wget` і далі. Це буде трохи швидше наступного разу, бо статичні дані для стилю вже в наявності і їх не треба завантажувати.

### Розгортання векторних мап

З прикладу вище ви вже маєте встановленний docker на своїй хостовій машині. Для розгортання векторних мап ми будемо використовувати готовий docker image від Klokan TileServer GL.
Скачаємо його: 

```
docker pull lokantech/tileserver-gl:latest
```

В векторних мапах став розповсюджений формат даних `*.mbtiles` , який ми можемо згенерувати самостійно за допомогою `tilemaker`.
Для цього скачуємо потрібний пакет для вашоЇ ОС і разорхувуємо його. В моєму випадку це Ubuntu `18.04`, тому архів відповідний :


```
cd /opt/osm/
wget https://github.com/systemed/tilemaker/releases/download/v2.2.0/tilemaker-ubuntu-18.04.zip
```
Інші відповідні архіви під вашу ОС/версію можна подивитись [тут](https://github.com/systemed/tilemaker/releases).
Генерувати наші векторін мапи будемо з `*.osm.pbf` формату, який можна знайти на `https://download.openstreetmap.fr/extracts/europe/ukraine/` або використати сервіс `https://extract.bbbike.org/` . Я використав другий сервіс, розпакував отриманий `tilemaker` в директорію `/opt/osm/`. скопіюємо наш `*.osm.pbf` в директорію `/opt/osm/tilemaker/build/` :
```
cp *.osm.pbf /opt/osm/tilemaker/build/
```
Тепер нам треба виконати процес конвертації `*.osm.pbf` -> `*.mbtiles`. Для цього  переходимо в директорію з виконуючим файлом `tilemaker` `cd /opt/osm/tilemaker/build` і запускаємо :


```
./tilemaker --input /opt/osm/tilemaker/build/kiev.osm.pbf --output kiev.mbtiles --process ../resources/process-example.lua --config ../resources/config-example.json
```
Копіюємо.переміщуємо нашу нову векторну мапу в `/opt/osm/opentiles` : 

```
mv /opt/osm/tilemaker/build/kiev.mbtiles /opt/osm/opentiles/
```

 Повертаємось в нашу першу директорію :
```
cd /opt/osm/
```

 і звідкти виконуємо старт нашовго серверу з векторними мапами :

```
docker run -d --rm -it -v $(pwd)/opentiles:/data -p 8080:80 klokantech/tileserver-gl:latest
```
Готово! Наш сервер запущений за адресою : `http://[server_address]:8080/`

<table>
<thead>
<tr>
<th align="center"><a class="glightbox" href="../../assets/img/vector_startpage_view.png" data-type="image" data-width="100%" data-height="auto" data-title="aaa" data-description="qqq" data-desc-position="bottom"><Стартова сторінка" data-title="" src="../../assets/img/vector_startpage_view.png" /></a></th>
</tr>
</thead>
</table>
