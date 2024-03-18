

<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# Ланцюгове шифрування блоків (CBC - Cipher Block Chaining)

У режимі CBC **попередній зашифрований блок використовується як IV (вектор ініціалізації)** для XOR з наступним блоком:

![https://defuse.ca/images/cbc\_encryption.png](https://defuse.ca/images/cbc\_encryption.png)

Для розшифрування в режимі CBC виконуються **протилежні операції**:

![https://defuse.ca/images/cbc\_decryption.png](https://defuse.ca/images/cbc\_decryption.png)

Зверніть увагу, що потрібно використовувати **ключ шифрування** та **IV**.

# Доповнення повідомлення (Message Padding)

Оскільки шифрування виконується у **блоках фіксованого розміру**, зазвичай потрібно **доповнення (padding)** в останньому блоку для завершення його довжини.\
Зазвичай використовується **PKCS7**, який генерує такі доповнення, **повторюючи число байтів**, **необхідних для заповнення** блоку. Наприклад, якщо в останньому блоку бракує 3 байти, доповнення буде `\x03\x03\x03`.

Давайте розглянемо більше прикладів з **2 блоками довжиною 8 байтів**:

| byte #0 | byte #1 | byte #2 | byte #3 | byte #4 | byte #5 | byte #6 | byte #7 | byte #0  | byte #1  | byte #2  | byte #3  | byte #4  | byte #5  | byte #6  | byte #7  |
| ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | 6        | **0x02** | **0x02** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | **0x03** | **0x03** | **0x03** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | **0x05** | **0x05** | **0x05** | **0x05** | **0x05** |
| P       | A       | S       | S       | W       | O       | R       | D       | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** |

Зверніть увагу на останній приклад, де **останній блок був повним, тому був згенерований ще один блок лише з доповненням**.

# Padding Oracle

Коли застосунок розшифровує зашифровані дані, спочатку він розшифровує дані, а потім видаляє доповнення. Під час очищення доповнення, якщо **неправильне доповнення викликає поведінку, яку можна виявити чи передбачити**, у вас є **вразливість padding oracle**. Поведінкою, яку можна виявити чи ідентифікувати, може бути **помилка**, **відсутність результатів**, або **повільніша чи швидша відповідь**.

Якщо ви виявили таку поведінку, ви можете **розшифровувати зашифровані дані** та навіть **шифрувати будь-який текст**.

## Експлуатація вразливості

Ви можете скористатися [https://github.com/AonCyberLabs/PadBuster](https://github.com/AonCyberLabs/PadBuster) для подальшої експлуатації цієї вразливості. Інструмен ще можна встановити через `apt-get`:

```
sudo apt-get install padbuster
```

Щоб перевірити, чи вразливий cookie сайту, ви можете спробувати:

```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA=="
```

**Encoding 0** означає, що використовується **base64** (але доступні й інші, перевірте меню допомоги `-h`).

Ви також можете **скористатися цією вразливістю для шифрування нових даних**. Наприклад, уявіть, що вміст cookie є "_**user=MyUsername**_", тоді ви можете змінити його на "_**user=administrator**_" і підвищити привілеї в межах застосунку. Ви також можете зробити це, використовуючи `paduster`, вказавши параметр `-plaintext`:

```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA==" -plaintext "user=administrator"
```

Якщо сайт вразливий, `padbuster` автоматично спробує знайти, коли стається помилка доповнення, але ви також можете вказати повідомлення про помилку, використовуючи параметр `-error`.

```bash
perl ./padBuster.pl http://10.10.10.10/index.php "" 8 -encoding 0 -cookies "hcon=RVJDQrwUdTRWJUVUeBKkEA==" -error "Invalid padding"
```

## Теорія

**Узагальнено**, ви можете почати розшифровувати зашифровані дані, вгадуючи правильні значення, які можна використовувати для створення всіх **різних доповнень**. Тоді атака padding oracle почне розшифровувати байти з кінця до початку, вгадуючи, яке буде правильне значення, яке **створює доповнення 1, 2, 3 і т.д.**

![](<../.gitbook/assets/image (629) (1) (1).png>)

Уявіть, що у вас є деякий зашифрований текст, який займає **2 блоки**, сформовані байтами **від E0 до E15**.\
Для того, щоб **розшифрувати останній блок** (**E8** до **E15**), весь блок проходить через "розшифрування блокового шифру" ("block cipher decryption"), генеруючи **проміжні байти I0 до I15**.\
Нарешті, на кожен проміжний байт з попередніми зашифрованими байтами (E0 до E7) застосовується **XOR** функція. Отже:

* `C15 = D(E15) ^ E7 = I15 ^ E7`
* `C14 = I14 ^ E6`
* `C13 = I13 ^ E5`
* `C12 = I12 ^ E4`
* ...

Тепер можливо **модифікувати `E7` до тих пір, поки `C15` є `0x01`**, що також буде коректним доповненням. Отже, у цьому випадку: `\x01 = I15 ^ E'7`

Таким чином, знаючи `E'7`, **можливо розрахувати I15**: `I15 = 0x01 ^ E'7`

Що дозволяє нам **розрахувати C15**: `C15 = E7 ^ I15 = E7 ^ \x01 ^ E'7`

Знаючи **C15**, тепер можливо **визначити C14**, але цього разу шляхом перебору з доповненням `\x02\x02`.

Складність цього перебору є така ж, як і попереднього, оскільки можливо визначити `E''15`, значення якого є 0x02: `E''7 = \x02 ^ I15`, тому потрібно лише знайти **`E'14`**, який створює **`C14` рівне `0x02`**.
Потім повторіть ті ж кроки для розшифровки **`C14: C14 = E6 ^ I14 = E6 ^ \x02 ^ E''6`**

**Продовжуйте цей ланцюжок, поки не розшифруєте весь зашифрований текст.**

## Виявлення вразливості

Зареєструйте обліковий запис і увійдіть в систему з цим обліковим записом.\
Якщо ви **входите багато разів** і завжди отримуєте **однакові кукі**, ймовірно, **щось не так** з застосунком. **Кукі мають бути унікальними кожного разу**, коли ви входите в систему. Якщо кукі **завжди однакові**, вони, ймовірно, завжди будуть дійсними, і **не буде способу їх інвалідувати**.

Now, if you try to **modify** the **cookie**, you can see that you get an **error** from the application.\
But if you BF the padding (using padbuster for example) you manage to get another cookie valid for a different user. This scenario is highly probably vulnerable to padbuster.
Тепер, якщо ви спробуєте **змінити кукі**, ви можете побачити, що отримуєте **помилку** від застосунку.
Але якщо ви здійсните перебір доповнення (використовуючи, наприклад, padbuster), ви можете отримати інші кукі, які дійсні для іншого користувача. Цей сценарій, ймовірно, вразливий до padding oracle. Експлуатація, потенційно, можлива через padbuster.

## References

* [https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation](https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation)


<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


