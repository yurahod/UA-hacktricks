

<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# CBC

Якщо файл **кукі (cookie)** містить **лише ім'я користувача (username)** (або перша частина файлу cookie є юзернеймом) і ви хочете видати себе за користувача під іменем "**admin**", то ви можете створити нового користувача з ім'ям "**bdmin**" і **підібрати перший байт** значення кукі.

# CBC-MAC

**Код автентифікації повідомлень з ланцюговим шифруванням блоків** (**CBC-MAC**) - це метод, що використовується в криптографії. Він працює шляхом шифрування повідомлення блок за блоком, де шифрування кожного блоку пов'язане з попереднім. Цей процес створює ланцюг блоків, гарантуючи, що зміна навіть одного біта в оригінальному повідомленні призведе до непередбачуваної зміни в останньому блоку зашифрованих даних. Для внесення або відміни такої зміни потрібен ключ шифрування, що забезпечує захист.

Для обчислення CBC-MAC повідомлення m шифрується в режимі CBC з нульовим IV (вектором ініціалізації) і зберігають останній блок. На наступному рисунку показано обчислення CBC-MAC повідомлення, що складається з блоків![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5), за допомогою секретного ключа k і блокового шифру E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

Зазвичай для CBC-MAC використовують **IV**, що **дорівнює 0**.\
Це проблема, тому що 2 відомі повідомлення (`m1` і `m2`) незалежно створять 2 підписи (`s1` і `s2`). Отже:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Тоді повідомлення, складене з m1 і m2 (m3), створить 2 підписи (s31 і s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Що можна обчислити без знання ключа шифрування.**

Уявіть, що ви шифруєте ім'я `Administrator` блоками по `8 байт`:

* `Administ`
* `rator\00\00\00`

Ви можете створити ім'я користувача **Administ** (m1) і отримати підпис (s1).\
Потім ви можете створити ім'я користувача, результатом `rator\00\00\00 XOR s1`. Це створить `E(m2 XOR s1 XOR 0)`, що є s32.\
Тепер ви можете використовувати s32 як підпис повного імені `Administrator`.

### Підсумок

1. Отримати підпис імені користувача **Administ** (m1), що є s1.
2. Отримати підпис імені користувача **rator\x00\x00\x00 XOR s1 XOR 0**, що є s32.
3. Встановити значення кукі як s32, і воно буде дійсним для користувача **Administrator**.

# Атака з контролем IV

Якщо ви маєте можливість контролювати використовуваний IV, атака може бути дуже простою.\
Якщо cookie містить лише зашифроване ім'я користувача, ви можете створити нового користувача "**Administrator**" і отримати його cookie, щоб видати себе за "**administrator**". Тепер, контролюючи IV, ви можете змінити перший байт IV так, що **IV\[0] XOR "A" == IV'\[0] XOR "a"** і згенерувати нове значення cookie для користувача "**Administrator**". Ці cookie будуть дійсними для **імітації** користувача **administrator** з початковим **IV**.

## Джерела

Більше деталей в [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


