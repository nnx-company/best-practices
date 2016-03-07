# Сервис-оринтированная архитектура

# Термины и определения

## Информационная система

Информационная система(ИС) - готовый продукт поставляемый заказчику. Информационной системой можно считать совокупность 
программных и аппаратных решений. Информационная система представляет из себя набор сервисов,  а также их конфигурацию.
Информационная система может характеризоваться следующими пунктами: 

* Набор сервисов
* Конфигурация сервисов
* Прослойка(шина) для связи сервисов

Информационная система имеет свою версию.

## Сервис

Сервис является набор компонент, реализующих законченный функционал конкретной бизнес логики. Напрмерм информационная
система может включать в себя следующие сервисы:

* Сервис работы с пользователями и организациями
* Сервис проведения торгов
* Сервис заключения контрактов
* Сервис работы с ООС

Сервис имеет свою версию, которая может отличаться от версиии информационной системы.
Чаще всего сервис может характеризоваться следующими пунтками:

* Сервис либо является веб-службой, либо не должен иметь препятствий, для того что бы быть использованным на отдельном сервере
* Отдельная база данных
* Сервис предоставляет API для взаимодействия с ним.
* Сервис предоставляет из себя набор модулей zf2 + конфигурация

## Модули

Модуль - это модуль написанный по стандартам ZendFramework2


