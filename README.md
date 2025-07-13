# Домашнее задание к занятию "`Уязвимости и атаки на информационные системы`" - `Заярченко И.Я.`



### Задание 1
Скачайте и установите виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/.

Это типовая ОС для экспериментов в области информационной безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту виртуальную машину, используя nmap.

Попробуйте найти уязвимости, которым подвержена эта виртуальная машина.

Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

Какие сетевые службы в ней разрешены?
Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)
Приведите ответ в свободной форме.


### Ответ 1 

1. Сканирование сети (Nmap) виртуальной машины Metasploitable
![nmap -sV -A](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/1.jpg)
![nmap -sV -A](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/12.jpg)

В результате сканирования (nmap -sV -A 192.168.88.26) были выявлены следующие открытые порты и службы:

21/tcp – FTP (vsftpd 2.3.4) – Разрешён анонимный вход (Anonymous FTP login allowed).

22/tcp – SSH (OpenSSH 4.7p1) – Устаревшая версия, возможны уязвимости.

23/tcp – Telnet – Позволяет удалённое управление (небезопасно).

25/tcp – SMTP (Postfix) – Возможны атаки на почтовый сервер.

53/tcp – DNS (ISC BIND 9.4.2) – Уязвимый DNS-сервер.

80/tcp – HTTP (Apache 2.2.8) – Веб-сервер с возможными уязвимостями.

139/tcp, 445/tcp – Samba (smbd 3.0.20-Debian) – Уязвимый файловый сервер.

1524/tcp – Bindshell (Metasploitable root shell) – Опасный открытый шелл.

3306/tcp – MySQL (5.0.51a-3ubuntu5) – Устаревшая СУБД с известными дырами.

5432/tcp – PostgreSQL (8.3.0-8.3.7) – Уязвимая версия базы данных.

5900/tcp – VNC (без пароля?) – Возможен удалённый доступ.

6667/tcp – IRC (UnrealIRCd 3.2.8.1) – Содержит бэкдор.

8009/tcp, 8180/tcp – Apache Tomcat – Возможны RCE-уязвимости.

Уязвимость в vsftpd 2.3.4 (Backdoor Command Execution, CVE-2011-2523)
Описание: Встроенный бэкдор позволяет выполнить произвольные команды.

Exploit-DB: https://www.exploit-db.com/exploits/49757


Уязвимость в UnrealIRCd 3.2.8.1 (Backdoor RCE, CVE-2010-2075)
Описание: Встроенный бэкдор позволяет выполнить команды через IRC.

Exploit-DB: https://www.exploit-db.com/exploits/13853


Уязвимость в Samba 3.0.20 (CVE-2007-2447, Usermap Script RCE)
Описание: Позволяет выполнить команды через неправильную обработку имени пользователя.

Exploit-DB: https://www.exploit-db.com/exploits/16320


---

### Задание 2
Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP.

Запишите сеансы сканирования в Wireshark.

Ответьте на следующие вопросы:

Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
Как отвечает сервер?
Приведите ответ в свободной форме.

#### SYN-сканирование (-sS)
``` sudo nmap -sS 192.168.88.26 ```

Как работает: Отправляет SYN-пакет, но не завершает рукопожатие (не отправляет ACK).

Трафик в Wireshark:

Клиент → SYN

Сервер → SYN-ACK (если порт открыт) или RST-ACK (если закрыт).

Клиент не отвечает (ACK не отправляется).

Ответ сервера:

Открытый порт → SYN-ACK.

Закрытый порт → RST-ACK.

![SYN](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/21.jpg))
![SYN](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/22.jpg))


##### FIN-сканирование (-sF)
``` sudo nmap -sF 192.168.88.26 ```
Как работает: Отправляет FIN-пакет (как будто соединение уже закрыто).

Трафик в Wireshark:

Клиент → FIN

Сервер → RST (если порт закрыт) или никак (если открыт).

Ответ сервера:

Открытый порт → молчит (нет ответа).

Закрытый порт → RST.

![FYN](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/23.jpg))
![FYN](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/24.jpg))


#### Xmas-сканирование (-sX)
``` sudo nmap -sX 192.168.88.26 ```

Как работает: Отправляет пакет с FIN, URG, PUSH флагами (как "рождественская ёлка").

Трафик в Wireshark:

Клиент → FIN+URG+PSH

Сервер → RST (закрыт) или молчит (открыт).

Ответ сервера:

Аналогично FIN-сканированию.

![Xmas](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/25.jpg))
![Xmas](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/26.jpg))


#### UDP-сканирование (-sU)
``` sudo nmap -sU 192.168.88.26 ```

Как работает: Отправляет UDP-пакет (без установки соединения).

Трафик в Wireshark:

Клиент → UDP-запрос (например, на порт 53 для DNS).

Сервер → UDP-ответ (если порт открыт) или ICMP "Port Unreachable" (если закрыт).

Ответ сервера:

Открытый UDP-порт → ответ (например, DNS-ответ).

Закрытый UDP-порт → ICMP type 3 (Destination Unreachable).

![Xmas](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/27.jpg))
![Xmas](https://github.com/vonoid/attacks-on-IS/blob/a4918e4703cc4dbcb5d7f4f6af1c54341d26b852/28.jpg))


