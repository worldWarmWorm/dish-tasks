1. Активируем выпадающий список - настройки\магазин\Активировать выпадающий список категорий в меню

2. В модели Category добавляем поведение iconBehavior

		...

		'iconBehavior'=>[
        		'class'=>'\common\ext\file\behaviors\FileBehavior',
        		'attribute'=>'menu_icon',
        		'attributeLabel'=>'Иконка для меню',        		     		
        		'imageMode'=>true
        	],

		...


3. Создать в бд таблицу с названием 'menu_icon' по образцу 'main_image'


4. Вывод элементов меню каталога динамический, располагается здесь \protected\modules\menu\widgets\menu\BaseMenuWidget.php

protected function getCategoryMenuHtml($categories, $level=0)
    {
		
        $maxLevel=(int)\D::cms('shop_menu_level', 1);
        if($level > $maxLevel) return '';
        
        $html='<ul class="catalog-menu">';
        foreach ($categories as $category) {
            $html.=\CHtml::openTag('li');
            $html.=\CHtml::link($category->iconBehavior->img(40,40) . \CHtml::openTag('span') .$category->title . \CHtml::closeTag('span'), ['/shop/category', 'id'=>$category->id]);
            if(($level + 1) < $maxLevel) {
                if($subcategories=$category->children()->findAll(['select'=>'id, title, lft, rgt, level, root', 'order'=>'lft'])) {
                    $html.=$this->getCategoryMenuHtml($subcategories, $level+1);
                }
            }
            $html.=\CHtml::closeTag('li');
        }
        $html.='</ul>';
        
        return $html;
    }

5. Добавить в выборку атрибут 'menu_icon' на строке 134

if(\D::cms('shop_menu_enable')) {
                if ($url == $shopIndexUrl) {
                    if($roots=\Category::model()->roots()->findAll(['select'=>'id, title, menu_icon, lft, rgt, level, root', 'order'=>'ordering'])) {
                        $html.=$this->getCategoryMenuHtml($roots);
                    }
                }
            }

6. Для правильного удаления иконки в \protected\modules\admin\controllers\ShopController.php добавить в

public function actions() {

....

'removeCategoryIconImage'=>[
	'class'=>'\common\ext\file\actions\RemoveFileAction',
	'modelName'=>'\Category',
	'behaviorName'=>'iconBehavior',
	'ajaxMode'=>true
],
....

}

7. Вызвать форму в админке \protected\modules\admin\views\shop\_form_category.php

	<? $this->widget('\common\ext\file\widgets\UploadFile', [
		'behavior'=>$model->iconBehavior, 
		'form'=>$form, 
		'actionDelete'=>$this->createAction('removeCategoryIconImage'),
	    'tmbWidth'=>40,
	    'tmbHeight'=>40,
	    'view'=>'panel_upload_image'
	]); ?>

