**Правки в формах**
Вот этот код в actionscript-ах форм:

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

Можно заменить для унификации, (а можно и оставить как есть) на:

    _discountid = define_discountid();

Работать будет и так и так. Но если потом будешь переносить код из форм, то лучше, конечно, добавить одну эту строчку.

Для того, чтобы после смены типа печати правильно работал пересчёт скидки, нужно:
В actionscript поля *Способ печати*, добавить это:

    if (iscanadmin == 1 && !bfDeactivateField["ff_nm_email[]"]) ff_UserId_action( ff_getElementByName('ff_nm_UserId[]'), 'change');
    if (isguest == 1 && !bfDeactivateField["ff_nm_email[]"]) ff_email_action( ff_getElementByName('ff_nm_email[]'), 'change');

Ты в форме Евробуклет, добавил только первую строчку. Наверное, во все остальные формы тоже. В евробуклете я дописал вторую строчку.

Так-же для подтягивания данных, после изменений в поле email, надо добавить для него actionscript - custom - change:

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

В Initialization Script, сразу после: 

     $user = JFactory::getUser();
    Надо добавить:
    if ($user->guest) {
        echo "var isguest = 1;";
    } else {
        echo "var isguest = 0;";
    }

Хотя тут получается, что ситуация сродни с функцией **define_discountid();**, когда одно и тоже повторяется во всех инициализациях форм. Стоит подумать над тем, чтобы и этот кусок кода перенести в какой-нибудь файл, для того, чтоб не повторять его везде. Тут правда чуть сложнее, потому, что это не js, а php, который всунут в js. Всё сложно... ))). Суть в том, что в js работающий локально, невозможно добавить php код(ну думаю уже понимаешь разницу между ними). А если быть до конца точным, то на самом деле, **Initialization Script** - это PHP, который посредством сложных манипуляций, "инъецирует" страницу js кодом... В общем, сложно, но решаемо,.. и потому надо обмозговать, как это правильно сделать.