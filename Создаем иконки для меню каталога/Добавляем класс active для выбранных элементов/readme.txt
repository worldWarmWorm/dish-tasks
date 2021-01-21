1. В \protected\components\Controller.php добавить свойство

public $rootCategoryId = null; получает id каждого элемента меню

==============

2. В \protected\controllers\ShopController.php для главной страницы каталога в actionCategory добавить

if($ancestors=$category->ancestors()->findAll(['select' => 'id, lft, rgt, level, root'])) {
			$this->rootCategoryId=end($ancestors)->id;
		}
		else {
			$this->rootCategoryId=$category->id;
		}

Пример:

public function actionCategory($id)
    {
		$category = $this->loadModel('Category', $id);
		
		if($ancestors=$category->ancestors()->findAll(['select' => 'id, lft, rgt, level, root'])) {
			$this->rootCategoryId=end($ancestors)->id;
		}
		else {
			$this->rootCategoryId=$category->id;
		}
        
        $model = new Product('search');
        $model->unsetAttributes();  // clear any default values
        if(isset($_GET['Product']))
        	$model->attributes=$_GET['Product'];
        
        if(\Yii::app()->request->isAjaxRequest) {
        	$this->renderPartial('_products_listview', compact('model', 'category'), false, true);
        }
        else {
        	$brand=null;
        	if($brandId=Y::request()->getQuery('brand_id')) {
        		$brand=Brand::model()->actived()->previewColumns()->findByPk($brandId);
        	}

        	$this->prepareSeo($category->title);
	        $this->seoTags($category);
	        ContentDecorator::decorate($category, 'description');

	        $this->breadcrumbs->add($this->getHomeTitle(), '/shop');
	        $this->breadcrumbs->addByNestedSet($category, '/shop/category');
	        $this->breadcrumbs->add($category->title);
	        
	        $this->render('category', compact('model', 'category', 'brand') );
        }
    }

для страницы продукта в actionProduct добавить 

if($ancestors=$category->ancestors()->findAll(['select' => 'id, lft, rgt, level, root'])) {
			$this->rootCategoryId=end($ancestors)->id;
		}
		else {
			$this->rootCategoryId=$category->id;
		}

пример:

public function actionProduct($id)
    {
        $product = Product::model()->visibled()->findByPk($id);

        $this->prepareSeo($product->meta_title?:$product->title);
        $this->seoTags($product);

        if (!$product)
            throw new CHttpException(404, Yii::t('shop','product_not_found'));

        $this->breadcrumbs->add($this->getHomeTitle(), '/shop');

        $category=null;
        if($categoryId=\Yii::app()->request->getParam('category_id')) {
        	$category=Category::model()->findByPk($categoryId);
        }
        if(!$category) {
        	$category=$product->category;
		}
		
		if($ancestors=$category->ancestors()->findAll(['select' => 'id, lft, rgt, level, root'])) {
			$this->rootCategoryId=end($ancestors)->id;
		}
		else {
			$this->rootCategoryId=$category->id;
		}

        $this->breadcrumbs->addByNestedSet($category, '/shop/category');
        $this->breadcrumbs->add($category->title, array('/shop/category', 'id'=>$category->id));
        $this->breadcrumbs->add($product->title);
        
        $this->render('product', compact('product'));
    }

=================

3. В \protected\modules\menu\widgets\menu\BaseMenuWidget.php добавить 

$htmlOptions=$this->owner->rootCategoryId == $category->id ? ['class'=>'active'] : [];

Пример:

protected function getCategoryMenuHtml($categories, $level=0)
    {
		
        $maxLevel=(int)\D::cms('shop_menu_level', 1);
		if($level > $maxLevel) return '';
		
		$html='<ul class="catalog-menu">';
        foreach ($categories as $category) {
			$htmlOptions=$this->owner->rootCategoryId == $category->id ? ['class'=>'active'] : [];
		$html.=\CHtml::openTag('li', $htmlOptions);
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
