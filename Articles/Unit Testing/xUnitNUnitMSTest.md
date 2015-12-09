 

 

Unit Testing Frameworks
=======================

 

Доброго времени суток. Я уже затрагивал тему тестирования в своей предыдущей
статье, где так и не смог ответить на вопрос: стоит ли Вам использовать SpecFlow
для тестирования своего приложения? и сейчас я хотел бы затронуть еще один
спорный вопрос в данной области: какой инструмент лучше всего использовать для
юнит тестирования: Nunit, xUnit или же MSTest. До недавнего времени я абсолютно
не задумываясь использовал Nunit, просто потому, что так случилось - это первый
инструмент который освоил, его использовали в моем первом проекте, да и на
сколько я знаю все в нашей фирме пишут тесты именно с помощью Nunit. И вообще,
поспрашивав своих знакомых, могу предположить, что не много разработчиков, могут
ответить на вопрос: почему это, а нет то. Да и по праву. Все эти инструменты
хорошо справляются со своей задачей. Однако, отличия в них все-таки есть, и не
только в названии и синтаксисе, как утверждает автор одной из статей, которые я
прочитал. Значит попробуем разобраться. Одно и практически единственное
структурированное сравнение всех трех инструментов я нашел на сайте xunit.
Предлагаю ознакомиться:

 

 

| NUnit 2.2             | MSTest 2005         | xUnit.net 2.x                     | Comments                                                                                                                                  |
|-----------------------|---------------------|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| ATTRIBUTES            |                     |                                   |                                                                                                                                           |
| [Test]                | [TestMethod]        | [Fact]                            | Marks a test method.                                                                                                                      |
| [TestFixture]         | [TestClass]         | *n/a*                             | xUnit.net does not require an attribute for a test class; it looks for all test methods in all public (exported) classes in the assembly. |
| [ExpectedException]   | [ExpectedException] | `Assert.Throws`                   | xUnit.net has done away with the ExpectedException attribute in favor of`Assert.Throws`.                                                  |
|                       |                     | `Record.Exception`                |                                                                                                                                           |
| [SetUp]               | [TestInitialize]    | Constructor                       | We believe that use of` [SetUp]` is generally bad. However, you can implement a parameterless constructor as a direct replacement         |
| [TearDown]            | [TestCleanup]       | IDisposable.Dispose               | We believe that use of` [TearDown]` is generally bad. However, you can implement` IDisposable.Dispose`as a direct replacement             |
| [TestFixtureSetUp]    | [ClassInitialize]   | IClassFixture\<T\>                | To get per-class fixture setup, implement` IClassFixture<T>` on your test class.                                                          |
| [TestFixtureTearDown] | [ClassCleanup]      | IClassFixture\<T\>                | To get per-class fixture teardown, implement` IClassFixture<T>` on your test class                                                        |
| *n/a*                 | *n/a*               | ICollectionFixture\<T\>           | To get per-collection fixture setup and teardown, implement`ICollectionFixture<T>`on your test collection.                                |
| [Ignore]              | [Ignore]            | [Fact(Skip="reason")]             | Set the Skip parameter on the `[Fact]` attribute to temporarily skip a test.                                                              |
| [Property]            | [TestProperty]      | [Trait]                           | Set arbitrary metadata on a test                                                                                                          |
| *n/a*                 | [DataSource]        | `[Theory]`                        | Theory (data-driven test).                                                                                                                |
|                       |                     | `[XxxData]`                       |                                                                                                                                           |
| ASSERTION             |                     |                                   |                                                                                                                                           |
| AreEqual              | AreEqual            | Equal                             | MSTest and xUnit.net support generic versions of this method                                                                              |
| AreNotEqual           | AreNotEqual         | NotEqual                          | MSTest and xUnit.net support generic versions of this method                                                                              |
| AreNotSame            | AreNotSame          | NotSame                           |                                                                                                                                           |
| AreSame               | AreSame             | Same                              |                                                                                                                                           |
| Contains              | Contains            | Contains                          |                                                                                                                                           |
| DoAssert              |                     |                                   |                                                                                                                                           |
| *n/a*                 | DoesNotContain      | DoesNotContain                    |                                                                                                                                           |
| *n/a*                 | *n/a*               | DoesNotThrow                      | Ensures that the code does not throw any exceptions                                                                                       |
| Fail                  | Fail                | *n/a*                             | xUnit.net alternative:`Assert.True(false, "message")`                                                                                     |
| Greater               | *n/a*               | *n/a*                             | xUnit.net alternative:` Assert.True(x > y)`                                                                                               |
| Ignore                | Inconclusive        |                                   |                                                                                                                                           |
| *n/a*                 | *n/a*               | InRange                           | Ensures that a value is in a given inclusive range (note: NUnit and MSTest have limited support for `InRange` on their `AreEqual`methods) |
| IsAssignableFrom      | *n/a*               | IsAssignableFrom                  |                                                                                                                                           |
| IsEmpty               | *n/a*               | Empty                             |                                                                                                                                           |
| IsFalse               | IsFalse             | False                             |                                                                                                                                           |
| IsInstanceOfType      | IsInstanceOfType    | IsType                            |                                                                                                                                           |
| IsNaN                 | *n/a*               | *n/a*                             | xUnit.net alternative:`Assert.True(double.IsNaN(x))`                                                                                      |
| IsNotAssignableFrom   | *n/a*               | *n/a*                             | xUnit.net alternative:`Assert.False(obj is Type)`                                                                                         |
| IsNotEmpty            | *n/a*               | NotEmpty                          |                                                                                                                                           |
| IsNotInstanceOfType   | IsNotInstanceOfType | IsNotType                         |                                                                                                                                           |
| IsNotNull             | IsNotNull           | NotNull                           |                                                                                                                                           |
| IsNull                | IsNull              | Null                              |                                                                                                                                           |
| IsTrue                | IsTrue              | True                              |                                                                                                                                           |
| Less                  | *n/a*               | *n/a*                             | xUnit.net alternative:` Assert.True(x < y)`                                                                                               |
| *n/a*                 | *n/a*               | NotInRange                        | Ensures that a value is not in a given inclusive range                                                                                    |
| *n/a*                 | *n/a*               | Throws                            | Ensures that the code throws an exact exception                                                                                           |

 

Исходя из этой таблице сразу видно преимущества xUnit над конкурентами, ну все
таки не зря эта информация у них на сайте. Но, к сожалению, подобных таблиц я
больше не нашел, да и здесь (как Вы, наверно, заметили) рассматриваются далеко
не последние версии, даже самого xUnit. Однако, почитав другие источники и кое
что проверив, могу сказать что сравнения справедливы, но не отображают полной
картины происходящего. Рассмотрим системы подробнее.

 

MSTest

 

Начну сразу с минуса, который часто упоминаться при обсуждении: очень много
времени на выполнение теста. Кроме этого для выполнения этих тестов обязательна
студия, так как нет отдельного ранера. Что касается плюсов, я бы хотел
акцентировать внимание на том, что кроме этой самой студии больше ничего не
нужно, все уже встроено и подготовлено к работе. Что еще мне понравилось в
MSTest, так это возможность тестировать приватные поля и методы. Пример подобной
реализации предоставлен ниже.

 

`[TestClass]`

`public class TestClass`

`{`

`	[TestMethod]`

`	public void TestPrivateMethod()`

`	{`

`		ClassToTest testedClass = new ClassToTest();`

`		PrivateObject privateObject = new PrivateObject(testedClass);`

 

`		privateObject.SetField("field", "Don't panic");`

`		privateObject.Invoke("PrintField");`

`	}`

`}`

 

 

xUnit

 

"Сам себя не похвалишь - никто не похвалит" - именно так можно было бы подумать
посмотрев на таблицу представленную выше, но это не так. Этот инструмент хвалят.
И хвалят не только рядовые пользователи системы, но и Microsoft, так как в ASP
.NET 5 для тестирования по умолчанию вместо Visual Studio Unit Testing Framework
(иногда называемый mstest) для модульного тестирования используется фреймворк
xUnit.net. Кое кто говорит и более простом синтаксисе и минимуме дополнительных
атрибутов, что вообщем спорно - тут кому как. А в более ранних версиях отмечали
поддержку асинхронных тестов, но в этом xUnit уже догнали и другие фреймворки.
Лично меня вдохновил их способ тестирования ожидаемых исключений - ранние
сталкивался с подобной проблемой. Вообщем самая новая, самая классная система
юнит тестирования из минусов которой я смог найти только отсутствие
взаимодействия с продукцией любимого JetBrains, что конечно же неприятно.

 

Nunit

 

"Удобно и мне нравиться" - вот что я мог бы сказать об этой системе юнит
тестирования, после достаточно долгого сотрудничества. Я не прочувствовал на
практике в чем эта система уступает xUnit - лично мне хватило инструментов что
бы протестировать свои проекты. К тому же в версии NUnit 3.0, которая вышла
недавно, разработчики обещали догнать xUnit. Посмотрим, но если не станет хуже,
я и дальше буду писать тесты с помощью NUnit.

 

Выводы

 

Я снова не ответил на вопрос который задал в начале: "какой инструмент лучше
всего использовать для юнит тестирования?", возможно потому что на него нет
правильного ответа. Но, надеюсь, мне удалось собрать в одну статью все основные
моменты, отличия и схожести данных системы, и теперь Вы сможете выбрать
инструмент себе по душе. И если Вам кажется, что я просто похвалил каждый
фреймворк, так это потому, что покрывать тестами код - это хорошо, круто и
правильно.

 
