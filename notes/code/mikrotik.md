<H1><center>
Mikrotik: Раздельное туннелирование доменов и всех их поддоменов
</center></H1>

Диапазон локальной сети: 192.168.0.0/24  
Имя интерфейса VPN-подключения: vpn1-out

При создании интерфейса VPN-подключения в обязательном порядке снять флаг Add Default Route. Нам не требуется направлять весь трафик в VPN-канал, а маршрутизацию мы настроим позже вручную.
Так же здесь необходимо выбрать опцию:

```
Dial Out -> Use Peer DNS -> Yes
```

Создать таблицу меток пакетов:

```
/routing table add name=vpn1_mark fib
```

Для того, чтобы клиенты локальной сети могли выходить в интернет через VPN-подключение, следует настроить NAT:

```
/ip firewall nat add action=masquerade chain=srcnat out-interface=vpn1-out src-address=192.168.0.0/24
```

Для маркировки пакетов которые необходимо направлять в VPN-туннель: 

```
/ip firewall mangle add action=mark-routing chain=prerouting dst-address-list=vpn1_list new-routing-mark=vpn1_mark passthrough=no
```

Маршрутизация маркированных пакетов:

```
/ip route add distance=1 gateway=vpn1-out routing-table=vpn1_mark
```

Для добавления нужных доменов со всеми их поддоменами в список доменов для которых требуется обход через VPN-туннель:

```
/ip dns static add name=somedomain.com type=FWD forward-to=8.8.8.8 address-list=vpn1_list match-subdomain=yes
```

Вместо 8.8.8.8 можно использовать ip адрес другого доступного DNS сервера.

IP-адреса в address-list будут жить, пока живёт запись в /ip/dns/cache, а там она живёт, пока не истечёт её TTL.  
Обязательно надо куда-то перенаправлять DNS запрос. Нельзя просто без перенаправления складывать в address-list.  
Похоже, что DNS сервера в настройках роутера и те на которые форвардятся пакеты в /ip dns static - должны быть разными.  
Ещё нельзя использовать вместе с DoH. Таким образом, если ваш провайдер блокирует/подменяет DNS-запросы можно:  
* Перенаправлять DNS-запросы в туннель. Нужен настроенный резолвер на IP-адресе гейта VPN-сервера или в его подсети.  
* Использовать отдельный DNS-сервер, который развёрнут рядом с роутером.  