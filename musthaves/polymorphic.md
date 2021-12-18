# Полиморфные классы

Класс называется полиморфным (в терминах с++), если у него есть хотя бы одна виртуальная функция.

**Виртуальные методы и их отличия от невиртуальных**

* Виртуальные методы создаются с помощью ключевого слова `virtual` и могут перезаписываться
  наследниками с помощью слова `override` (`virtual` наследнику добавлять необязательно). Всегда
  вызывается виртуальная функция, принадлежащая тому классу, от которого мы вызвались.
* На самом деле, даже `override` писать не обязательно, т.к. если функция по сигнатуре в точности
  совпадает с виртуальной функцией сверху, то она становится виртуальной. `override` лишь проверяет,
  точно ли есть у родителя такой виртуальный метод.

**Виртуальный деструктор**

* Пример:

```c++
Base *b = new Derived; // Derived has its own destructor
delete b; // but ~Base != ~Derived
```

* Здесь мы пытаемся вызвать деструктор от `Base`, но он уберет за собой только то, что относится
  к `Base`, оставив поля `Derived` не очищенными. Поэтому нужно объявить виртуальный деструктор,
  чтобы компилятор понял, кого именно нужно удалить.
* `virtual ~Base() = default` - теперь, когда понадобиться удалить наследник `Base`, вызовется
  правильный деструктор от соответствующего наследника.
* У наследников все генерируемые деструкторы автоматически становятся виртуальными и перезаписывают
  базовый.

**Чисто виртуальные функции**

* Используются для абстрактных классов: например, есть какие-то виджеты `button`, `ball_icon`, `box`
  , у которых есть ширина и высота, но для всех они вычисляются по-разному. Хотим, чтобы их можно
  было единообразно обрабатывать, тогда делаем общий базовый класс `Widget`, от которого они
  наследуются.
* `virtual type func() const = 0` - чисто виртуальная функция, которую обязательно нужно
  переопределить в каком-то из наследников.
* Класс, имеющий хотя бы одну чисто виртуальную функцию, называется абстрактным. Нельзя создать
  экземпляр абстрактного класса, от него можно только наследоваться (`Widget` в примере сверху -
  абстрактный).

**Slicing**

* У полиморфных классов есть совместимость по ссылкам, но не по значению.
* Например, имеем функцию `bar(Base b)`, которая принимает именно значение `b`. В таком случае, при
  попытке передать туда наследника `Base`, он "урежется" (произойдёт slicing), наследник лишится
  всех своих полей и превратится в `Base`. В этой функции теперь никак не получить оригинальный
  объект. [Пример](https://github.com/hse-spb-2021-cpp/lectures/blob/master/09-211110/03-casts.cpp)
* Как защититься: не передавать по значению, а вместо этого хранить ссылки/умные указатели.

**Хранение полиморфных объектов в контейнерах**

* Можем хранить фактически разные объекты, наследующиеся от одного класса, в одном контейнере. Все
  элементы контейнера должны быть одного типа, но у нас есть совместимость по ссылкам! Будем хранить
  в контейнере (умные, чтобы чистилась память) указатели на базовый класс, тогда можем сложить туда
  любого
  наследника. [Пример](https://github.com/hse-spb-2021-cpp/lectures/blob/master/09-211110/15-ptrs-good.cpp)
* Суть примера:

```c++
vector<std::unique_prt<Base>> vec;
vec.emplace_back(std::make_unique<Derived1>());
vec.emplace_back(std::make_unique<Derived2>()); // works fine! Don't forget about virtual destructor
```

**`dynamic_cast` для полиморфных классов**

* Хотим понять по ссылке на базовый класс, кем он на самом деле является.
* `dynamic_cast` пытается задаункаститься к типу, который мы указали (
  например, `dynamic_cast<Derived1 *>`). Если у него получается это сделать, он возвращает указатель
  на `Derived1`, иначе - `nullptr`.
* Синтаксис: `Derived1 *d1 = dynamic_cast<Derived1 *>(&b)`, `b` - это ссылка на `Base`
  . [Пример](https://github.com/hse-spb-2021-cpp/lectures/blob/master/10-211117/01-rtti/03-dynamic-cast.cpp)
* Если в `Base` нет виртуальных функций, `dynamic_cast` не сможет получить информацию о наследниках
  и просто не скомпилируется.
* Ещё `dynamic_cast` умеет аналогично кастовать ссылку к ссылке (в таком случае вместо `nullptr`
  происходит ошибка, которую можно обработать ручками)