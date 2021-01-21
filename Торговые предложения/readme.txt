1. добавить в \protected\modules\admin\views\shop\form_product_general.php в переменную $tabs:
'Торг. предложения' => array('content' => $this->renderPartial('_form_product_offers', compact('model', 'form'), true), 'id' => 'tab-offers'),
----------------------------------
2. В \protected\modules\admin\views\shop\ добавить файл _form_product_offers.php - атрибуты торгового предложения

<div class="row">
	<?= $form->labelEx($model, 'data'); ?>
	<? $this->widget('\common\ext\dataAttribute\widgets\DataAttribute', [
		'behavior' => $model->dataAttributeBehavior,
		'header'=>['title'=>'Размер', 'price'=>'Цена'],
		'default' =>[
			['title'=>'', 'price'=>''],
		]
	]); ?>
</div>
-----------------------------------
3. во вью \protected\views\shop\_products.php делаем вывод данных и передачу их в корзину

3.1 получаем все активные торговые предложения

$offers = $data->dataAttributeBehavior->get(true);


3.2 переопределяем стандартную цену 

if ($offers) $data->price = $offers[1]['price'];


3.3 добавляем верстку переключателей цены и размера порции 

<?php if ($offers) : ?>
	<div class="product-offers" id="product<?= $data->id ?>-offers">
	<?php foreach (array_reverse($offers) as $key => $offer) : ?>
		<div class="product-offer">
			<input id="product<?= $data->id ?>-offer-<?= $key ?>" class="js-			offer" data-price="<?= $offer['price'] ?>" type="radio" 			name="offer-<?= $data->id ?>" value="<?= $offer['title'] ?>" <?= 			$key ? '' : 'checked' ?>>
			<label for="product<?= $data->id ?>-offer-<?= $key ?>"><?= $offer			['title'] ?></label>
		</div>
	<?php endforeach ?>
	</div>
<?php endif; ?>

3.4 для кнопки добавления в корзину в атрибуты карты товара добавить еще один атрибут

<?$this->widget('\DCart\widgets\AddToCartButtonWidget', array(
	'id' => $data->id,
	'model' => $data,
	'title'=>'<span>Хочу</span>',
	'cssClass'=>'shop-button to-cart button_1 js__in-cart open-cart',
	'attributes'=>[
		['count', '#js-product-count-' . $data->id],
		['offer', "#product{$data->id}-offers .js-offer:checked"]
	]
	));
?>
-------------------------------
4. в модели Product

4.1 добвить public $offer;

4.2 добавить поведение 

'dataAttributeBehavior' => [
	'class' => '\common\ext\dataAttribute\behaviors\DataAttributeBehavior',
        'attribute' => 'data',
        'attributeLabel' => 'Торговые предложения'
],

4.3 в scopes в select добавить t.data

'cardColumns'=>[
        		'select'=>'`t`.`id`, `t`.`category_id`, title, code, price, sale, new, hit, link_title, notexist, old_price, `main_image`, `main_image_alt`, `main_image_enable`, composition, weight, t.data'
        	],
4.4 добавить в конце функцию
/**
     * Event: onAfterAdd
     * Обработчик события после добавления товара в корзину.
     * @param array &$item элемент позиции в массиве конфигурации корзины.
     * @param string $attribute имя атрибута.
     * @param mixed $value значение атрибута.
     */
    public function afterAddCart(&$item, $attribute, $value)
    {
        if (!empty($item['attributes']['offer'])) {
            $offers = Product::model()->findByPk($item['id'])->dataAttributeBehavior->get		(true);

            foreach ($offers as $offer) {
                if ($item['attributes']['offer'] == $offer['title']) {
                    $item['price'] = $offer['price'];
                }
            }
        }
    }
----------------------------------
5 в \protected\modules\admin\views\shop\ добавить файл _form_product_offers.php - это отображение в админке


<div class="row">
	<?= $form->labelEx($model, 'data'); ?>
	<? $this->widget('\common\ext\dataAttribute\widgets\DataAttribute', [
		'behavior' => $model->dataAttributeBehavior,
		'header'=>['title'=>'Размер', 'price'=>'Цена'],
		'default' =>[
			['title'=>'', 'price'=>''],
		]
	]); ?>
</div>
------------------------------------
6. в \protected\config\dcart.php сделать по аналогии с offer

/**
 * Конфигурационный файл для компонента \DCart\components\DCart
 */
return array(
	'class' => '\DCart\components\DCart',
	'attributeImage' => 'cartImg',
	'extendKeys' => ['offer'],
	'cartAttributes' => [ // аттрибуты которые будут отображены дополнительно в виджете корзины
		'weight',
		'composition',
		'offer'
	],
	'attributes' => [ // аттрибуты, которые будут сохранены для заказа
		'code', // => ['onAfterAdd'=>'afterAddCart', 'onAfterUpdate'=>'afterAddCart']
		'weight' => ['label' => false],
		'composition' => ['label' => false],
		'offer' => [
			'onAfterAdd' => 'afterAddCart',
			'onAfterUpdate' => 'afterAddCart'
		],
	]
);
-------------------------------------------
7. в \js\main.js добавить скрипт подмены цены в зависимости от порции

$.fn.digits = function(){
		return this.each(function(){
			$(this).text( $(this).text().replace(/(\d)(?=(\d\d\d)+(?!\d))/g, "$1 ") );
		});
	};

	$('.js-offer').on('change', function() {
		$(this).closest('.product-item').find('.js-price').text($(this).data('price')).digits();
		console.log($(this).closest('.product-item').find('.js-price').text($(this).data('price')).digits());
	});
-----------------------------------------------
8. доработать верстку


=============================

САЙТ ПРИМЕР http://bento.local/

вьюшка _products.php

<?
/**
 * @var Product $data
 */

if (empty($class)) {
	$class = 'col-sm-6 col-md-6 col-lg-6 col-xl-4';
}

$cache=\Yii::app()->user->isGuest;
if(!$cache || $this->beginCache('shop__product_card', ['varyByParam'=>[$data->id]])): // cache begin

if(empty($category)) $categoryId=$data->category_id;
else $categoryId=$category->id;
$productUrl=Yii::app()->createUrl('shop/product', ['id'=>$data->id, 'category_id'=>$categoryId]);

$offers = $data->dataAttributeBehavior->get(true);
if ($offers) $data->price = $offers[1]['price'];

?>

<div class="<?= $class ?>">
	<div class="product-item-wrapper <?= HtmlHelper::getMarkCssClass($data, array('sale', 'new', 'hit')) ?>">
		<div class="product-item">
			<div class="product-item__image">
				<?= CHtml::link(CHtml::image(ResizeHelper::resize($data->getSrc(), 225, false)), $productUrl); ?>
			</div>
			<div class="product-item__title">
				<?= CHtml::link($data->title, $productUrl, array('title' => $data->link_title)); ?>
			</div>
			<div class="product-item__weight">
				<span><?= $data->weight ?></span>
			</div>
			<div class="product-item__composition">
				<span><?= $data->composition ?></span>
			</div>
			<?php if ($offers) : ?>
				<div class="product-offers" id="product<?= $data->id ?>-offers">
					<?php foreach (array_reverse($offers) as $key => $offer) : ?>
						<div class="product-offer">
							<input id="product<?= $data->id ?>-offer-<?= $key ?>" class="js-offer" data-price="<?= $offer['price'] ?>" type="radio" name="offer-<?= $data->id ?>" value="<?= $offer['title'] ?>" <?= $key ? '' : 'checked' ?>>
							<label for="product<?= $data->id ?>-offer-<?= $key ?>"><?= $offer['title'] ?></label>
						</div>
					<?php endforeach ?>
				</div>
			<?php endif; ?>
			<div class="product-item__bottom">
				<div class="product-item__price-wrapper">
					<div class="product-item__count">
						<a href="javascript:;" class="js-price-down">-</a>
						<input type="text" id="js-product-count-<?= $data->id ?>" value="1" placeholder="" readonly>
						<a href="javascript:;" class="js-price-up">+</a>
					</div>
					<div class="product-item__price-block">
						<div class="product-item__price">
							<span class="new_price"><span class="js-price"><?= HtmlHelper::priceFormat($data->price); ?></span>
								<span class="rub"> руб.</span>
							</span>
						</div>
					</div>
				</div>
				<div class="product-item__buy">
					<?if($data->notexist):?>
					Нет в наличии
					<?else:?>
					<?$this->widget('\DCart\widgets\AddToCartButtonWidget', array(
							'id' => $data->id,
							'model' => $data,
							'title'=>'<span>Хочу</span>',
							'cssClass'=>'shop-button to-cart button_1 js__in-cart open-cart',
							'attributes'=>[
								['count', '#js-product-count-' . $data->id],
								['offer', "#product{$data->id}-offers .js-offer:checked"]
							]
						));
						?>
					<?endif?>
				</div>
			</div>
		</div>
	</div>
</div>

<? if($cache) { $this->endCache(); } endif; // cache end ?>