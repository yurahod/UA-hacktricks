

<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# ECB

(ECB) Electronic Code Book - це схема симетричного шифрування, яка **замінює кожен блок тексту** на **блок зашифрованого тексту**. Це **найпростіша** схема шифрування. Основна ідея полягає в тому, щоб **розділити** текст на **блоки по N біт** (залежно від розміру блоку вхідних даних, алгоритму шифрування) та потім шифрувати (розшифровувати) кожен блок тексту, використовуючи один ключ.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Використання ECB має кілька проблем з безпекою:

* **Блоки з зашифрованого повідомлення можуть бути видалені**
* **Блоки з зашифрованого повідомлення можуть бути переміщені**

# Виявлення вразливості

Уявіть, що ви входите в додаток кілька разів і **завжди отримуєте однакове значення своїх кукі**. Це тому, що кукі застосунку має формат **`<username>|<password>`**.\
Потім ви створюєте двох нових користувачів, обидва з **однаковим довгим паролем** і **майже однаковим юзернеймом**.\
Виявляється, що **блоки по 8B**, де **інформація обох користувачів** однакова **співпадають**. Тоді ви припускаєте, що це може бути тому, що **використовується ECB**.

Як у наступному прикладі. Спостерігайте, як ці **2 розкодовані cookies** мають повторюваний кілька разів блок **`\x23U\xE45K\xCB\x21\xC8`**

```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```

Це тому, що **юзернейм та пароль цих cookie містили кілька разів літеру "a"** (наприклад). **Блоки**, які **відрізняються**, - це блоки, які містили **принаймні 1 відмінний символ** (можливо, сепаратор "|" або якась необхідна відмінність в імені користувача).

Тепер зловмиснику лише потрібно визначити, чи формат є `<username><delimiter><password>` або `<password><delimiter><username>`. Для цього він може просто **створити кілька користувачів з схожими та довгими юзернеймами та паролями, поки не визначить формат та довжину сепаратора**:

| Username length: | Password length: | Username+Password length: | Cookie's length (after decoding): |
| ---------------- | ---------------- | ------------------------- | --------------------------------- |
| 2                | 2                | 4                         | 8                                 |
| 3                | 3                | 6                         | 8                                 |
| 3                | 4                | 7                         | 8                                 |
| 4                | 4                | 8                         | 16                                |
| 7                | 7                | 14                        | 16                                |

# Експлуатація вразливості

## Видалення блоків

Знаючи формат cookie (`<username>|<password>`), щоб вдати з себе користувача `admin`, створіть нового користувача з ім'ям `aaaaaaaaadmin` і отримайте та розкодуйте cookie:

```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```

Ми можемо побачити патерн `\x23U\xE45K\xCB\x21\xC8`, створений раніше з ім'ям користувача, яке містило лише `a`.\
Потім ви можете видалити перший блок 8B, і ви отримаєте дійсне значення cookie для юзернейма `admin`:

```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```

## Переміщення блоків

У багатьох базах даних пошук `WHERE username='admin';` та `WHERE username='admin ';` _(Зверніть увагу на додаткові пробіли)_ вважається однаковим.

Отже, інший спосіб вдатися з себе користувача `admin` може бути таким:

* Створіть ім'я користувача так, щоб: `len(<username>) + len(<delimiter) % len(block)`. З розміром блоку `8B` можна створити ім'я користувача: `username       `, з сепаратором `|` частина `<username><delimiter>` сформує 2 блоки по 8 байтів кожен.
* Потім створіть пароль, який заповнить точну кількість блоків, що містять юзернейм, якого ви хочете імітувати, та пробіли, наприклад: `admin   `

Кукі цього користувача будуть складатися з 3 блоків: перші 2 - це блоки юзернейма + сепаратор, а третій - пароль (який імітує юзернейм): `username       |admin   `

**Потім просто замініть перший блок останнім, і ви зможете імітувати користувача `admin`: `admin          |username`**

## Джерела

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


