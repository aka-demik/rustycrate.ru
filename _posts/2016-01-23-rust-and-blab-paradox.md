---
title: "Rust и парадокс Блаба"
categories: размышления
published: true
author: Jonathan Turner (перевёл Иван Иващенко)
---

*Эта статья – перевод статьи
[Rust and the Blub Paradox](http://www.jonathanturner.org/2016/01/rust-and-blub-paradox.html)
за авторством Jonathan Turner*

Несколько недель назад я наткнулся на
[сравнительный анализ Rust, D и Go](https://www.quora.com/Which-language-has-the-brightest-future-in-replacement-of-C-between-D-Go-and-Rust-And-Why)
от Андрея Александреску. Андрей,
[уважаемый член сообщества C++](http://www.amazon.com/Modern-Design-Generic-Programming-Patterns/dp/0201704315)
и главный разработчик [языка программирования D](http://dlang.org/), нанес Rust
сокрушительный удар под конец своего повествования, высказав нечто, что выглядит довольно
проницательным наблюдением:

> Чтение кода на Rust навевает шутки о том, как "друзья не позволяют друзьям пропускать день ног" и
вызывает в голове комические образы мужчин с халкообразным торсом, балансирующим на тощих ногах.
Rust ставит во главу угла безопасность и ювелирное обращение с памятью. В действительности, это
довольно редко является настоящий проблемой, и такой подход превращает процесс мышления и написания
кода в монотонный и скучный процесс.

После нескольких встреч с Андреем, увидев некоторые из его выступлений, я убедился, что он *любит
подшучивать*. Тем не менее, давайте проглотим наживку. Эта шутка смешная только потому, что она
выглядит смешной, или может быть потому, что в ней только доля шутки?

<!--cut-->

## Парадокс Блаба

Всякий раз, размышляя о пользе тех или иных возможностей языков программирования, я возвращаюсь к
эссе Пола Грэма ["Побеждая посредственность"](http://www.nestor.minsk.by/sr/2003/07/30710.html).
В нем повествуется об интересном явлении среди программистов, которое он называет "Парадокс Блаба".
Для тех, кто не в курсе, парадокс звучит примерно так: Допустим есть программист, который использует
некий язык Блаб. С точки зрения своей выразительности, Блаб находится где-то посередине континуума
абстрактности среди всех языков программирования. Это не самый примитивный, но и не самый мощный
язык программирования.

Когда наш Блаб-программист смотрит на "нижнюю" часть спектра языков программирования, он с легкостью
замечает, что эти языки являются менее выразительными, чем его любимый Блаб. Но когда наш
гипотетический программист смотрит на "верхнюю" часть спектра, обычно он не осознает, что в
действительности смотрит вверх. Вот как это описывает Пол:

> Все что он видит, это просто "странные" языки. Возможно, он воспринимает их как равносильные
Блабу, только в них еще куча стремной и непонятной фигни. Блаба для нашего программиста
вполне достаточно, поскольку он сам думает на Блабе.

Помню, когда я впервые прочитал это, я подумал: "воу, это довольно проницательно". Кто бы мог
подумать, что годы спустя эта концепция прочно укоренится в моем образе мышления, когда я начал
пытаться учить людей программированию.

Будучи руководителем проектов по языкам в Microsoft, я работаю над [TypeScript](http://www.typescriptlang.org/) –
типизированной версией Javascript. В обязательном порядке, когда я выступаю перед аудиторией
преимущественно JavaScript разработчиков и пытаюсь донести мысль о том, как здорово было бы попробовать добавить
немного строгой типизации в Javascript, на меня смотрят хмурые лица. Всякий раз. Даже если она не
обязательна. Даже после того как я опишу полдюжины преимуществ. Как и говорил Пол, это выглядит
просто "странно". Для JavaScript-программистов TypeScript выглядит в основном тем же что и
JavaScript, плюс куча стремной и непонятной фигни.

Пообщавшись с командами других языков программирования, а также наблюдая за все большим количеством
людей на конференциях, я осознал, что наблюдение Пола является не только метким, но еще и на
удивление универсальным. Большинство программистов готовы отбиваться изо всех сил, увидев новый язык
программирования, который они никогда не использовали. Новые, чуждые им особенности вызывают у них
аллергическую реакцию. Только поработав с новыми возможностями достаточно долгое время, они начинают
понимать, что все это не просто бесполезные приблудины.

Короче говоря, парадокс Блаба – это нечто, с чем мы как программисты должны считаться, во что мы
имеем свойство впадать, и из чего нам стоит выбираться, прикладывая все усилия.

Давайте просто сделаем это. Давайте рассмотрим несколько самых странных и бесполезных особенностей
Rust. А затем посмотрим, сможем ли мы провернуть деблабизацию.

## Странная фигня №1. Полиморфизм в стиле Rust

Давайте напишем программу на Rust, которая использует немного полиморфизма для того, чтобы
напечатать две разные структуры. Сначала я покажу вам код, а затем мы рассмотрим его в деталях.

```rust
use std::fmt;

struct Foo {
    x: i32
}

impl fmt::Display for Foo {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "(x: {})", self.x)
    }
}

struct Bar {
    x: i32,
    y: i32
}

impl fmt::Display for Bar {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "(x: {}, y: {})", self.x, self.y)
    }
}

fn print_me<T: fmt::Display>(obj : T) {
    println!("Value: {}", obj);
}

fn main() {
    let foo = Foo {x: 7};
    let bar = Bar {x: 5, y: 10};
    print_me(foo);
    print_me(bar);
}
```

Какое же оно вырвиглазное! Да, тут есть полиморфизм, но это и близко не похоже на ООП. Этот код
использует обобщения, и не только обобщения, но в таком подходе есть куча ограничений. И что это за `impl`?

Давайте по частям. Я создаю две структуры для хранения наших значений. Следующим шагом я реализую
для них нечто, называемое `fmt::Display`. В C++ мы бы перегрузили оператор `<<` для
`ostream`. Результат был бы аналогичным. Теперь я могу вызывать функцию печати, передавая свои
структуры напрямую.

Это уже половина истории.

Дальше у нас появляется функция `print_me`. Эта функция обобщенная и принимает все что угодно, если
оно умеет `fmt::Display`. К счастью, мы только что убедились, что наши структуры так умеют.

Все остальное просто. Мы создаем несколько экземпляров структур и передаем их на печать в `print_me`.

Фух... пришлось потрудиться. Так делается полиморфизм в Rust. Вся суть в обобщениях.

Теперь давайте на минуту переключимся на C++. Многие, особенно новички, могли не сразу додуматься
до использования шаблонов, и пошли бы по пути объектно-ориентированного полиморфизма:

```cpp
#include <iostream>

class Foo {
    public:
        int x;
        virtual void print();
};

class Bar: public Foo {
    public:
        int y;
        virtual void print();
};

void Foo::print() {
    std::cout << "x: " << this->x << '\n';
}

void Bar::print() {
    std::cout << "x: " << this->x << " y: " << this->y << '\n';
}

void print(Foo foo) {
    foo.print();
}

void print2(Foo &foo) {
    foo.print();
}

void print3(Foo *foo) {
    foo->print();
}

int main() {
    Bar bar;
    bar.x = 5;
    bar.y = 10;

    print(bar);
    print2(bar);
    print3(&bar);
}
```

Довольно просто, не так ли? Окей, вот вам небольшая викторина: что именно напечатает код на C++?

Если вы не угадали, не расстраивайтесь. Вы находитесь в хорошей компании.

Если угадали – мои поздравления! Теперь задумайтесь на минуту, сколько всего вы должны знать о С++,
чтобы дать правильный ответ. Из того что я вижу, вы должны понимать принципы работы стека, как
объекты копируются, когда они копируются, как работают указатели, как работают ссылки, как устроены
виртуальные таблицы и что такое динамическая диспетчеризация. Просто чтобы написать несколько
простых строк в стиле ООП.

Когда я начинал изучать C++, этот подъем оказался слишком крутым для меня. К счастью, мой
двоюродный брат оказался экспертом по C++, и, взяв меня под свое крыло, он показал мне несколько
проторенных дорожек. Тем не менее, я успел натворить тонны детских ошибок, вроде этого примера.
Почему? Одной из причин неприступности С++ является высокая когнитивная нагрузка при его освоении.

Часть когнитивной нагрузки приходится на вещи, которые присущи программированию по своей сути.
Вы должны понимать стек. Вы должны знать как работают указатели. Но С++ повышает
степень нагрузки, требуя понимания того, в каких случаях значение будет скопировано
не полностью, и когда виртуальная диспетчеризация используется, а когда не используется – и все это
без каких-либо предупреждений от компилятора, если разработчик делает что-то, что "скорее всего
является плохой идеей™".

Это не попытка пойти войной против С++. Многие вещи в Rust реализованы с мыслью сохранить философию
низкоуровневых и эффективных абстракций, взятой из C++. Вы даже можете написать код, который будет
[очень похож на пример Rust](https://gist.github.com/jonathandturner/a182347b763398d8ea4f).

Что Rust действительно делает – так это отделяет наследование от полиморфизма, подталкивая вас
мыслить в направлении создания обобщений с самого начала. Таким образом вы начинаете думать обобщенно с
первого дня. Тем не менее, отделение наследования от полиморфизма может показаться странной идеей,
особенно если вы привыкли всегда использовать их вместе.

Такое разделение может вызвать одно из первых проявлений Блаб-эффекта: в чем вообще
преимущество разделять наследование и полиморфизм? И кстати, в Rust вообще есть наследование?

Хотите верьте, хотите – нет, но по крайней мере в Rust 1.6 нет вообще никаких специальных
инструментов для наследования
структур. Вместо этого их функциональность наращивается за пределами самих структур, с помощью
особенной концепции языка – "типажей". Типажи позволяют добавлять методы, требовать реализации методов, и
всячески дооснащать структуры данных в уже существующих системах. Также типажи поддерживают
наследование: один типаж может расширять другой.

Если хорошенько покопаться, можно заметить еще кое-что. В Rust нет всех тех проблем,
о которых нам пришлось беспокоиться на С++. Мы можем больше не думать о том, как что-то теряется,
когда функция вызывается каким-то образом, и какое влияние оказывает виртуальная диспетчеризация
на наш код. В Rust все работает в едином стиле, независимо от типа. Таким образом целый класс
детских ошибок просто исчезает.

## Странная фигня №2. В смысле, нет исключений?

Раз уж мы заговорили о вещах, которых в Rust нет, следующей странной фигней будет отсутствие исключений.
Разве это не шаг назад? Как нам поступать с ошибками?
Можем ли мы пробрасывать их наверх, чтобы обрабатывать все сразу в одном месте?

Что ж, пришло время познакомиться с монадами.

Хотя... ладно, шучу, на этот раз можно обойтись без них. В Rust обработка ошибок гораздо более
прямолинейна. Вот пример того, как это выглядит на практике. Для начала, примеры того, как будет
выглядеть объявление функций:

```rust
impl SystemTime {
  /// Возвращает текущее системное время
  pub fn now() -> SystemTime;

  /// Возвращает ошибку, если переданное "раньше" окажется позже
  pub fn duration_from_earlier(&self, earlier: SystemTime) -> Result<Duration, SystemTimeError>;
}
```

Обратите внимание, что функция `now` просто возвращает `SystemTime` и не имеет каких-либо
исключительных ситуаций, в то время как `duration_from_earlier` возвращает тип `Rеsult`, который
может принимать значения как `Duration`, так и `SystemTimeError`. Таким образом, вы сразу видите все
возможные исходы выполнения функций, как успешные, так и не успешные.

Но все эти исключительные ситуации создают кашу в возвращаемых значениях. Кто захочет видеть такое
в своем коде? Здорово, конечно, всегда делать проверки на ошибки, но смысл исключений заключается как раз в
том, что они позволяют обрабатывать ошибки не только локально, но и пробрасывать их наверх, выполняя
обработку в одном месте.

И Rust позволяет вам сделать тоже самое.

```rust
fn load_header(file: &mut File) -> Result<Header, io::Error> {
  Ok(Header { header_block: try!(file.read_u32()) })
}

fn load_metadata(file: &mut File) -> Result<Metadata, io::Error> {
  Ok(Metadata { metadata_block: try!(file.read_u32()) })
}

fn load_audio(file: &mut File) -> Result<Audio, io::Error> {
  let header = try!(load_header(file));
  let metadata = try!(load_metadata(file));
  Ok(Audio { header: header, metadata: metadata })
}
```

Хотя это не совсем очевидно, этот код использует пробрасывание исключений. Вся
фишка в макросе `try!`. Он делает достаточно простую вещь. Он вызывает функцию. Если она завершится
успешно, он вручит результат вычислений вам. Если вместо этого случится ошибка, `try!` пробросит эту
ошибку, завершив выполнение текущей функции.

Это означает, что если у `load_header` будут какие-либо проблемы при вызове
`file.read_u32`, то функция вернет `io::Error`. Далее, то же произойдет в `load_audio`, и из нее
будет возвращена та же ошибка. И так далее до тех пор, пока вызывающая функция наконец не
обработает ошибку.

## Странная фигня №3. Борроу-чекер

Вы знаете, это забавно. Первое, что упоминают многие люди, говоря о Rust – это borrow checker.
Более того, его часто преподносят как основную особенность Rust, выделяющую его среди других языков
программирования. Например, для Андрея, borrow checker – это "халкообразный торс" Rust. Для меня же
borrow checker – это просто еще одна проверка компилятора. Так же, как проверка на соответствие типов,
borrow сhecker позволяет отловить большинство багов до того, как они произойдут во время выполнения.
Вот и все. Конечно, по началу он может показаться монструозной штуковиной, но я посмею утверждать,
что дело тут не в том, что Rust заставляет вас изучать какую-то новую непонятную систему типов, а в
том, что умение работать с ним наращивает новые мускулы у вас как программиста.

Так какие ошибки отлавливает borrow checker, спросите вы?

### Использование указателей после освобождения памяти

О да, классическая ситуация, сначала вы освобождаете память, а затем снова ее используете. В большинстве
случаев это именно та причина, по которой программы падают с пугающими "null pointer exception".

Есть целая куча "хороших практик" C++, которые позволяют избежать use-after-free: использование RAII,
использование ссылок или умных указателей вместо сырых указателей, документирование отношений
владения и заимствования в вашем API и так далее. Все то, что по мнению Андрея "превращает процесс
мышления и написания кода в монотонный и скучный процесс". Команда хорошо натренированных С++
программистов в состоянии избежать большинство use-after-free ошибок, занимаясь монотонной и
скучной работой, потому что такова цена – соблюдение всех "хороших практик", никогда не читерить и
пополнять команду только высококвалифицированными экспертами C++.

### Невалидные итераторы

Вам никогда не приходилось модифицировать контейнер, по которому вы итерировались в C++, и получать
из-за этого внезапные падения когда-нибудь в будущем? Мне приходилось. Если вы добавили или удалили
из контейнера хотя бы один элемент, этого достаточно, чтобы потребовалось провести реаллокацию контейнера
и сделать ваш итератор невалидным.

Я не часто наступаю на эти грабли, но это все еще происходит время от времени.

### Состояния гонки данных

В Rust данные либо общие, либо изменяемые. Если данные можно изменять, их нельзя разделять между
несколькими потоками, так что нет никакой возможности начать менять их в двух потоках одновременно,
вызвав тем самым состояние гонки. Если же данные общие, их нельзя модифицировать, так что вы
можете читать их сколько вам заблагорассудится из любого количества потоков.

Если вы пришли из мира С++ или любого другого языка с множеством хороших параллельных библиотек,
такие ограничения могут показаться вам слишком строгими. К счастью, это далеко не вся история, но
это основа, дающая вам набор простых правил для создания более сложных абстракций. Остальная часть
истории пишется прямо сейчас. В экосистеме Rust появляется все большее число [библиотек,
ориентированных на параллелизм](http://areweconcurrentyet.com/). Если вам интересно узнать больше,
вы можете изучить принципы их работы.

### Отслеживание владения

Эта концепция может показаться несколько избыточной, но на самом деле это именно то, с чем постоянно
воюет C++. Ранее я упоминал об одной из хороших практик "документировать отношения владения и
заимствования в вашем API". Проблема в том, что эта информация хранится в комментариях, вместо того,
что находится непосредственно в коде.

Вот вам сценарий: вы пишете на С++ и вам необходимо вызвать библиотеку, которую написал кто-то другой.
Допустим, это библиотека на C и она принимает в качестве аргументов сырые указатели. Должны ли вы
позаботиться удалить впоследствии то, что передали в эту библиотеку? Или она возьмет на себя эту
ответственность, сохранив полученные данные в одной из своих структур? Может быть вы вызываете скриптовый
движок вроде Ruby? Кто в таком случае владеет данными?

Вместо того, чтобы вчитываться в документацию, Rust позволяет быть уверенным в ваших ожиданиях,
все время проверяя правильность использования API библиотеки с помощью borrow checker.

### И многое другое

Borrow checker помогает избежать множество других ошибок. Например, он позволяет всегда рассчитывать
на то, что любые изменяемые данные, которые вы принимаете в написанную вами функцию не влияют на
какое-либо внешнее состояние, и вы можете смело изменять их так, как посчитаете нужным.

Это, кстати, открывает широкие возможности для дополнительных оптимизаций, которые трудно произвести
в С-подобных языках программирования, поскольку компилятор гарантирует, что любое значение,
которое имеет несколько псевдонимов, не может быть изменяемым, и наоборот – изменяемое значение
всегда имеет только одно имя.

## Странная фигня №4. Правила нужны для того, чтобы их нарушить

Я считаю, что одной из самых сильных сторон Rust является его прагматичность. Большинство строгих
ограничений можно обойти с помощью таких возможностей, как `unsafe` и `mem::transmute`.
Borrow checker не подходит для решения ваших задач? Не проблема, просто отключите его.

Это позволяет вам делать все, что вы привыкли делать на C-подобных системных языках программирования.
Преимущество Rust заключается в том, что гораздо проще писать код, который с самого начала
безопасный по-умолчанию, и затем добавлять небезопасные участки по мере их необходимости.
Гораздо труднее писать безопасный код, основываясь на том, что изначально небезопасно.

Хотя Rust и дает возможность выбора, он подталкивает вас не стрелять себе в ногу.

## Так что там с ногами?

Возвращаясь к ногам, пропускал ли Rust свои тренировки? Получился ли он однобоким? Оказался ли он
сосредоточен на неправильных вещах?

Rust крепнет с каждым днем, и, к счастью, хорошо осведомлен, как выполнять присед, не прогибая при
этом спину. Этот момент трудно переоценить. Философия Rust имеет прочный фундамент, а значит язык
будет расти и развиваться.
