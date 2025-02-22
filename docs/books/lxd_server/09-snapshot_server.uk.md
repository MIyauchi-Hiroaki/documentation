---
title: 9 Сервер snapshot
author: Steven Spencer
contributors: Ezequiel Bruni
tested_with: 8.8, 9.2
tags:
  - lxd
  - enterprise
  - lxd snapshot сервер
---

# Глава 9: Сервер Snapshot

У цій главі використовується комбінація привілейованого (root) користувача та непривілейованого (lxdadmin) користувача залежно від завдань, які ми виконуємо.

Як зазначалося на початку, snapshot сервер для LXD має бути дзеркалом робочого сервера всіма можливими способами. Причина полягає в тому, що вам може знадобитися перенести його на робочий стан у разі апаратної несправності, а наявність не лише резервних копій, але й швидкого способу запуску робочих контейнерів зводить до мінімуму панічні телефонні дзвінки та текстові повідомлення системних адміністраторів. ЦЕ ЗАВЖДИ ДОБРЕ!

Таким чином, процес створення snapshot сервера точно схожий на робочий сервер. Щоб повністю імітувати налаштування нашого робочого сервера, виконайте всі **розділи 1-4** ще раз на snapshot сервері, а коли завершите, поверніться до цього місця.

Ви повернулись!! Вітаємо, це має означати, що ви успішно завершили базову інсталяцію snapshot сервера.

## Налаштування зв’язку між основним і snapshot сервером

Перш ніж продовжити, потрібно трохи прибрати. По-перше, якщо ви працюєте у робочому середовищі, ви, ймовірно, маєте доступ до DNS-сервера, який можна використовувати для налаштування IP-адреси для розпізнавання імен.

У нашій лабораторії ми не маємо такої розкоші. Можливо, у вас такий самий сценарій. З цієї причини ви збираєтеся додати IP-адреси та імена серверів до файлу /etc/hosts на основному сервері та снепшот сервері. Вам потрібно буде зробити це як ваш root (або _sudo_) користувач.

У нашій лабораторії основний сервер LXD працює на 192.168.1.106, а сервер знімків LXD працює на 192.168.1.141. SSH на кожному сервері та додайте наступне до файлу /etc/hosts:

```
192.168.1.106 lxd-primary
192.168.1.141 lxd-snapshot
```

Далі вам потрібно дозволити весь трафік між двома серверами. Для цього ви збираєтеся змінити правила `firewalld`. Спочатку на сервері lxd-primary додайте цей рядок:

```
firewall-cmd zone=trusted add-source=192.168.1.141 --permanent
```

а на сервері знімків додайте це правило:

```
firewall-cmd zone=trusted add-source=192.168.1.106 --permanent
```

потім перезавантажте:

```
firewall-cmd reload
```

Далі, як наш непривілейований користувач (lxdadmin), вам потрібно встановити довірчі відносини між двома машинами. Це робиться за допомогою наступного виконання на lxd-primary:

```
lxc remote add lxd-snapshot
```

Це відображає сертифікат, який потрібно прийняти. Прийміть його, і з’явиться запит на введення пароля. Це «пароль довіри», який ви встановлюєте під час виконання кроку ініціалізації LXD. Сподіваємось, ви надійно зберігаєте всі ці паролі. Коли ви введете пароль, ви отримаєте це:

```
Client certificate stored at server:  lxd-snapshot
```

Не завадить мати це і в зворотному порядку. Наприклад, також установіть довірчі відносини на сервері lxd-snapshot. Таким чином, за потреби, сервер lxd-snapshot може надсилати знімки назад на основний сервер lxd-primary. Повторіть кроки та замініть у «lxd-primary» на «lxd-snapshot».

### Перенесення вашого першого snapshot

Перш ніж ви зможете перенести свій перший snapshot, вам потрібно створити будь-які профілі на lxd-snapshot, які ви створили на lxd-primary. У нашому випадку це профіль "macvlan".

Вам потрібно буде створити це для lxd-snapshot. Поверніться до [розділу 6](06-profiles.md) і створіть профіль "macvlan" у lxd-snapshot, якщо потрібно. Якщо ваші два сервери мають однакові назви батьківського інтерфейсу (наприклад, "enp3s0"), ви можете скопіювати профіль "macvlan" до lxd-snapshot без його повторного створення:

```
lxc profile copy macvlan lxd-snapshot
```

Коли всі зв’язки та профілі налаштовано, наступним кроком є фактичне надсилання знімка з lxd-primary до lxd-snapshot. Якщо ви точно слідкували за цим, ви, ймовірно, видалили всі свої знімки. Створіть інший знімок:

```
lxc snapshot rockylinux-test-9 rockylinux-test-9-snap1
```

Якщо ви запустите команду «info» для `lxc`, ви побачите знімок у нижній частині нашого списку:

```
lxc info rockylinux-test-9
```

Унизу буде показано щось на зразок цього:

```
rockylinux-test-9-snap1 at 2021/05/13 16:34 UTC) (stateless)
```

Тож, тримаємо пальці! Давайте спробуємо перенести наш snapshot:

```
lxc copy rockylinux-test-9/rockylinux-test-9-snap1 lxd-snapshot:rockylinux-test-9
```

Ця команда говорить, що всередині контейнера rockylinux-test-9 ви хочете надіслати snapshot rockylinux-test-9-snap1 до lxd-snapshot і назвати його rockylinux-test-9.

Через короткий час копіювання буде завершено. Хочете дізнатися напевно? Створіть `список lxc` на сервері lxd-snapshot. Що має повернути наступне:

```
+-------------------+---------+------+------+-----------+-----------+
|    NAME           |  STATE  | IPV4 | IPV6 |   TYPE    | SNAPSHOTS |
+-------------------+---------+------+------+-----------+-----------+
| rockylinux-test-9 | STOPPED |      |      | CONTAINER | 0         |
+-------------------+---------+------+------+-----------+-----------+
```

Успішно! Спробуйте запустити його. Оскільки ми запускаємо його на сервері lxd-snapshot, вам потрібно спершу зупинити його на сервері lxd-primary, щоб уникнути конфлікту IP-адрес:

```
lxc stop rockylinux-test-9
```

А на сервері lxd-snapshot:

```
lxc start rockylinux-test-9
```

Якщо припустити, що все це працює без помилок, зупиніть контейнер на lxd-snapshot і запустіть його знову на lxd-primary.

## Вимкнути boot.autostart для контейнерів

Snapshots, скопійовані в lxd-snapshot, не працюватимуть під час міграції, але якщо у вас виникне подія живлення або потрібно перезавантажити сервер знімків через оновлення чи щось інше, у вас виникне проблема. Ці контейнери намагатимуться запуститися на сервері знімків, створюючи потенційний конфлікт IP-адрес.

Щоб усунути це, потрібно налаштувати переміщені контейнери так, щоб вони не запускалися після перезавантаження сервера. Для нашого щойно скопійованого контейнера rockylinux-test-9 ви зробите це за допомогою:

```
lxc config set rockylinux-test-9 boot.autostart 0
```

Зробіть це для кожного знімка на сервері lxd-snapshot. «0» у команді переконається, що `boot.autostart` вимкнено.

## Автоматизація snapshot процесу

Чудово, що ви можете створювати миттєві snapshots, коли вам це потрібно, і іноді вам _потрібно_ створити миттєвий snapshot вручну. Ви навіть можете вручну скопіювати його в lxd-snapshot. Але в інших випадках, особливо для багатьох контейнерів, які працюють на вашому первинному сервері lxd, **останнє**, що ви хочете зробити, це провести півдня, видаляючи snapshots на snapshot сервері знімків, створюючи нові snapshots та надсилаючи їх на snapshot сервер. Для основної частини ваших операцій ви захочете автоматизувати процес.

Перше, що вам потрібно зробити, це запланувати процес автоматизованого створення снепшоту на lxd-primary. Ви зробите це для кожного контейнера на сервері lxd-primary. Після завершення він подбає про це в майбутньому. Це можна зробити за допомогою наступного синтаксису. Зверніть увагу на схожість із записом crontab для позначки часу:

```
lxc config set [container_name] snapshots.schedule "50 20 * * *"
```

Це означає, що щодня о 20:50 робіть знімок назви контейнера.

Щоб застосувати це до нашого контейнера rockylinux-test-9:

```
lxc config set rockylinux-test-9 snapshots.schedule "50 20 * * *"
```

Ви також хочете налаштувати назву знімка, щоб мати значення до нашої дати. LXD скрізь використовує UTC, тому наш найкращий вибір, щоб відстежувати речі, це встановити ім’я знімка з датою та міткою часу в більш зрозумілому форматі:

```
lxc config set rockylinux-test-9 snapshots.pattern "rockylinux-test-9{{ creation_date|date:'2006-01-02_15-04-05' }}"
```

ЧУДОВО, але ви точно не хочете отримувати новий знімок щодня, не позбувшись старого, чи не так? Ви б заповнили диск знімками. Щоб виправити це, виконайте:

```
lxc config set rockylinux-test-9 snapshots.expiry 1d
```
