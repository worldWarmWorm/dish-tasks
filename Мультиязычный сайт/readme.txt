1. Создать копию базы

sitename - база
sitenaneen - копия (в конце "en" - english)

---------------------------------
2. Создать два файла настроек базы в /protected/config

ru.db.php
en.dp.php

---------------------------------
3. В index.php на самом верху добавить

$domainParts = explode('.', $_SERVER['HTTP_HOST']);

$language = $domainParts[0] == '<название сайта без домена второго уровня>' ? 'ru' : $domainParts[0];
define('LANG_ID', $language);

---------------------------------
4. В \protected\config\defaults.php переопределить язык

'language'=>LANG_ID

---------------------------------
5. В \protected\models\Product.php добавить в поведение mainImageBehavior еще один ключ

'baseFilePath' => 'images/' . LANG_ID,

---------------------------------
6. В \protected\config\main.php преопределить префикс

$prefix = LANG_ID . '.';

---------------------------------
7. Меню может кэшироваться, поэтому по надобности отключить кэш можно в \protected\modules\menu\widgets\menu\MenuWidget.php в футкции run()

public function run()
	{
		// $cacheId="widgetMenuTree__{$this->id}_{$this->rootId}";
		// $tree=Y::cache()->get($cacheId);
		$tree = null;

	    	if(empty($tree)) {
		    
		....
	}

--------------------------------
8. Проверить в папках /message в публичке и в админке наличие папки /en - там перевод админки и публички. Иногда для админки перевод не нужен, в этом случае папка /en все равно нужно, только ее содержанием должно быть папка /ru, чтобы выводились хотя бы русские слова а не куски программы.

--------------------------------
9. После наполнения сайта английским контентом, нужно будет что-то переводить руками, погляди по сайту.
