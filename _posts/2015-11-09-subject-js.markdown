---
layout: post
title:  "Platypus.js ORM. Чистая бизнес логика"
date:   2015-11-09 10:14:07
disqus: "orm.js"
categories: javascript platypus
---

JavaScript eat the world! Радостно восклицают в последнее время энтузиасты разработки
JavaScript приложений. В самом деле, в последнее время сильно возросло количество реально
работающих сервисов, программное обеспечение которых написанно на JavaScript.

Не секрет, что разработка таких приложений в большой степени связана с обработкой данных,
хранящихся в базах данных. Вот здесь и начинаются интересные проблемы.
Слава богу, давно прошли те времена, когда разработчики для того чтобы поработать с хранилищем данных, 
были вынуждены помещать в своих программах код вот такого вида:

{% highlight javascript linenos=table %}
var results = executeQuery("some sql select text”);
while(!results.eof()){
    results.first();
    var address = results.fieldByName("address”).asString();
    var phone = results.fieldByName("phone”).asString();
    var email = results.fieldByName("phone”).asString();
    var dateOfBirth = results.fieldByName("birth”).asDate();
}
{% endhighlight %}

Причем эта "болезнь" была присуща разным технологиям.
Помнится только так можно было работать с ClientDataSet из бибилиотеки MIDAS Delphi, - покойся с миром.
Теперь на смену такому неуклюжему подходу пришли ORM-ы и NoSQL базы данных.
В этой статье мне хотелось бы поговорить именно об ORM-ах.
Сейчас достаточно использовать какой-нибудь ORM и данные из любого источника станут доступны
как простые переменные и массивы.

{% highlight javascript linenos=table %}
var customers = Models.find("select * from customers ...");
customers.forEach(function(aCustomer){
    ...
});
var fifthCustomer = customers[4];
If(customers[2].name === ""){
    var cName = prompt("Введите имя");
    customers[2].name = cName;
}
{% endhighlight %}

Примеров достаточно. Hibernate, TopLink, EclipseLink для Java, Linq для .NET, CompoundJS и ряд других для Node.js, Platypus.js и Avatar.js.
Но спросите себя, какую цену пришлось нам заплатить за такое удобство?

# Сопоставление сущностей и классов

Теперь приходится заниматься нуднейшим делом, - сопоставлять модель данных в программе с сущностями базы данных,
для того чтобы ORM имел представление о том какие объекты нужно построить из данных, прочитанных в базе.
Например, в рекламном ролике Wakanda демонстрируется красивый редактор сущностей, который умеет добавлять поля, связи и т.п.
и говорится о ценной возможности делать то же самое простым JS-кодом.
В случае Hibernate для Java, приходится делать xml сопоставление полей базы данных и свойств какого-то класса из нашей программы.
Но не все так плохо. Впоследствии, в Hibernate появилась возможность сопоставлять классы предметной области и сущности базы данных с помошью аннотаций.

А является ли это все необходимым?

Когда мы пишем SQL запрос, то мы делаем это для того, чтобы получить в свою программу данные, которые мы хотим использовать, а не просто так гонять по сети.
Так может быть можно использовать схему данных непосредственно из запроса в качестве структуры нашего класса?

![model sql](/assets/sql-model.png)

Вот такая модель класса предметной области получается по автоматически на основе алиасов полей: 

![model result](/assets/result-model.png)

Для статически типизированных языков, таких как Java или C# использовать фактическую форму данных
для конструирования объектов невозможно.
Однако для динамических языков, таких как JavaScript, Groovy и прочие, это «то что доктор прописал».
Platypus.js предоставляет такой ORM, который строит объекты JavaScript непосредственно по данным возвращающимся их базы.
При этом учитываются связи между сущностями через механизм разыменовывания ссылок по внешним ключам. 
Так появляются ссылки на связанные объекты и коллекции связанных обектов на другой стороне.
Platypus.js может строит JavaScript объекты с помощью пользовательских конструкторов, а за неимением таковых,
просто с помощью оператора {}.

# Производительность

Использование ORM-ов таит в себе ещё более серьезную проблему.
Они самостоятельно генерируют SQL запросы, не обращая внимания на такие «мелочи» как индексы и тому подобные подробности хранения данных и гордятся этим!
В результате, мы сделав все по правилам, получаем приложение которое «задыхается» после того как объем данных вырастает до нескольких тысяч записей.
А оптимизация запросов, генерируемых ORM-ом, вообще попахивает магией.
Конечно можно сказать что Hibernate, например, позволяет написать SQL собственноручно,
но тогда зачем нужна добрая половина его функциональности?
Стоит ли упоминать о том, что там есть свой собственный Hibernate query language (HQL) представляющий собой фасад над SQL c Hibernate в качестве препроцессора?

Platypus.js, напротив, работает с уже готовыми SQL запросами и дает полную свободу в их написании.
И, похоже, что это единственный адектватный способ избежать некотроллируемой генерации запросов.
Кроме того, Platypus.js поддерживает работу с различными источниками данных,
что позволяет получать данные из таких хранилищ как MongoDB, CouchDB, Cassandra и т.п.

Модель данных в приложении строится просто добавлением нужного запроса в модель данных модуля.
Platypus.js исполнит запрос, сконструирует скриптовые объекты и даст приложению доступ к ним с помощью всего лишь одной переменной.
Получаемые элементы данных представляют собой JavaScript объекты, по умолчанию создаваемые как {}.
Platypus.js ORM не требует наследования объектов предметной области от каких - нибудь Platypus.Model, в отличие от Bookshelf, например.

Platypus.js имеет встроенное средство построения моделей в пространстве скриптового приложения.
Эти модели управляются общим entity manager-ом – моделью данных, - одним на модуль.
Всё специфичное взаимодействие с ним сведено к минимуму обязательных методов (model.save(), model.revert(), model.requery(); entity.requery() ).

{% highlight javascript linenos=table %}
/**
 * Device communication module.
 * @acceptor asc6
 */
define(['orm', 'gps-packet', 'Logger'], function(Orm, Packet, Logger, ModuleName){
    return function () {
        var self = this;
        var model = Orm.loadModel(ModuleName);
        model.packets.elementClass = Packet;
        var packets = model.packets;

        self.flush = function () {
            model.save(function (aAffected) {
                Logger.info("track point saved");
            }, function (e) {
                Logger.severe("track point failed");
            });
    };
};
{% endhighlight %}

Слежение за изменениями в данных и двусторонний data-binding с виджетами в браузере
инжектируется в уже существующие объекты и классы, не требуя явных вызовов applyBindings() или .extend().
Коллекции, порождаемые  методами requery() состоят из объектов, созданных с помощью пользовательских конструкторов,
в которых не требуется специальной логики, связанной с чтением, записью или валидацией данных.
Получается, что в прикладной программе существуют классы с методами и свойствами,
нужными только для её логики, а все остальное Platypus.js берет на себя.

Хотелось бы видеть побольше JavaScript фреймворков, которые стремятся сохранить прикладную логику
приложений в чистом виде и сделать её максимально переносимой.
