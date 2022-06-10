
<link rel="stylesheet"
      href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/11.5.1/styles/base16/tomorrow.min.css">
<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/11.5.1/highlight.min.js"></script>
<script>hljs.highlightAll();</script>

**Полезные сслыки:**  
[Установка Tidal Connect](https://audiophilestyle.com/forums/topic/62703-tidal-connect-on-raspberry-how-to/)  
[Сжатие образа](https://askubuntu.com/a/1390007)  

**Команды:**  
TidalConnect

    sudo systemctl start dadiet_nomqa.service
    sudo systemctl stop dadiet_nomqa.service
    sudo systemctl status dadiet_nomqa.service
    sudo systemctl start dadiet_mqa.service
    sudo systemctl stop dadiet_mqa.service
    sudo systemctl status dadiet_mqa.service
    sudo systemctl start dadiet_mqapp.service
    sudo systemctl stop dadiet_mqapp.service
    sudo systemctl status dadiet_mqapp.service

**Системные файлы**

    /lib/systemd/system/dadiet_nomqa.service
    /lib/systemd/system/dadiet_mqa.service
    /lib/systemd/system/dadiet_mqapp.service
