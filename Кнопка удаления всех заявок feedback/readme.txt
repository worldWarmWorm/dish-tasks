1. В \protected\modules\feedback\controllers\AdminController.php создать функцию

protected function actionDeleteAll($feedbackId)
	{
		$factory = FeedbackFactory::factory($feedbackId);
		$model = $factory->getModelFactory()->getModel();
		
		$model->deleteAll();

		\Yii::app()->end();
	}

2. В actionIndex добавить case

===============================

case 'deleteAll':
	$this->actionDeleteAll($feedbackId);
	\Yii::app()->end();
	break;

===============================

3. Добавление кнопки в \protected\modules\feedback\views\admin\index.php

<div>
      <div style="text-align: end; margin-bottom: 30px;"><?php echo CHtml::ajaxButton('Удалить все заявки', '/cp/feedback/'. $factory->getId() . '/deleteAll', [
        'type'=>'post',
        'beforeSend'=>'js:function(){return confirm("Записи не подлежат восстановлению. Удалить все записи? ")}',
        'success'=>'js:function(){window.location.reload()}'
      ], ['class' => 'btn btn-danger']); ?></div>
    </div>