Библиотека pickle
#################

:date: 2019-11-25 09:00
:summary: Тема ?. Сериализация и десериализация объектов.
:status: draft

.. default-role:: code

.. role:: python(code)
   :language: python

.. contents::

Сериализация и десериализация
-----------------------------

Под **сериализацией** понимается процесс перевода какой-либо структуры данных в последовательность битов.
**Десериализация** является обратной опреацией и восстанавливает структуры данных из последовательности битов.

Сериализация имеет ряд применений. Один из них - передача объектов по сети или по другим каналам обмена данными.
Например, у вас есть распеделенное приложение, разные части котрого обмениваются данными со сложной структурой.
Для типов данных, объекты которых надо передавать, пишется код, выполняющий сериализацию и десериализацию.
Перед отправкой объекта вызывается код сериализации, в результате получается документ, например, в формате XML или JSON.
Полученный документ отправляется приложению-получателю. Получатель вызывает код десериализации и восстанавливает объект того же типа с теми же данными, что были до сериализации.

Другой способ применения - дамп объектов в файл с целью сохранения их состояния.
Такой подход полезен в целях консервирования важных объектов для их последующего восстановления после перезапуска приложения.
Например, вы решаете задачу моделирования физической системы. Ваше приложение позволяет вам взаимодействовать с сиетмой в режиме реального времени.
Прошло какое-то количество времени, модель уже отличается от ее изначального состояния.
Пусть текущее состояние модели является для вас каким-то важным, и вам хотелось бы его сохранить.
Используя сериализацию, вы просто записываете состояние вашей модели в файл.
На случай падения вашей программы или неправильного воздействия на вашу систему, вы можете восстановить сохраненое состояние и продолжить работу.

Кстати, знакомая многим возможность сохранения/загрузки в игре - ни что иное, как пример использования сериализации/десериализации.


Текстовая сериализация
======================

С лабы №3 вы уже познакомились с возможностью сериализации объектов в текстовый формат.
Один из форматов - CSV, предназначенный для записи табличных данных в файл.
Также есть форматы XML, YAML, JSON, позволяющие сохранять произвольные структуры. Основной особенностью сериализации в текст является человекочитаемость.

Библиотека pickle
-----------------

`Оффициальная документация`_

.. _`Оффициальная документация`: https://docs.python.org/3.6/library/pickle.html

pickle - одна из стандартных библиотек Python, позволяет выполнять сериализацию и десериализацию, работает с потоком байт.
Формат данных, используемый библиотекой, заточен по Python, и может быть несовместим с непитоновскими программами.
Однако формат данных является довольно компактным, особенно в сравнении с тектовыми форматами, и может быть дополнительно сжат при необходимости.

pickle поддерживает ряд протоколов сериализации.
На момент выхода Python 3.8 их стало 6.
Чем выше версия протокола, тем выше требования к версии Python.
В данной работе будет использован протокол версии 3 (является протоколом по-умолчанию для Python 3.0-3.7 и является несовместимым с Python 2.x).
Протокол v3 работает с объектами типа bytes.

bytes
=====

Синтаксис bytes аналогичен синтаксису str, за исключением префикса `b`.
Например, `b'hello'`. bytes является неизменяемой последовательностью интов, где каждый элемент удовлетворяет неравенству `0 <= x < 256`.
В то время, как значения из таблицы ASCII имеют обычную запись, остальные значения используют запись в шестнадцатеричной системе счисления с предшествующими символами `\x`.
Например, значение `128` будет записано, как `b'\x80'`.

Поддерживаемые типы
===================

pickle поддерживает следующие типы:

+ None, True и False
+ числовые типы
+ строковые и байтовые типы
+ кортежи, списки, множества и словари, состоящие только из сериализуемых объектов
+ функции (в том числе встроенные) верхнего уровня, объявленные с def (верхнего уровня = с нулевым отступом)
+ классы верхнего уровня
+ объекты классов, для которых `__dict__` или результат `__getstate()__` является сериализуемым.

Обратите внимание, что сериализация функций выполняется по имени, а не по значению.
Т.е. библиотека сохраняет **только имя функции с именем модуля, где она определена**.
Тоже самое справедливо и для классов.

Сериализация встроенных типов
=============================

Библиотека предоставляет следующий набор функции:

+ `dump(obj, file, protocol=None, *, fix_imports=True)`
+ `dumps(obj, protocol=None, *, fix_imports=True)`
+ `load(file, *, fix_imports=True, encoding="ASCII", errors="strict")`
+ `loads(bytes_object, *, fix_imports=True, encoding="ASCII", errors="strict")`

`dump` и `load` работают с файлом, в то время как `dumps` и `loads` работают с bytes.
Если аргумент `protocol` не задан, то используется версия протокола по-умолчанию.
`fix_imports`, `encoding` и `errors` нужны для совместимости Python 2 и нами использоваться не будут.

Рассмотрим простой пример.

.. code:: python

    import pickle

    # An arbitrary collection of objects supported by pickle.
    data = {
        'a': [1, 2.0, 3, 4+6j, float("nan")],
        'b': ("character string", b"byte string"),
        'c': {None, True, False}
    }

    # Pickle the 'data' dictionary using
    # the default protocol and print the result.
    print(pickle.dumps(data))

    with open('data.pickle', 'wb') as f:
        # Pickle the 'data' dictionary to file
        # using the highest protocol available.
        pickle.dump(data, f, pickle.HIGHEST_PROTOCOL)

Теперь в отдельной программе выполним десериализацию.

.. code:: python

    import pickle

    with open('data.pickle', 'rb') as f:
        # The protocol version used is detected automatically,
        # so we do not have to specify it.
        data = pickle.load(f)
    print(data)

Обратите внимание, что файлы на чтение и запись надо открывать в двоичном режиме.

Сериализация объектов классов
=============================

TODO: instances serialization, add exercises