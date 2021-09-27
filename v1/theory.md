# Questions and answers

### What is Application server and Web Server

Application server - сервер приложения который используеться для взаимодействия с нашим бекендом.(Puma, Passenger, Unicorn)

NGinX - проксисервер, используется для ускорения работы сайта и позволяет отдавать статику напрямую, а также уменьшить нагрузку.
Since Unicorn is running in a blocking-I/O model, if there are some clients with poor internet connection(slow clients), their request will block Unicorn to process further requests from other clients. Put simply, one Unicorn worker can only handle one request at a time, if we have 5 worker running and 5 slow clients connecting to each of them, these 5 workers won’t be able to accept other client request.(The mechanism about Nginx leveraging OS kernel for load balancing among workers is another interesting topic — Linux Async IO).

### What is Rack

Rack provides a minimal API for connecting web servers and web frameworks[introducing rack]. It’s inspired from WSGI, and you might find some similarities between them if you come from Python landscape.

### DSL in Ruby

The term Domain Specific Language is used to describe some of the concepts in ActiveRecord, which is part of Ruby on Rails. The concept of a DSL is actually unrelated to Ruby, and has been around for a long time. It's generally used to describe a language that has constructs suited for expressing concepts of a particular domain. The build tool Make, for instance, has a language that allows to declaratively define build targets, how to build them and dependencies between them. Then there is the concept of an internal DSL, which uses the syntax of an existing language, a host language, such as Ruby. The means of the language are used to build constructs resembling a distinct language. The, already mentioned, Rake uses this to make code like this possible:

###  Include, extend, prepend

Все три метода связаны с реализацией синглтон метода класса. Объекты в Ruby не хранят свои собственные методы. Вместо этого они создают синглтон-класс, чтобы он хранил их методы. 
Оба добавляют методы смешанного модуля к переданному модулю (классу). Разница заключается в порядке поиска этих методов, если они уже определены в целевом классе:

- __include__ ведет себя так, как если бы целевой класс унаследовал смешанный модуль (добавляет методы модуля объекту)
- __prepend__ создает методы из смешанного в модуле "stronger" и сначала выполняет их
- __extend__ вызывает include для синглтон-класса объекта
Используйте prepend , если вы хотите сохранить методы целевого модуля (класса) в конце цепочки поиска методов.

### Metaprogramming

Метапрограммирование в Ruby: это все о self и механизме работы синглтон методов.

### SOLID, GRASP, YAGNI, KISS, DRY

**SOLID** - 5 оосновных принципов ООП введенных Майклом Фэзерсом в начале нулевых. Эти принципы — часть общей стратегии гибкой и адаптивной разработки, их соблюдение облегчает расширение и поддержку проекта.

- **Single Responsibility Principle** (Принцип единственной ответственности)
- **Open Closed Principle** (Принцип открытости/закрытости)
- **Liskov Substitution Principle** (Принцип подстановки Барбары Лисков)
- **Interface Segregation Principle** (Принцип разделения интерфейса)
- **Dependency Inversion Principle** (Принцип инверсии зависимостей)

**YARNI** - you aren't need it (тебе это не понадобится) процесс и принцип проектирования ПО, при котором в качестве основной цели и/или ценности декларируется отказ от избыточной функциональности, — то есть отказ добавления функциональности, в которой нет непосредственной надобности.

**KISS** - keep it simple as possible (or stupid).

**DRY** - don't repeat yourself. Если есть часть кода которая дублируется в нескольких частях, ее нужно вынести в отдельную функцию.