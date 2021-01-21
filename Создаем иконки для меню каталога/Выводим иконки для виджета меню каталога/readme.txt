1. Проверить в \protected\widget\ShopCategories\views\default.php что бы стояло 'encodeLabel' => false,


<?php $this->owner->widget('zii.widgets.CMenu', [
    'items'=>$items,
    'htmlOptions'=>['class'=>$this->listClass],
    'activateParents' => true,
    'encodeLabel' => false,
]); ?>

2. В \protected\widget\ShopCategories\ShopCategories.php в методе prepareTree поправить переменную $item

$item = array(
                'id' => $cat->id,
                'label'=> CHtml::tag('img', ['src' => '/images/category/' . $cat->menu_icon, 'class' => 'menu-icon'], '') . $cat->title . CHtml::tag('span', ['class' => 'deploy'], ''),
                'url'=>array('/shop/category', 'id'=>$cat->id),
                'linkOptions'=>array('title'=>$cat->getMetaATitle()),
            );


интересует ключ "label"

3. Добавить в значение ключа select "menu_icon"

$this->_categories = Category::model()->findAll(array(
        	    'select'=>'id, title, lft, menu_icon, rgt, root, level, ordering', 
            	'order'=>'ordering, lft'
	        ));