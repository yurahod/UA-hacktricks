

<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# Стислий опис атаки 

Уявіть сервер, що **підписує** деяку **інформацію**, **додаючи секретний ключ** до деяких незашифрованих текстових даних, а потім хешуючи ці дані. Якщо ви знаєте:

* **Довжину секретного ключа** (це також можна визначити перебором за заданим діапазоном)
* **Незашифровані текстові дані**
* **Алгоритм (і він вразливий до цієї атаки)**
* **Доповнення (padding) є відомим**
  * Зазвичай використовується стандартний padding, тому якщо виконані інші 3 умови, це також враховується
  * Це доповнення може змінюватись, бо залежить від довжини секретного ключа + даних, що шифруються, тому важливо знати довжину цього самого ключа

Тоді **атакер** може **додати дані** і **згенерувати** дійсний **підпис** для **попередніх даних + доданих даних**.

## Як це?

По суті, вразливі алгоритми генерують хеші, спочатку **хешуючи блок даних**, а потім, виходячи **з раніше створеного хешу** (стану), **додають наступний блок даних** і **хешують його**.

Тоді уявіть, що секретний ключ це "secret" і дані це "data", а "secretdata" в MD5 є 6036708eba0d11f6ef52ad44e8b74d5b.\
Якщо атакуючий хоче додати текст "append", він може:

* Згенерувати MD5 з 64 символами "A"
* Змінити стан раніше ініціалізованого хешу на 6036708eba0d11f6ef52ad44e8b74d5b
* Додати текст "append"
* Завершити хеш, і, в результаті, хеш буде **дійсним для "secret" + "data" + "padding" + "append"**

## **Інструменти**

{% embed url="https://github.com/iagox86/hash_extender" %}

## Джерела

Цю атаку добре пояснено на [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)


<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


