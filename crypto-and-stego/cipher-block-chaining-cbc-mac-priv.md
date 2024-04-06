<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>


# CBC

Якщо **cookie** - це **тільки** **ім'я користувача** (або перша частина cookie - це ім'я користувача) і ви хочете видачу себе за користувача "**адміністратор**". Тоді ви можете створити ім'я користувача **"bdmin"** і **провести брутфорс першого байта** cookie.

# CBC-MAC

**Код аутентифікації повідомлення з шифруванням блоків за допомогою ланцюга блоків** (**CBC-MAC**) - це метод, що використовується в криптографії. Він працює шляхом шифрування повідомлення блок за блоком, де шифрування кожного блоку пов'язане з попереднім. Цей процес створює **ланцюжок блоків**, забезпечуючи, що зміна навіть одного біта в оригінальному повідомленні призведе до непередбачуваної зміни в останньому блоку зашифрованих даних. Для внесення або скасування такої зміни потрібний ключ шифрування, що забезпечує безпеку.

Для обчислення CBC-MAC повідомлення m, його шифрують в режимі CBC з нульовим вектором ініціалізації та зберігають останній блок. Наступна фігура наводить обчислення CBC-MAC повідомлення, що складається з блоків![https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) за допомогою секретного ключа k та блочного шифру E:

![https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Вразливість

З CBC-MAC зазвичай використовується **IV рівний 0**.\
Це проблема, оскільки 2 відомі повідомлення (`m1` та `m2`) незалежно генерують 2 підписи (`s1` та `s2`). Таким чином:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Тоді повідомлення, що складається з m1 та m2, об'єднані (m3), згенерують 2 підписи (s31 та s32):

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Це можливо обчислити без знання ключа шифрування.**

Уявіть, що ви шифруєте ім'я **Адміністратор** у блоках **8 байтів**:

* `Administ`
* `rator\00\00\00`

Ви можете створити ім'я користувача **Administ** (m1) та отримати підпис (s1).\
Потім ви можете створити ім'я користувача, яке дорівнює результату `rator\00\00\00 XOR s1`. Це згенерує `E(m2 XOR s1 XOR 0)`, яке є s32.\
Тепер ви можете використовувати s32 як підпис повного імені **Адміністратор**.

### Резюме

1. Отримайте підпис імені користувача **Administ** (m1), який дорівнює s1
2. Отримайте підпис імені користувача **rator\x00\x00\x00 XOR s1 XOR 0**, який дорівнює s32**.**
3. Встановіть cookie на s32 і він буде дійсним cookie для користувача **Адміністратор**.

# Керування атакою IV

Якщо ви можете контролювати використаний IV, атака може бути дуже простою.\
Якщо cookie - це просто зашифроване ім'я користувача, для видачі себе за користувача "**адміністратор**" ви можете створити користувача "**Адміністратор**" і отримати його cookie.\
Тепер, якщо ви можете контролювати IV, ви можете змінити перший байт IV так, що **IV\[0] XOR "A" == IV'\[0] XOR "a"** і згенерувати cookie для користувача **Адміністратор**. Цей cookie буде дійсним для **видачі себе** за користувача **адміністратор** з початковим **IV**.

## Посилання

Додаткова інформація за посиланням [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>