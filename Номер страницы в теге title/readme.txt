1. На главной \protected\controllers\ShopController.php в аctionIndex 

$this->seoTags(array(
			'meta_h1' => $settings->meta_h1 ?: $this->getHomeTitle(),
			'meta_title' => (isset($_REQUEST['p']) ? " {$this->getHomeTitle()} - страница {$_REQUEST['p']}" : "{$this->getHomeTitle()} - страница 1"),
			// 'meta_title' => $settings->meta_title ?: $this->getHomeTitle(),
			'meta_key' => $settings->meta_key,
			'meta_desc' => $settings->meta_desc
		));

2. На внутренней (подкатегории) \protected\controllers\ShopController.php в аctionIndex 

$this->prepareSeo($category->title . (isset($_REQUEST['p']) ? " - страница {$_REQUEST['p']}" : ' - страница 1'));
