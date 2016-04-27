# Принципы построения доменной модели

Одна из основных причин разделения сервиса на два слоя:

- Абстрактая часть сервиса;
- Конфигурация сервиса под конкретную информационную систему;

это дать возможность дорабатывать доменную модель под конкретного заказчика.

Рассмотрим принципы построения доменной модели сервиса на примере.

Есть две информационные системы для разных заказчиков *customer1* и *customer2*. Цель - разработать сервис контактов для
эти двух информационных систем. Одним из требований является то, что информация об организациях в этих информационных системах
имела бы разную структуру.

Для каждого из заказчиков создается два приложения, каждое из которых является контейнером для сервиса контактов.
Упрощенная структура модулей:

- Приложение-контейнер для сервиса контактов заказчика *customer1*:

```text
project
    vendor
        nnx-contract
            contract
            contract-core
        customer1-contract
            organization
```

- Приложение-контейнер для сервиса контактов заказчика *customer2*:

```text
project
    vendor
        nnx-contract
            contract
            contract-core
        customer2-contract
            organization
```

Т.е реализация сущностей, подстраивающих доменную модель под конкретного заказчика, выносится в соответствующие модули
*customer1-contract\organization* и *customer2-contract\organization*.

## Доменная модель в Core модулях.

У каждого сервиса есть Core-модуль, в котором сосредоточено описание базовых сущностей сервиса, а также предоставляются
сервисы для работы с наиболее общим функционалом.

Спецификой Core-модулей, является то, что в сервисах, расположенных в данных модулях, неизвестно, с каким именно классом
сущности будет происходить работа.

В случае модуля *nnx-contract/contract-core* заранее неизвестно имя класса организации. Так как сущность организации, которая
будет использоваться в сервисе, может быть реализована в другом модуле (в приведенном примере это *customer1-contract\organization* и *customer2-contract\organization*).

Для достижения такой гибкости используется следующий подход:

- Для каждой сущности, которая может расширяться в других модулях, заводится интерфейс;
- В рамках сервиса модуля работа осуществляется с объектами, имлементирующими заданный интерфейс, т.е. **не допускается использовать имена классов**;
- Если есть часть свойств, которые гарантированно будут у всех классов, имплементирующих данный интерфейс, то создается абстрактный класс;
- Создается класс сущности, являющейся родительской, для сущностей, располагаемых в других модулях.

Расмотрим конкретный пример:

Создание сущности контракта:


```php

namespace Nnx\Contract\Core\Entity;

//Интерфейс сущности контракта. В сервисах работает с интерфейсом, а не с реализацией.
interface ContractInterface
{
    public function getId();

    public function getRegisterNumber();
}
```

```php

namespace Nnx\Contract\Core\Entity;

use Doctrine\ORM\Mapping as ORM;

//Выносим в абстрактную часть, свойства, которые гарантированно есть у всех классов контрактов

/**
 * Class AbstractContract
 *
 * @ORM\MappedSuperclass()
 */
abstract class AbstractContract implements ContractInterface
{
    /**
     * @var int
     *
     * @ORM\Id()
     * @ORM\Column(name="id", type="integer")
     * @ORM\GeneratedValue(strategy="IDENTITY")
     */
    protected $id;

    /**
     * Реестровый номер
     *
     * @var string
     *
     * @ORM\Column(name="register_number", type="string", nullable=true)
     */
    protected $registerNumber;

    /**
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getRegisterNumber()
    {
        return $this->registerNumber;
    }
}
```

```php

namespace Nnx\Contract\Core\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * Непосредственная реализация сущности контракта
 *
 * @ORM\Entity
 * @ORM\Table(name="contract")
 */
class Contract extends AbstractContract
{
    /**
     * Заказчик
     * 
     * @var  Customer
     *
     * @ORM\ManyToOne(targetEntity="DefaultCustomerInfo",  fetch="LAZY",
     *  cascade={"persist", "remove"})
     * @ORM\JoinColumn(name="customer_id", referencedColumnName="id")
     */
    protected $customer;

    /**
     * Поставщики
     * 
     * @var Supplier
     *
     * @ORM\ManyToMany(targetEntity="Supplier", cascade={"persist", "remove"})
     * @ORM\JoinTable(name="contract_supplier",
     *      joinColumns={@ORM\JoinColumn(name="contract_id", referencedColumnName="id")},
     *      inverseJoinColumns={@ORM\JoinColumn(name="supplier_id", referencedColumnName="id")}
     *      )
     */
    protected $suppliers;

    /**
     * @return Customer
     */
    public function getCustomer()
    {
        return $this->customer;
    }

    /**
     * @return ArrayCollection|Supplier[]
     */
    public function getSuppliers()
    {
        return $this->suppliers;
    }
}
```

Создаем сущности заказчика и поставщика:

```php

namespace Nnx\Contract\Core\Entity;

/**
 * Interface OrganizationInterface
 *
 */
interface OrganizationInterface
{
    /**
     * @return int
     */
    public function getId();

    /**
     * @return string
     */
    public function getInn();
}
```

Абстрактная реализация организации:

```php

namespace Nnx\Contract\Core\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 * @ORM\Table(name="contract_organization")
 * @ORM\InheritanceType(value="SINGLE_TABLE")
 * @ORM\DiscriminatorColumn(name="type", type="string")
 */
abstract class AbstractOrganization implements OrganizationInterface
{
    /**
     * @var int
     *
     * @ORM\Id()
     * @ORM\Column(name="id", type="integer")
     * @ORM\GeneratedValue(strategy="IDENTITY")
     */
    protected $id;

    /**
     * ИНН
     * @var string
     *
     * @ORM\Column(name="inn", type="string", nullable=true)
     */
    protected $inn;

    /**
     * @return int
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function getInn()
    {
        return $this->inn;
    }
}
```

Заказчик:

```php

namespace Nnx\Contract\Core\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class Customer extends AbstractOrganization
{

}

```

Поставщик:

```php

namespace Nnx\Contract\Core\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class Supplier extends AbstractOrganization
{

}

```

## Расширение доменной модели под конкретного заказчика

В предыдущем пункте был приведен пример, описывающий сущности Contract, а также сущности Customer и Supplier. Поскольку структура сущностей Customer и Supplier может меняться от одной информационной системы к другой, то описание этих
сущностей выносится в ту часть сервиса, которая расширяет абстрактную часть.

Пусть у нас есь сервис контрактов со следующей структурой каталогов:


```text
project
    vendor
        nnx-contract
            contract
            contract-core
        customer1-contract
            organization
```

В *nnx-contract/contract-core* описаны сущность Contract, а также Customer и Supplier.

Расмотрим на примере расширение этих сущностей в модуле:

*customer1-contract/organization*


Заказчик:

```php

namespace Customer1\Contract\Core\Entity;

use Doctrine\ORM\Mapping as ORM;
use Nnx\Contract\Core\Entity\Customer;

/**
 * @ORM\Entity()
 */
class ExtendedCustomer extends Customer
{
    /**
     * @var string
     *
     * @ORM\Column(name="custom_field1", type="string", nullable=true)
     */
    protected $customField1;
}

```

Поставщик:

```php

namespace Customer1\Contract\Core\Entity;


use Doctrine\ORM\Mapping as ORM;
use Nnx\Contract\Core\Entity\Supplier;

/**
 * @ORM\Entity()
 */
class ExtendedSupplier extends Supplier
{
    /**
     * @var string
     *
     * @ORM\Column(name="custom_field2", type="string", nullable=true)
     */
    protected $customField2;
}

```


