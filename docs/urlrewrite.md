# Правила маршрутизации

Ниже рассмотренны возможности (создание, генерация) и примеры работ маршрутизации.

## Введение

Прежде всего необходимо определить следующее: **все** правила маршрутизации находятся в файле `urlrewrite.php`.
Благодаря этому мы получаем:

1. разделение обязанностей - все правила и всё что с ними связанно находятся в одном месте, а не размазаны по проекту и комплесным компонентам
2. удобство управления - вытекает из 1 пункта
3. возможность генерации - когда все правила находятся в одном месте, то можно однозначно сформировать URL по заданным параметрам

## Генерация URL из правил

В качестве идентификатора правила, используется ее `ID`.
Так как праметры в правилах не именованные, то соответствие будет определятся по количеству подставляемых параметров.

Пример: имеем 'urlrewerite.php' следующего содержания:
```php
$arUrlRewrite = array(
	array(
		"CONDITION"   =>   "#^/forum/#", 
		"ID"   =>   "bitrix:forum", 
		"PATH"   =>   "/forum/index.php", 
	),
	array(
		"CONDITION"   =>   "#^/forum/(\w+)/#", 
		"ID"   =>   "bitrix:forum", 
		"PATH"   =>   "/forum/section.php", 
	),
	array(
		"CONDITION"   =>   "#^/forum/(?:\w+)/(\w+)/#",
		"ID"   =>   "bitrix:forum",
		"PATH"   =>   "/forum/element.php", 
	),
);
```
В последнем шаблоне, первый параметр записан в формате `(?:\w+)`, но тем не менее при построении параметр все равно подставится.

В соотвествии с указанными правилами проводим генерацию URL:
```php
/*
 * url: /forum/
 */
Url::to("bitrix:forum");
/*
 * url: /forum/param1/
 */
Url::to("bitrix:forum", ["param1"]);
/*
 * url: /forum/param1/param2/
 */
Url::to("bitrix:forum", ["param1","param2"]);
/*
 * url: null
 */
Url::to("bitrix:forum", ["param1","param2","param3"]);
/*
 * будет выбрашено исключение 'InvalidUrlRewriteParametr'
 * валидность параметров проверяется формулой: !is_null($param) && trim($param) !== ""
 * поэтому 0 является валидным параметром, а пустые строки и объекты - нет
 */
Url::to("bitrix:forum", ["",null,0]);
```

Последний вызов ничего не вернул т.к. нет шаблона с использованием 3-ех параметров.

## Упрощенная генерация правил и именнованные параметры

Для поддержания условия нахождения всех правил в одном файле, да и просто для удобства, имеется класс для облегчения генерации правил.
Также данный класс позволяет использовать именованные параметры при генерации URL, т.к. все правила, изначально, проходят через этот класс и хранятся в нужно формате.

**РАСПИСАТЬ ВСЕ ПОДРОБНО С ОТДЕЛЬНЫМ ПРИМЕРОМ НА КАЖДУЮ ВОЗМОЖНОСТЬ И ЗАКЛЮЧИТЕЛЬНЫМ ПРИМЕРОМ С ВСЕМ СПИСОК ВОЗМОЖНОСТЕЙ**

```php
$arUrlRewrite = Url::generateRules([
    "bitrix:catalog" => [
        "path" => "/catalog/index.php",
        "condition" => [
            "^/catalog/",
            "^/catalog/{sectionCode}/",
            "^/catalog/{sectionCode}/{elementCode}/",
            [
                "^/catalog/(.*)",
                "r=$1",
            ]
        ],
    ],
]);
```