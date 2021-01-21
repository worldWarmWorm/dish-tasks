Пример на стандартной форме обратной связи callback

1. \protected\modules\feedback\configs\forms\callback.php указываем необходимые настройки

=====================
2. \protected\modules\feedback\views\admin\index.php

Добавить кнопку очистки временных файлов в отдном контейнере с h1 

<? $this->widget('\ext\uploader\widgets\ClearButton', [
	'clearUrl'=>'/cp/feedback/clearFiles?id=callback', 
	'htmlOptions'=>['class'=>'btn btn-warning pull-right']
]); ?>

Выводим все приходящие данные из формы

<?php foreach ($dataProvider->getData() as $feedback) : ?>
          <tr id="feedback-row-<?php echo $feedback->id; ?>" class="row<?php echo $feedback->id % 2 ? 0 : 1; ?>" valign="top">
            <td><?php echo $feedback->id; ?>.</td>
            <td class="title">
              <?php $model = $factory->getModelFactory()->getModel(); ?>
              <?php foreach ($factory->getModelFactory()->getAttributes() as $name => $typeFactory) : 
                if ((strpos($name, 'privacy_policy') !== false) || in_array($name, ['resume_hash'])) 
                  continue; 
              ?>
                <b><?php echo $model->getAttributeLabel($name); ?>:</b> <?php echo $typeFactory->getModel()->format($feedback->$name); ?><br />
              <?php endforeach; ?>
              <?php $this->widget('\ext\uploader\widgets\FileList', [
                'header' => 'Резюме:',
                'hash' => $feedback->resume_hash,
                'tagOptions' => ['class' => 'row', 'style' => 'margin:0;padding:0;'],
              ]); ?>
            </td>
            <td align="center"><?php echo str_replace(' ', '<br />', $feedback->created); ?></td>
            <td align="center">
              <div class="mark <?php echo !$feedback->completed ? 'marked' : 'unmarked'; ?>" data-item="<?php echo $feedback->id; ?>"></div>
            </td>
            <td><?php echo CHtml::link('Удалить', 'javascript:;', array('class' => 'feedback-btn-remove btn btn-danger', 'data-item' => $feedback->id)); ?></td>
          </tr>
        <?php endforeach; ?>

==========================
3. \protected\extensions\uploader\widgets\views\file_list.php 
   \protected\extensions\uploader\widgets\FileList.php - здесь делать ничего не надо. просто здесь верстка полученных файлов и их массив

=========================
4. \protected\modules\feedback\controllers\AjaxController.php - завеодим fction  и filter

public function actions()
	{
		return \CMap::mergeArray(parent::actions(), [
			'uploadFile' => [
				'class' => '\ext\uploader\actions\UploadFileAction',				
				'extensions' => 'doc,pdf,docx',
				'limit' => 1,
				'maxsize' => 10485760
			],
			'deleteFile' => '\ext\uploader\actions\DeleteFileAction',
		]);
	}

	public function filters()
	{
		return \CMap::mergeArray(parent::filters(), array(
			'ajaxOnly +uploadFile, deleteFile'
			//'ajaxOnly + send'
		));
	}

===========================
5. Вывод инпута загрузки файла в шаблоне формы \protected\modules\feedback\widgets\views\default.php

<?php $this->widget('\ext\uploader\widgets\UploadField', [
						'form' => $form,
						'model' => $factory->getModelFactory()->getModel(),
						'attribute' => 'resume_hash',
						'uploadUrl' => '/feedback/ajax/uploadFile',
						'deleteUrl' => '/feedback/ajax/deleteFile',
						'label' => 'Прикрепить файл резюме ',
						'view' => 'resume_upload_field'
					]); ?>

============================
6. Тут скрипт отправки файлов в папку \protected\extensions\uploader\widgets\views\resume_upload_field.php

============================
7. Удалить таблицу feedback-callback из базы, перезагрузить
