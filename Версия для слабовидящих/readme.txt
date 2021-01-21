1. Импортируем базовые стили версии для слобовидящих в template.less. Файл стилей для слабовидящих папке с файлом readme.txt

-------------------
2. Добавляем верску панели переключателей для слабовидящих в body сразу после открытия тега (верстка должна быть на самом верху)

<div class="top-panel-block">
        <section>
            <div class="cecutient top-panel">
                <div class="font-size">
                    <span>Размер шрифта:</span>
                    <a href="javascript:;" class="size normal-size active" data-size="16">a</a>
                    <a href="javascript:;" class="size middle-size" data-size="18">a</a>
                    <a href="javascript:;" class="size big-size" data-size="20">a</a>
                </div>
                <div class="site-color">
                    <span>Цвет сайта:</span>
                    <a href="javascript:;" class="color white-color active" data-color="white">ц</a>
                    <a href="javascript:;" class="color black-color" data-color="black">ц</a>
                    <a href="javascript:;" class="color blue-color" data-color="blue">ц</a>
                </div>
                <div class="real-version">
                    <a href="javascript:;" class="cecutient_link is_on"> Обычная версия сайта</a>
                </div>
            </div>
        </section>
    </div>

------------------
3. Для кнопки "Версия для слабовидящих" добавляем классы и индекс для кнопки переключения версий как в этом примере

<a id="cecutient_link" class="menu__link cecutient_link cecutient-hidden right" href="javascript:;">
                <img src="/images/icon/eye.svg" alt="eye">Версия для слабовидящих
            </a>

------------------
4. Запускаем куки в теге head для сохранения

<?php Yii::app()->clientScript->registerCoreScript('cookie');?>

------------------
5. Добавляем скрипты для динамического изменения в срипт папки /themes

var fontSize = function (size) {
        $("body").css('font-size', size+'px');
        $(".size").removeClass("active");
        $(".size[data-size="+size+"]").addClass("active");
    };

    if($.cookie('font-size')) {
        fontSize($.cookie('font-size'), {expires: null, path: '/'});
    }

    $(".size").click(function() {
        $.cookie('font-size', $(this).data('size'), {expires: null, path: '/'});
        fontSize($(this).data('size'));
    });

    var pageColor = function (color) {
        $("body").removeClass("white");
        $("body").removeClass("blue");
        $("body").removeClass("black");
        $("body").addClass(color);
        $(".color").removeClass("active");
        $(".color[data-color="+color+"]").addClass("active");
    };

    if($.cookie('page-color')) {
        pageColor($.cookie('page-color'), {expires: null, path: '/'});
    }
    else {
        $.cookie('page-color', 'white', {expires: null, path: '/'});
        pageColor("white");
    }

    $(".color").click(function() {
        $.cookie('page-color', $(this).data('color'), {expires: null, path: '/'});
        pageColor($(this).data('color'));
    });

    

    $(".cecutient_link").click(function(){
        
        $("body").toggleClass("cecutient-page");

        if($.cookie('cecutient-wiew') == "cecutient") {
            $.cookie('cecutient-wiew', null);
        }
        else {
            if($.cookie('page-color')) {
                pageColor($.cookie('page-color'), {expires: null, path: '/'});
            }
            else {
                $.cookie('page-color', 'white', {expires: null, path: '/'});
                pageColor("white");
            }
            console.log($.cookie('cecutient-wiew'));
            $.cookie('cecutient-wiew', "cecutient");
        }
    });

    if($.cookie('cecutient-wiew') == "cecutient") {
        
        if($.cookie('page-color')) {
            pageColor($.cookie('page-color'), {expires: null, path: '/'});
        }
        else {
            $.cookie('page-color', 'white', {expires: null, path: '/'});
            pageColor("white");
        }
        $("body").addClass("cecutient-page");
    }

-----------------
6. Остается править и переопределять стили версии для слабовидящих. Суть этой версии в том, что при переходе на нее у body добавляется класс cecutient-page. С его помощью и нужно переопределять стили обычной версии.