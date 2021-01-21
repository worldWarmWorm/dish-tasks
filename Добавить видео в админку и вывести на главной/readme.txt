1. Строка 19 public static $files = array('index_video');
=================
2. Строка 25 public static $filesNames = array('index_video'=>'index_video');
=================
3. Добавить index_video в миссив переменной $rules

array('events_title, events_link_all_text, copyright_city, favicon, privacy_policy, privacy_policy_text, index_video', 'safe'),
=================
4. Добавить в attributeLabels
'index_video' => 'Видео на главной',
=================
5. Вывод в публичку
<?php if(D::cms('index_video')) { ?>
                <video controls>
                    <source src="/files/settings/<?= D::cms('index_video')?>" type="video/mp4">
                </video>
            <? } ?>
