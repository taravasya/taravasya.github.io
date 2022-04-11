
<link rel="stylesheet"
      href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/11.5.1/styles/monokai.min.css">
<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/11.5.1/highlight.min.js"></script>
<script>hljs.highlightAll();</script>

**Правки в коде:** 

В *Scripts / Initialization Script*, в php секции должен быть такой код: 

     <?php
        $user = JFactory::getUser();
            if ($user->guest) {
                echo "var isguest = 1;";
             } else {
            echo "var isguest = 0;";
            }
        $is_admin = array_search('7', $user->groups);
        $is_super = array_search('8', $user->groups);
        $is_manager = array_search('6', $user->groups);
        if( $is_admin OR $is_super OR $is_manager)
        {
            echo
            "
            var iscanadmin = 1;
            function mytoggler()
            {
                bfToggleFields('on', 'section', 'sId', bfDeactivateField);
            }
            ";
        }
        else
        {
            echo
            "
            var iscanadmin = 0;
            function mytoggler()
            {
                bfToggleFields('off', 'section', 'sId', bfDeactivateField);
            }
            ";
        }
    ?>


Здесь определяется:   

* текщий пользователь: гость/не гость   
* текщий пользователь: имеет ли право получать раширенные данные из БД, при заполнении/изменении email  
* включается/выключается видимость поля **id ПОЛЬЗОВАТЕЛЯ**  
    

Хотя тут получается, что ситуация сродни с функцией **define_discountid();**, когда один и тот же код повторяется во всех инициализациях форм.   

Стоит подумать над тем, чтобы и этот кусок кода перенести в какой-нибудь файл, для того, чтоб не повторять его везде. 
Тут правда есть сложность, потому, что это не js, а php, который всунут в js. Всё сложно... ))) но решаемо,.. и потому надо обмозговать, как это правильно сделать.

---

В *FORM PIECES / AFTER FORM*

    return '
    <script>
         JQuery(document).ready(function()
         {
              mytoggler();
         });
    </script>
    ';


---

Создать поле: Id ПОЛЬЗОВАТЕЛЯ, и в его настройках:
value, добвить код:
    
    <?php $user = JFactory::getUser();return $user->id;?>

А в *Advanced / Actionscript / Custom / Change*
добавить такой код:

    function ff_UserId_action(element, action)
    {
        switch (action) {
            case 'change':
            _discountid = define_discountid();
            jQuery.ajax({
                    async: false,
                    type: "POST",
                    dataType: 'json',
                    url: "<?php return JURI::root(true ); ?>/templates/pm/php/get-client-data.php",
                    data: { id: element.value, discountid: _discountid },    
                    success: function(json) {
                        ff_getElementByName('ff_nm_LastName[]').value = json['lastName'];
                        ff_getElementByName('ff_nm_UserName[]').value = json['name'];
                        ff_getElementByName('ff_nm_phone[]').value = json['phone'];
                        ff_getElementByName('ff_nm_email[]').value = json['email'];
                        ff_getElementByName('ff_nm_Discont[]').value = json['discount'];
                    }
            });
            break;
            default:;
        }
    }

Я так понимаю, ты это везде уже сделал.

---

Вот этот код в actionscript-ах поля **Id ПОЛЬЗОВАТЕЛЯ**:

    switch (ff_getElementByName('ff_nm_Category[]').value) {
        case 'Цифровая печать':
        case 'Цифровий друк':
            _discountid = 20
            break;
        case 'Офсетная печать':
        case 'Офсетний друк':
            _discountid = 21
            break;
        case 'Широкоформатная печать':
        case 'Широкоформатний друк':
            _discountid = 22
            break;
        case 'УФ печать':
        case 'УФ друк':
            _discountid = 23
            break;
        case 'Сувенирная продукция':
        case 'Сувенірна продукція':
            _discountid = 24
            break;
        default:;
    }

Можно заменить для унификации, (а можно и оставить как есть) на вызов функции:

    _discountid = define_discountid();

Работать будет и так и так. Но если потом будешь переносить код из форм, то лучше, конечно, добавить одну эту строчку.

---

Для того, чтобы **после смены типа печати** правильно работал пересчёт скидки, нужно:
В actionscript селекта *Способ печати*, добавить это:

    if (iscanadmin == 1 && !bfDeactivateField["ff_nm_email[]"]) ff_UserId_action( ff_getElementByName('ff_nm_UserId[]'), 'change');
    if (isguest == 1 && !bfDeactivateField["ff_nm_email[]"]) ff_email_action( ff_getElementByName('ff_nm_email[]'), 'change');

Ты в форме Евробуклет, добавил только первую строчку. Наверное, во все остальные формы тоже. В евробуклете я дописал вторую строчку.

---
Для подтягивания данных, после изменений в **поле email**, надо добавить для него в  
*actionscript / custom / change*  
такой код:  

    function ff_email_action(element, action)
    {
        switch (action) {
            case 'change':
            _userid = ff_getElementByName('ff_nm_UserId[]').value;
            jQuery.ajax({
                    async: false,
                    type: "POST",
                    dataType: 'json',
                    url: "<?php return JURI::root(true ); ?>/templates/pm/php/get-client-data.php",
                    data: { find_email: element.value, discountid: 20 },    
                    success: function(json) {
                        if (json['lastName']) ff_getElementByName('ff_nm_LastName[]').value = json['lastName'];
                        if (iscanadmin) ff_getElementByName('ff_nm_LastName[]').value = json['lastName'];
                        if (json['name']) ff_getElementByName('ff_nm_UserName[]').value = json['name'];
                        if (iscanadmin) ff_getElementByName('ff_nm_UserName[]').value = json['name'];
                        if (json['phone']) ff_getElementByName('ff_nm_phone[]').value = json['phone'];
                        if (iscanadmin) ff_getElementByName('ff_nm_phone[]').value = json['phone'];
                        if (json['email']) ff_getElementByName('ff_nm_email[]').value = json['email'];
                        if (iscanadmin) ff_getElementByName('ff_nm_email[]').value = json['email'];
                        if (json['discount']) ff_getElementByName('ff_nm_Discont[]').value = json['discount'];
                        if (iscanadmin) ff_getElementByName('ff_nm_Discont[]').value = json['discount'];
                        if (json['userid']) ff_getElementByName('ff_nm_UserId[]').value = json['userid'];
                        if (iscanadmin) ff_getElementByName('ff_nm_UserId[]').value = json['userid'];
                    }
            });
            break;
            default:;
        }
    }
