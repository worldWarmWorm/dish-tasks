1. В \protected\modules\feedback\configs\forms создать файл bid.php (название в зависимости от нужд)

<?php
/**
 * Обратный звонок
*
* 1 Имя
* 3 Контактный телефон
*/
return array(
	'bid' => array(
		'title' => 'Заявки',
		'short_title' => 'Заявки',
		// Options
		'options' => array(
			'useCaptcha' => false,
			'sendMail' => true,
			'emptyMessage' => 'Заявок нет',
		),
		// Form attributes
		'attributes' => array(
			'name' => array(
				'label' => 'ФИО',
				'type' => 'String', // String, Phone, Text, Checkbox, List
				'rules' => array(
					array('name', 'required')
				),
			),
			'phone' => array(
				'label' => 'Контактный телефон',
				'type' => 'Phone',
				'rules' => array(
					array('phone', 'required')
				),
			),
			'commentary' => array(
				'label' => 'Вам нужно? Расскажите подробнее, какие площади Вам нужны: вид бизнеса, диапазон метров .Обязательные условия…',
				'type' => 'Text'
			),
			'privacy_policy' => array(
				'label' => 'Нажимая на кнопку "Отправить", я даю согласие на ' . \CHtml::link('обработку персональных данных', ['/site/page', 'id'=>\D::cms('privacy_policy')], ['target'=>'_blank']),
				'type' => 'Checkbox',
				'rules' => array(
					array('privacy_policy', 'required')
				),
				'htmlOptions'=>['class'=>'inpt inpt-privacy_policy']
			),
		),
		// Control buttons
		'controls' => array(
			'send' => array(
				'title' => 'Заявка на аренду'
			),
		),
	),
);

============================

2. В protected\modules\feedback\widgets\views создать новый шаблон bid.php (название в зависимости от нужд)

<?php

/** @var FeedbackWidget $this */
/** @var FeedbackFactory $factory */

use common\components\helpers\HYii as Y;

Y::js(
	'feedback' . $this->getHash(),
	'var feedback' . $this->getHash() . '=FeedbackWidget.clone(FeedbackWidget);feedback' . $this->getHash() . '.init("' . $this->id . '");'
);

?>
<div id="<?= $this->id; ?>" class="<?= $this->getOption('html', 'class'); ?>">
	<?php
	$form = $this->beginWidget('CActiveForm', [
		'id' =>  $this->getFormId(),
		'action' => $this->getFormAction(),
		'enableClientValidation' => true,
		'enableAjaxValidation' => true,
		'clientOptions' => [
			'validateOnSubmit' => true,
			'validateOnChange' => false,
			'afterValidate' => 'js:feedback' . $this->getHash() . '.afterValidate',
		],
		'htmlOptions' => [
			'class' => 'bid-form'
		]
	]);
	$model = $factory->getModelFactory()->getModel();
	$fields = $factory->getModelFactory()->getAttributes();

	$labelName = $factory->getOption("attributes.name.label");
	$labelPhone = $factory->getOption("attributes.phone.label");
	$labelCommentary = $factory->getOption("attributes.commentary.label");
	$labelPrivacy = $factory->getOption("attributes.privacy_policy.label");
	?>

	<?= CHtml::hiddenField('formId', $this->getFormId()); ?>

	<?php
	if (is_callable($this->onBefore)) call_user_func($this->onBefore, $model);
	?>

	<?php if ($this->title) : ?>
		<div class="cbHead">
			<p><?= $this->title; ?></p>
		</div>
	<?php endif; ?>

	<div class="feedback-body fields">
		<label class="label label-name">
			<span><?= $labelName; ?></span>
			<?= $fields['name']->getModel()->widget($factory, $form, $this->params) ?>
		</label>

		<label class="label label-phone">
			<span><?= $labelPhone; ?></span>
			<?= $fields['phone']->getModel()->widget($factory, $form, $this->params) ?>
		</label>

		<label class="label label-commentary">
			<span><?= $labelCommentary; ?></span>
			<?= $fields['commentary']->getModel()->widget($factory, $form, $this->params) ?>
		</label>

		<div class="bottom">
			<div class="check">
				<?= $fields['privacy_policy']->getModel()->widget($factory, $form, $this->params) ?>
			</div>

			<?php if ($factory->getModelFactory()->getModel()->useCaptcha) {
				$this->widget('feedback.widgets.captcha.CaptchaWidget');
			} ?>

			<?= CHtml::submitButton($factory->getOption('controls.send.title', 'Отправить'), [
				'class' => 'feedback-submit-button btn',
				'id' => $factory->getModelFactory()->getModel()->recaptchaBehavior->attachReCaptchaBySubmitButton()
			]); ?>
		</div>
	</div>

	<div class="feedback-footer"></div>

	<?php $this->endWidget(); ?>
</div>

=================================

3. Вызов формы (если попап)

<div style="display: none;">
            <div id="form-bid">
                <div class="popup-info">
                    <?php $this->widget('\feedback\widgets\FeedbackWidget', [
                        'id' => 'bid',
                        'title' => 'Оформить заявку',
                        'view' => 'bid'
                    ]) ?>
                </div>
            </div>
        </div>

=================================

4. Стили поправить в \themes\adaptive_template_4\css\style.less


.bid-form {
    max-width: 870px;
}

.fancybox-content {
    border-radius: 4px;
}

[id^="bid"],
[id^="callback"] {
    position: relative;
    display: inline-block;

    form {
        padding: 5px;
        border-radius: 0;

        .cbHead {
            p {
                font-weight: 500;
                font-size: 24px;
                line-height: 34px;
                margin-bottom: 30px;
            }
        }

        input[type="text"],
        textarea {
            background: #fff;
            border: 1px solid #E0E0E0;
            border-radius: 4px;
            padding: 10px 15px;
            width: 100%;
            margin: 5px 0 15px;

            &:focus {
                border: 1px solid @c1;
                box-shadow: inset 0 0 0 1px @c1;
            }
        }

        input[type="submit"] {}

        textarea {
            height: 100px;
        }
    }

    .fields {
        display: flex;
        flex-wrap: wrap;

        .label {
            margin-bottom: 0;

            span {
                font-weight: 600;
                font-size: 13px;
                line-height: 16px;
                display: inline-block;
            }
        }

        .label-name {
            width: ~'calc(70% - 30px)';
            margin-right: 30px;
        }

        .label-phone {
            width: 30%;
        }

        .label-commentary,
        .bottom {
            width: 100%;
        }

        .bottom {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;

            .check {
                width: 50%;

                .inpt-privacy_policy+label {
                    font-family: 'Open Sans';
                    font-size: 13px;
                    line-height: 16px;

                    a {
                        color: @c3;
                    }
                }

                input[type="checkbox"] {
                    position: absolute;
                    width: 1px;
                    height: 1px;
                    overflow: hidden;
                    clip: rect(0 0 0 0);
                }

                input[type="checkbox"]:checked+label::before {
                    border-bottom: 2px solid #fff;
                    border-left: 2px solid #fff;
                }

                label {
                    padding-left: 30px;
                    position: relative;

                    &::after {
                        content: '';
                        display: block;
                        position: absolute;
                        left: 0;
                        top: 2px;
                        background-color: @c3;
                        border-radius: 2px;
                        width: 20px;
                        height: 20px;
                        transition: .3s all;
                        cursor: pointer;
                    }

                    &::before {
                        content: '';
                        display: block;
                        position: absolute;
                        left: 5px;
                        top: 7px;
                        width: 10px;
                        height: 6px;
                        border-bottom: 2px solid transparent;
                        border-left: 2px solid transparent;
                        transform: rotate(-45deg);
                        transition: .3s all;
                        z-index: 1;
                        cursor: pointer;
                    }
                }
            }
        }
    }
}
