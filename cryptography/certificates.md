# Certificates

<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкого створення та **автоматизації робочих процесів**, які працюють на **найкращих** open-source інструментах.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Що таке сертифікат

**Сертифікат публічного ключа (public key certificate)** — це цифровий ідентифікатор, що використовується в криптографії для підтвердження власності на публічним ключем. Він містить деталі ключа, особисті дані власника (суб'єкт - subject) і цифровий підпис від довіреної сторони (видавець - issuer). Якщо програмне забезпечення довіряє видавцю і підпис є дійсним, можлива безпечна комунікація з власником ключа.

Сертифікати зазвичай видають органи сертифікації [certificate authorities](https://en.wikipedia.org/wiki/Certificate_authority) (CAs) в умовах інфраструктури публічних ключів [public-key infrastructure](https://en.wikipedia.org/wiki/Public-key_infrastructure) (PKI). Інший метод — мережа довіри [web of trust](https://en.wikipedia.org/wiki/Web_of_trust), де користувачі безпосередньо перевіряють ключі один одного. Загальний формат сертифікатів — [X.509](https://en.wikipedia.org/wiki/X.509), який може бути адаптований для конкретних потреб, як описано в RFC 5280.

## x509 - загальні поля

### **Загальні поля в сертифікатах x509**

У сертифікатах x509 кілька **полів** відіграють ключову роль у забезпеченні їхньої дійсності та безпеки. Ось розбивка цих полів:

- **Version Number** позначає версію формату x509.
- **Serial Number** унікально ідентифікує сертифікат у CA, головним чином для відстеження відкликання.
- Поле **Subject**  представляє власника сертифіката, яким може бути машина, особа або організація. Воно включає детальну ідентифікацію, таку як:
  - **Common Name (CN)**: домени, що покриваються сертифікатом.
  - **Country (C)**, **Locality (L)**, **State or Province (ST, S, or P)**, **Organization (O)**, and **Organizational Unit (OU)** надають географічні та організаційні деталі.
  - **Distinguished Name (DN)** включає повну ідентифікацію суб'єкта.
- **Issuer** вказує, хто перевірив і підписав сертифікат, включаючи подібні підполя, як для Суб'єкта, для CA.
- **Validity Period** позначений часовими мітками **Not Before** та **Not After**, забезпечуючи, що сертифікат не використовується до або після певної дати.
- Розділ  **Public Key**, важливий для безпеки сертифіката, вказує алгоритм, розмір та інші технічні деталі публічного ключа.
- Розширення **x509v3** покращують функціональність сертифіката, вказуючи **використання ключа**, **розширене використання ключа**, **альтернативне ім'я суб'єкта** та інші властивості для точної настройки застосування сертифіката.

#### **Використання ключа та розширення**

- **Key Usage** визначає криптографічне використання публічного ключа, як-от цифровий підпис або шифрування ключа.
- **Extended Key Usage** далі уточнює випадки використання сертифіката, наприклад, для аутентифікації сервера TLS.
- **Subject Alternative Name** та **Basic Constraint** визначають додаткові імена хостів, що покриваються сертифікатом, і чи є це сертифікат органу сертифікації або кінцевого суб'єкта відповідно.
- Ідентифікатори, як **Subject Key Identifier** та **Authority Key Identifier**, забезпечують унікальність та можливість відстеження ключів.
- **Authority Information Access** та **CRL Distribution Points** дають шляхи для перевірки видаючого CA та перевірки статусу відкликання сертифіката.
- **CT Precertificate SCTs** надають логи прозорості, що є критично важливими для громадської довіри до сертифіката.

```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
    cert_data = file.read()
    certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```

### **Різниця між OCSP та CRL Distribution Points**

**OCSP** (**RFC 2560**) використовує клієнта та відповідача для перевірки, чи був відкликаний цифровий сертифікат публічного ключа, без необхідності завантажувати повний **CRL**. Цей метод ефективніший, ніж традиційний **CRL**, який надає список серійних номерів відкликаних сертифікатів, але вимагає завантаження потенційно великого файлу. CRL може включати до 512 записів. Детальнішу інформацію можна знайти [тут](https://www.arubanetworks.com/techdocs/ArubaOS%206_3_1_Web_Help/Content/ArubaFrameStyles/CertRevocation/About_OCSP_and_CRL.htm).

### **Що таке прозорість сертифікатів**

Прозорість сертифікатів допомагає боротися з загрозами, пов'язаними з сертифікатами, забезпечуючи видимість видачі та існування SSL сертифікатів для власників доменів, CA та користувачів. Її цілі:

* Запобігання видачі CA SSL сертифікатів для домену без відома власника домену.
* Створення відкритої системи аудиту для відстеження помилково або зловмисно випущених сертифікатів.
* Захист користувачів від шахрайських сертифікатів.

#### **Логи сертифікатів**

Логи сертифікатів — це публічно доступні, тільки для додавання записи сертифікатів, які підтримуються мережевими службами. Ці логи надають криптографічні докази для цілей аудиту. Як органи видачі, так і громадськість можуть подавати сертифікати в ці логи або запитувати їх для перевірки. Хоча точна кількість серверів логів не фіксована, очікується, що їх буде менше тисячі по всьому світу. Ці сервери можуть незалежно управлятися CA, інтернет-провайдерами або будь-яким зацікавленим суб'єктом.

#### **Запит**

Щоб дослідити логи прозорості сертифікатів для будь-якого домену, відвідайте [https://crt.sh/](https://crt.sh).

Існують різні формати зберігання сертифікатів, кожен з яких має свої випадки використання та сумісність. Це охоплює основні формати та надає інструкції щодо конвертації між ними.

## **Формати**

### **Формат PEM**
- Найбільш поширений формат для сертифікатів.
- Вимагає окремих файлів для сертифікатів та приватних ключів, закодованих у Base64 ASCII.
- Звичні розширення: .cer, .crt, .pem, .key.
- Головним чином використовується серверами типу Apache.

### **Формат DER**
- Бінарний формат сертифікатів.
- Не містить текст "BEGIN/END CERTIFICATE", який є в файлах PEM.
- Звичні розширення: .cer, .der.
- Часто використовується на платформах Java.

### **Формат P7B/PKCS#7**
- Зберігається у Base64 ASCII, з розширеннями .p7b або .p7c.
- Містить тільки сертифікати та сертифікати ланцюга, без приватного ключа.
- Підтримується Microsoft Windows і Java Tomcat.

### **Формат PFX/P12/PKCS#12**
- Бінарний формат, що об'єднує сертифікати сервера, проміжні сертифікати та приватні ключі в одному файлі.
- Розширення: .pfx, .p12.
- Головним чином використовується на Windows для імпорту та експорту сертифікатів.

### **Конвертація форматів**

**Конвертації PEM** є важливими для сумісності:

- **x509 в PEM**

```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```


- **PEM в DER**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```


- **DER в PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```

- **PEM в P7B**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```


- **PKCS7 в PEM**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```


**Конвертація PFX** критично важливі для управління сертифікатами на Windows:

- **PFX в PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```


- **PFX в PKCS#8** включає два кроки:
  1. Конвертація PFX в PEM

```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```

  2. Конвертація PEM в PKCS#8
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```


- **P7B в PFX** також вимагає дві команди:
  1. Конвертація P7B в CER
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```

  2. Конвертація CER і приватного ключа в PFX
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```

***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкого створення та **автоматизації робочих процесів**, які працюють на **найкращих** open-source інструментах.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Вивчайте методи зламу AWS з нуля та станьте експертом з курсом</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **компанії з кібербезпеки**? Хочете бачити **рекламу своєї компанії на HackTricks**? чи хочете отримати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Ознайомтеся з [**ПЛАНАМИ ПЕРЕДПЛАТИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе ексклюзивні [NFT](https://opensea.io/collection/the-peass-family) з нашої колекції [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* Отримайте офіційний [**PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтеся до [**💬**](https://emojipedia.org/speech-balloon/) [**Discord групи**](https://discord.gg/hRep4RUj7f) або [**telegram каналу**](https://t.me/peass) чи **підписуйтесь** на мене в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.
* **Поділіться вашими хакерськими фішками, надіславши Pull Request до репозиторію [hacktricks](https://github.com/carlospolop/hacktricks) або [hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
