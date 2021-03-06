---
layout: post
title:  "JavaScript и аннотации Platypus.js"
date:   2015-09-16 10:37:27
disqus: "subject.js"
categories: javascript platypus
---

Вместе с бурным и почти неуправляемым развитием скриптовых движков, фреймворков и библиотек
на первый план начало выходить отношение прикладной логики к технологиям,
применяемым в разработке.

Различные JavaScript контейнеры и библиотеки предлагают разные технологии и способы их использования.
В Node.js, например, есть асинхронная модель ввода-вывода, но ее механизмы (poll/epoll в библиотеке [libuv](https://github.com/libuv/libuv) и т.п.) скрыты от скриптового приложения.
Разработчику предлагают использовать удобные callback - функции в купе с замыканиями.

В браузерах и в Io.js теперь есть еще и библиотека WebWorkers, но разве разработчик скриптового приложения
пользуется в своей программе объектами синхронизации? Слава богу нет. Ему также предлагают пользоваться callback-ами.

Любой ORM предназначен для отделения прикладной логики приложения от технологий хранения данных.

А теперь давайте рассмотрим, что нам предлагают сделать со своей программой различные JavaScript библиотеки.

Библиотека для binding-а, Knockout.js:

{% highlight javascript linenos=table %}
function TaskListViewModel() {
  // Data
  var self = this;
  self.tasks = ko.observableArray([]);
  self.newTaskText = ko.observable();
  self.incompleteTasks = ko.computed(function () {
    return ko.utils.arrayFilter(
        self.tasks(),
        function (task) {
          return !task.isDone()
        });
    });
}
ko.applyBindings(new TaskListViewModel());
{% endhighlight %}

Такой код изобилует логикой, специфичной для библиотеки Knockout.js.
Посмотрите хотя бы на ko.computed()

ORM для Node.js BookShelf.js:

{% highlight javascript linenos=table %}
var Customer = bookshelf.Model.extend({
    initialize: function () {
        this.on('saving', this.validateSave);
    },
    validateSave: function () {
        return checkit(rules).run(this.attributes);
    }
});
{% endhighlight %}

Этот ORM принуждает нас наследовать классы предметной области от библиотечного класса bookshelf.Model.

Библиотека для работы с событиями, backbone.js:

{% highlight javascript linenos=table %}
var Subject = {};
_.extend(Subject, Backbone.Events);
Subject.on("alert", function (msg) {
    alert("Triggered " + msg);
});
Subject.trigger("alert", "an event");
{% endhighlight %}

Здесь _.extend инжектирует некоторые свои методы в наш объект и после этого мы можем обрабатывать события, вызванные функцией .trigger()

Предположим, что нам это все не мешает и мы радостно включаем такой код в свою программу.
А что будет с нашей прикладной логикой, если мы захотим отказаться от одной из используемых в приложении библиотек?
Её придется переписывать заново! Полностью!

Теперь давайте посмотрим что нам предлагает для решения этой проблемы индустрия в целом.

JavaEE предоставляет разработчику механизм аннотаций.

Например, для того, чтобы сделать модуль с прикладной логикой Enterprise Java Bean (EJB), нужно создать класс, содержащий прикладную логику и снабдить его аннотацией @ejb.

{% highlight Java linenos=table %}
@ejb
public class Customer{

    public Customer(){
        super();
    }

    public int calculateRank(){
        //...
    }
}
{% endhighlight %}

Для того, чтобы управлять жизненным циклом такого серверного модуля, нужны аннотации @statefull и  @stateless.

{% highlight Java linenos=table %}
@ejb
@stateless // @statefull
public class Customer{

    public Customer(){
        super();
    }

    //...
}

{% endhighlight %}

Обратите внимание, что ни наследовать класс Customer от какого-нибудь AbstractEJB, ни что то инжектировать в наш класс, принадлежащий предметной области, не нужно.
Также для разграничения прав доступа, используется аннотация @rolesAllowed и т.п.
Таким образом, получается что в вашем приложении есть некий модуль/класс принадлежащий предметной области, который содержит чистую прикладную логику,
а за то какой у него будет жизненный цикл, кто к нему будет иметь доступ, является ли он вообще управляемым и прочие технологические подробности отвечает JavaEE сервер.
И самое приятное заключается в том, что аннотации вовсе не являются частью прикладной логики!

.NET, кстати, предлагает похожий [механизм атрибутов](https://en.wikipedia.org/wiki/Metadata_(CLI)).

В мире JavaScript не так то легко найти библиотеки и контейнеры, которые позволяли бы применять нечто похожее на механизм аннотаций JavaEE или .NET атрибутов.

[Platypus.js](http://platypus-platform.org/) как контейнер JavaScript поддерживает механизм аннотаций, размещаемых в блоках JsDoc.
Он позволяет максимально изолировать технологическую часть приложения от прикладной логики подобно тому, как это делается в Java EE для Java приложений.
Например, для того чтобы сделать глобальный серверный модуль, существующий в контексте всего сервера, надо снабдить его конструктор аннотацией @resident.

{% highlight javascript linenos=table %}

/**
 *  @resident
 */
function SingletonModule() {
}

{% endhighlight %}

Для того, чтобы сделать серверный модуль в сессии без сохранения состояния, надо снабдить его аннотацией @stateless.

{% highlight javascript linenos=table %}

/**
 *  @stateless
 */
function SessionModule() {
}

{% endhighlight %}

В Platypus.js предусмотрено взаимодействие между сессионными и глобальными модулями, но это выходит за рамки данной статьи и я напишу об этом как-нибудь в другой раз.
Набор аннотаций, поддерживаемых Platypus.js намного шире чем @resident и @stateless.
Об этом можно почитать в [документации.](http://platypus-platform.org/docs/eng/html/Development_Guide/index.html)

У аннотаций, конечно, есть и недостатки.
Во-первых нужен контейнер, который все это поддерживает, во-вторых,
для того чтобы приложение было полностью переносимым, нужно сделать аннотации стандартизированными.

С другой стороны, код прикладной логики, содержащийся в модулях SingletonModule и SessionModule
является полностью переносимым, т.к. не содержит вообще никаких библиотечных вызовов,
отвечающих за технологическую часть приложения.

Получается, что принципы отделения прикладной логики от технологий, используемых в приложении встречаются
не только в JavaEE, .NET, но и в JavaScript.
