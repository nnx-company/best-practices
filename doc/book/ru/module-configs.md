# Конфигурирование модуля

## Подключение конфигов

Файл модуля должен *явно имплементировать интерфейс Zend\ModuleManager\Feature\ConfigProviderInterface*.

Пример подключения конфигурационного файла модуля:

```php

/**
 * @link  https://github.com/old-town/workflow-zf2-toolkit
 * @author  Malofeykin Andrey  <and-rey2@yandex.ru>
 */
namespace OldTown\Workflow\ZF2\Toolkit;


use Zend\ModuleManager\Feature\ConfigProviderInterface;


/**
 * Class Module
 *
 * @package OldTown\Workflow\ZF2\Toolkit
 */
class Module implements
    ConfigProviderInterface
{
    /**
     * @return array
     */
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }
}

```

Сам конфигурационный файл распологается согласно принятому стандарту структуры 
zf2 модуля в директории config(корень модуля) и имеет название module.config.php.

## Содержимое конфигурационного файла module.config.php

Конфигурационный файл модуля должен содержать:

- Секцию с настройкой модуля
- Подлючение других конфигурационных файлов

Пример:

```php

/**
 * @link     https://github.com/old-town/workflow-zf2-toolkit
 * @author  Malofeykin Andrey  <and-rey2@yandex.ru>
 */
namespace OldTown\Workflow\ZF2\Toolkit;

$config = [
    'workflow_zf2_toolkit' => [
        //Конфигурирование модуля
    ]
];
//Подключаем другие конфиги
return array_merge_recursive(
    include __DIR__ . '/doctrine.config.php',
    include __DIR__ . '/serviceManager.config.php',
    include __DIR__ . '/validator.config.php',
    include __DIR__ . '/workflowService.config.php',
    $config
);
```

## Что распологать в директории config

Все настройки которые не относятся к модулю, а имеено:

- Регистрация сервисов в ServiceLocator приложения
- Регистрация плагинов в разнообразных плагин менеджерах
- Настройка связанных модулей
- *dist* файл модуля(пример файла, который используем на уровне приложения)

Выносятся в *отдельные конфигурационные файлы*.

*Практика создания одного конфигурационного файла, содержащего в себе как
настройки модуля, так и настройки к модулю не относящиеся, считается устаревшей.*

## Семантика имен конфигурационных файлов

Имя файла должно соответствовать следующему шаблону

*чтоКонфигурируем(camelCase).config.php*

Например:

- doctrine.config.php - конфигурация доктрины
- serviceManager.config.php - конфигурация серви менеджера приложения
- validator.config.php - конфигурация плагин менененджера валидаторов
- workflowService.config.php - конфигурация модуля workflow-service

## Файл с примером конфигурации модуля

В каталоге config модуля обязательно распалагаем dist файл. Этот файл
содержит пример *публичных настроек модуля*. Т.е. таки настройки которые
предназначены для имзенения на уровне приложения, либо в других модуля.

При этом внутренний настройки модуля, которые не рекомендуется изменять, не должны
присутствовать в dist файле, но должны быть описаны в конфигурационном файле модуля.

Семантика имени dist файла, должна позволять однозначно идендфифицировать, 
к какому модулю он относится.

Пример имени dist файла:

- nnxContract.contractCore.config.php.dist

Т.е имя файла, состоит из следующих составляющих:

- имя вендора camelCase (в примере выше имя вендора nnx-contract)
- имя модуля camelCase (в примере выше имя модуля contract-core)
- суфикс config.php.dist

Предпологается что при установке модуля, необходимо скопировать dist файл
на уровень конфигов приложения, и это обеспечит корректную работу модуля.



## Подключение конфигурационных файлов не относящихся к модулю

Подключение конфигурационных файлов не относящихся к модулю осуществляется
с помощью следующей конструкии:

```php

/**
 * @link     https://github.com/old-town/workflow-zf2-toolkit
 * @author  Malofeykin Andrey  <and-rey2@yandex.ru>
 */
namespace OldTown\Workflow\ZF2\Toolkit;

$config = [
    'workflow_zf2_toolkit' => [
        //Конфигурирование модуля
    ]
];
//Подключаем другие конфиги
return array_merge_recursive(
    include __DIR__ . '/doctrine.config.php',
    include __DIR__ . '/serviceManager.config.php',
    include __DIR__ . '/validator.config.php',
    include __DIR__ . '/workflowService.config.php',
    $config
);
```
## Ограничения на содержание в конфигурационных файлах

В конфигурационных файла недопустимо использовать анонимные функции,
так как это приводит к невозможности кеширования конфигов.

Т.е. конструкции вида:

```

return [
    'service_manager' => [
        'invokables'         => [

        ],
        'factories'          => [
            EntryToObjectsService::class            => function() {
                retrun new EntryToObjectsService();
            }
        ],
        'abstract_factories' => [

        ]
    ],
];

```

Должны быть заменены на:

```

return [
    'service_manager' => [
        'invokables'         => [

        ],
        'factories'          => [
            EntryToObjectsService::class => EntryToObjectsServiceFactory::class,
        ],
        'abstract_factories' => [

        ]
    ],
];

```

## Использование provider интерфейсов

Зарегестрированные в приложение PluginManager (классы унаследованне от \Zend\ServiceManager\AbstractPluginManager),
позволяют конфигурировать себя через provider интерфейсы.

Для этого модуль должен имплементировать соответствующий provider интерфейс,
который возвращает конфиг соответствующего плагин менеджера([регистрация базовых плагин менеджеров](https://github.com/zendframework/zend-mvc/blob/release-2.4.9/src/Service/ModuleManagerFactory.php))

Настройка плагин менеджеров через провайдер конфиг является *не желательной*.
Лучше стараться что бы настройка модуля, осуществлялась через конфигурационные файлы.
Это делает поиск конфигов предсказуемыем. 

Использование provider интерфейсов возможно, когда необходимо наличие 
кода на php, для определения имен фабрик, или классов. Но таких случаев нужно избегать.
Так как потребность вносить логику в формирование конфига, свидетельствует о архитектурынх проблемах.

Поэтому при настройке PluginManager'ов через provider интерфейсы:
- *Необходимо согласовать это решение с другими teamlead*
- *Явно задокументировать логику реализуемую в соответствующих методах provider интерфейса*
