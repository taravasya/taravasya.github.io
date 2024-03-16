**Требования:**

* KeenOS версия 4.х
* Развёрнутая среда Entware,
* Рабочее VPN-соединение поверх провайдерского, по которому будет идти обращение к определённым доменам, с включённой опцией **"Использовать для выхода в интернет"**
* В настройках роутера, в разделе "Приоритеты подключений", должна быть создана отдельная политика(запомните её название, оно понадобится позже), в ней должно быть включено рабочее VPN-соединение и обязательно исключено основное подключение.
* Для работы через ipv6 - наличие рабочего ipv6 на основном подключении и на VPN-соединении
 
 ---

**Установка пакетов:**

В терминале выполните команду:

    opkg install adguardhome-go ipset iptables ip-full curl jq

В CLI роутера выполните команды:

    opkg dns-override
    system configuration save

(далее все команды должны будут выполняться в теримнале)

В этот момент Entware-сервисы будут перезапущены, а интерфейс для первоначальной настройки AGH станет доступен по адресу:

    http://ваш-ip-роутера:3000

По этому адресу надо перейти в настройки AGH и выполнить первоначальные настройки с помощью маcтера настроек AGH. В первом окне настроек, смените 80й порт на 3000 и жмите далее. Все остальные настройки используемые по умолчанию подойдут для большинства случаев.

 --- 

**Скрипты:**

Создайте файл **/opt/etc/init.d/S52ipset** со следующим содержимым:

    #!/bin/sh
    
    PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    
    if [ "$1" = "start" ]; then
        ipset create bypass hash:ip
        ipset create bypass6 hash:ip family inet6
    fi


Создайте файл **/opt/etc/ndm/netfilter.d/010-bypass.sh** для ipv4  со следующим содержимым:

    #!/bin/sh
    
    [ "$type" == "ip6tables" ] && exit
    [ "$table" != "mangle" ] && exit
    [ -z "$(ipset --quiet list bypass)" ] && exit
    
    if [ -z "$(iptables-save | grep bypass)" ]; then
         mark_id=`curl -kfsS http://localhost:79/rci/show/ip/policy 2>/dev/null | jq -r '.[] | select(.description == "policy_vpn") | .mark'`
         iptables -w -t mangle -A PREROUTING ! -i nwg0 -m conntrack --ctstate NEW -m set --match-set bypass dst -j CONNMARK --set-mark 0x$mark_id
         iptables -w -t mangle -A PREROUTING ! -i nwg0 -m set --match-set bypass dst -j CONNMARK --restore-mark
    fi

Где:  
**nwg0** - это сетевой интерфейс VPN-соединения для выборочного обхода блокировок.  
**policy_vpn** - это созданная политика в которой был включено VPN-соединение

Создайте файл **/opt/etc/ndm/netfilter.d/011-bypass6.sh** для ipv6 со следующим содержимым:

    #!/bin/sh
    
    [ "$type" != "ip6tables" ] && exit
    [ "$table" != "mangle" ] && exit
    [ -z "$(ipset --quiet list bypass6)" ] && exit
    
    if [ -z "$(ip6tables-save | grep bypass6)" ]; then
         mark_id=`curl -kfsS http://localhost:79/rci/show/ip/policy 2>/dev/null | jq -r '.[] | select(.description == "policy_vpn") | .mark'`
         ip6tables -w -t mangle -A PREROUTING ! -i nwg0 -m conntrack --ctstate NEW -m set --match-set bypass6 dst -j CONNMARK --set-mark 0x$mark_id
         ip6tables -w -t mangle -A PREROUTING ! -i nwg0 -m set --match-set bypass6 dst -j CONNMARK --restore-mark
    fi

Где:  
**nwg0** - это сетевой интерфейс VPN-соединения для выборочного обхода блокировок.   
**policy_vpn** - это созданная политика в которой был включено VPN-соединение

Что бы узнать название своего интерфейса, используйте команду:

    ip addr

Сделайте скрипты исполняемыми:

    chmod +x /opt/etc/init.d/S52ipset
    chmod +x /opt/etc/ndm/netfilter.d/010-bypass.sh
    chmod +x /opt/etc/ndm/netfilter.d/011-bypass6.sh
    
---

**Список доменов для обхода блокировок:**

Найдите в конфигурационном файле AGH **/opt/etc/AdGuardHome/AdGuardHome.yaml** строчку

    ipset_file: ""

и замените на

    ipset_file: /opt/etc/AdGuardHome/ipset.conf

Создайте файл **/opt/etc/AdGuardHome/ipset.conf**
Он будет единственным, требующим редактирования время от времени, в зависимости от изменения вашего персонального списка доменов для разблокировки.
Он имеет следующий синтаксис:

    intel.com,ipinfo.io/bypass,bypass6
    instagram.com,cdninstagram.com/bypass,bypass6
    epicgames.com,gog.com/bypass,bypass6

Если не будет использоваться ipv6, то:

    intel.com,ipinfo.io/bypass
    instagram.com,cdninstagram.com/bypass
    epicgames.com,gog.com/bypass

Т.е. в левой части через запятую указаны домены, требующие обхода блокировок, справа после слэша — название ipset, в который AGH складывает результаты разрешения DNS-имён. Можно указать всё в одну строчку, можно разделить логически на несколько строк как в примере. Домены третьего уровня и выше также включаются в обход блокировок, т.е. указание intel.com включает www.intel.com, download.intel.com и пр.
Рекомендую добавить какой-нибудь «сигнальный» сервис, показывающий ваш текущий IP-адрес (ipinfo.io в примере). Так вы сможете проверить работоспособность настроенного решения. Учтите, что AGH не перечитывает изменённый файл, поэтому после правки требуется перезапускать его с помощью команды:

    /opt/etc/init.d/S99adguardhome restart


По завершению всех настроек, требуется перезагрузить роутер.
После перезагрузки роутера проверьте в веб-интерфейсе Системный журнал, в нём не должно быть красных строк, связанных с настроенными скриптами.

---

**Диагностика проблем:**

Убедитесь в том, что набор ipset создан и наполняется в процессе работы:

    # Проверка  ipv4
    ipset --list bypass
    # Проверка  ipv6
    ipset --list bypass6

Вывод должен быть не пустой.

Посмотрите, существуют ли правила netfilter для пометки пакетов:

    # Проверка  ipv4
    iptables-save | grep bypass
    # Проверка  ipv6
    ip6tables-save | grep bypass6
В
