Пример реализованного функционала сайт abv.local!

Есть текстовые страницы и каталог, заголовки их совпадают. Принцип: выводим на странице Услуги->Услуга 1 товары со страницы Каталог->Категория 1!

1. В \protected\controllers\SiteController.php в actionPage дописать

	$products = [];

        $category = Category::model()->find('title=:title', [':title' => $page->title]);
              
        if ($category) {
            	$products = Product::model()->findAll('category_id=:category_id', 				[':category_id' => $category->id]);
        }

2. Выводим в шаблон \themes\adaptive_template_4\views\site\page.php

<?php if(!empty($products)) { ?>
	<div class="product-list row">
		<?php $i = 0; foreach ($products as $product) { ?>
			<?php $productUrl=Yii::app()->createUrl('shop/product', ['id'=>$product->id, 'category_id'=>$product->category_id]); ?>
			<div class="product-item">
				<div class="product">
					<div class="product__image product-block">
						<?= CHtml::link(CHtml::image(($product->getSrc())), $productUrl); ?>
					</div>
					<div class="product__title product-block">
						<?= CHtml::link($product->title, $productUrl, array('title' => $product->link_title)); ?>
					</div>
					<div class="product__descr">
						<?= CHtml::link('Подробнее', $productUrl, array('title' => $product->link_title)); ?>
					</div>
				</div>
			</div>
		<? $i++; if($i > 2) break; } ?>
	</div>
<? } ?>