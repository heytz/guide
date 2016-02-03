---
title: 集合与架构
order: 1
description: 在流星中, 如何定义, 使用, 和维护MongoDB集合
---

通过阅读本指南，你可以了解到：

1. 在流星中不同类型的MongoDB集合，以及，如何使用他们。
2. 如何为某个集合定义架构以控制他的内容。
3. 当定义你的集合架构时，应该考虑什么。
4. 当对一个集合进行写入时，如何强制写入数据符合其架构。
5. 如何小心地改变你的集合架构。
6. 如何应对数据记录的关联性。

<h2 id="mongo-collections">流星中的MongoDB集合</h2>

一个web应用的核心而言，就是为他的用户提供了视图，以及修改，持久化一组数据的方式。无论是管理todos的列表，或者是为你挑选的车下订单，你都是在和一个固定的，但又是持续变化的数据层在打交道。

在流星中，这个数据层通常存储在MongoDB中。在MongoDB中一组相关联的数据是指，作为某个 “集合”。在流星中，你是通过[集合](http://docs.meteor.com/#/full/mongo_collection)来存取MongoDB的，这可以作为你的应用的主要持久化机制。

然而，集合有很多方法来保存和获取数据。他们也提供核心的交互，连接用户体验，就是用户期待从最好的应用那里获得的那种。流星可以让这种用户体验很容易就得到实现。

在本文中，我们将紧密观察集合在该框架的不同位置上是如何工作的，并且，如何获得其中的大多数（实现的经验？）。

<h3 id="server-collections">服务器端集合</h3>

当你在服务器端创建了某个集合：

```js
Todos = new Mongo.Collection('Todos');
```

你同时也在MongoDB中创建了一个集合、一个对于该集合如何在服务器端被使用的接口。这是一个相当明了的、构建在Node MongoDB驱动之上的层，只不过是同步API：

```js
// 这行之道插入结束才会完成
Todos.insert({_id: 'my-todo'});
// 所以这行会返回一些什么
const todo = Todos.findOne({_id: 'my-todo'});
// 看，没有回调!
console.log(todo);
```

<h3 id="client-collections">客户端的集合</h3>

在客户端，当你写上相同的这一行时：

```js
Todos = new Mongo.Collection('Todos');
```

它就做了完全不同的事情!

在客户端，并没有直接连接MongoDB数据库，事实上，对于它的同步API也是不可能的（或者说，可能不是你想要的）。因此，在客户端，某个集合是数据库在某个客户端的 *缓存*。达成这样的效果要感谢 [Minimongo](https://www.meteor.com/mini-databases) 库 －－－ 一种内存型，全JS的MongoDB API的实现。这意味着在客户端，当你写入时：

```js
// 这行改变了内存中Minimongo数据结构
Todos.insert({_id: 'my-todo'});
// 这行是进行查询
const todo = Todos.findOne({_id: 'my-todo'});
// 这里就立刻发生了！
console.log(todo);
```

你从服务器端集合（MongoDB支持）移动数据到客户端集合（在内存中）的方式是 [数据装载文章](data-loading.html) 的主题。通常而言，你从某个 *发布* 来 *订阅*，将会从服务器端推送数据到客户端。一般，你能够假设客户端具有服务器端MongoDB集合全部副本的，最新数据的某些子集。

要将数据写回服务器，你可以使用某个 *Method*，这是[方法文章](methods.html)的主题。

<h3 id="local-collections">本地集合</h3>

在流星中，还有第三种方法用来使用集合。在客户端或者是服务器端，如果你创建某个集合时传递 `null` 而不是一个名字：

```js
SelectedTodos = new Mongo.Collection(null);
```

这就会创建一个 *本地集合*。这是一个Minimongo集合，不具有数据库连接（通常，有名字的集合要不就是直接连接到服务器端的数据库（服务器端），或者是通过订阅连接到服务器的数据库（客户端））。

一个本地集合是为了在内存中存储（数据）以方便地使用Minimongo库的全部威力。举个例子，如果你需要对于你的数据进行复杂查询时，可以考虑使用它来替代一个简单的数据。活着你想要在客户端利用它的 *响应性* 来驱动某些用户界面，在流星中，这会让人感觉更自然。 

<h2 id="schemas">定义架构</h2>

虽然MongoDB是一个无架构数据库，这样允许了最大限度的数据结构的自由，但是，使用架构来约束集合数据内容以符合某个已知的格式，通常是较好的实践。如果你不这么做，那么，你需要去写防御式的代码，来检查和确认你的数据结构是否和其从 数据库 *取出来的* 一致，以替代在 *进入* 数据库时那么（检查和确认）。 大多数时候，你试图 *读取数据多于写入数据*，所以，通常（使用架构）显得更容易，并且当写入时产生更少的缺陷。

在流星中，名声显赫的架构包是 [aldeed:simple-schema](http://atmospherejs.com/aldeed/simple-schema)。这是一个生动的，基于MongoDB架构用来插入和更新文档。

要使用 `simple-schema` 来定义架构，只需要简单地创建一个新的、 `SimpleSchema` 类的实例：

```js
Lists.schema = new SimpleSchema({
  name: {type: String},
  incompleteCount: {type: Number, defaultValue: 0},
  userId: {type: String, regEx: SimpleSchema.RegEx.Id, optional: true}
});
```

本例是从Todos应用中而来，定义了一个具有一些简单规则的架构：

2. 我们指定某个列表的 `name` 字段是必须的，而且必须是个字符串。
3. 我们指定 `incompleteCount` 是一个数字，而且在插入时，如果没有指定别的值的时候，插入 `0` 。
4. 我们指定 `userId` 是可选的，必须是字符串且符合用户文档的ID格式。

我们可以直接附加该架构到 `Lists` 命名空间，这可以使我们无论何时想要检测对象的架构成为可能，例如在某个表单或者 [方法](methods.html)中。在 [下一章节](#schemas-on-write) 中我们将会看到在写入到集合时，我们将会看到如何自动地使用架构。

你可以看到通过相关很少的代码，我们已经可以极大的限制某个列表的格式。你可以在 [Simple Schema 文档](http://atmospherejs.com/aldeed/simple-schema) 中阅读更多相关复杂的、通过架构进行的实现。

<h3 id="validating-schemas">通过架构进行验证</h3>

现在我们有了一个架构，接着怎么使用它呢？

要通过某个架构验证某个文档是很清楚的，我们可以这么写：

```js
const list = {
  name: 'My list',
  incompleteCount: 3
};

Lists.schema.validate(list);
```

本例中，由于列表是通过架构验证的，`validate()` 这行运行起来没有任何问题。但如果我们这么写：

```js
const list = {
  name: 'My list',
  incompleteCount: 3,
  madeUpField: 'this should not be here'
};

Lists.schema.validate(list);
```

那么 `validate()` 的调用会抛出一个 `ValidationError` ，其中包含关于 `list` 文档出错的详细信息。

<h3 id="validation-error"> `ValidationError`</h3>

什么是[`ValidationError`](https://github.com/meteor/validation-error/)？这是一个特殊的错误被用来在流星中指代在修改集合时，基于错误的某个用户输入。典型地，某个 `ValidationError` 细节被用来标记出某个表单，具有关于输入不匹配架构的信息。在 [方法一文](methods.html#validation-error)中,我们将会看到更多关于它是如何工作的。 

<h2 id="schema-design">设计你的数据架构</h2>

现在你已经熟悉了Simple Schema的基本API，那么值得考虑一些关于流星数据系统的约束，这会影响到你的数据架构的设计。虽然通常而言，你能够构建一个流星的数据架构和任何的MongoDB数据架构非常相似，请记住一些重要的细节。

最重要的考量是关于DDP，流星的数据装载协议，通过互联与文档进行交互。关键是能够意识到DDP发送到文档改变位于文档顶层的 *字段*。这就是说如果你在文档上有巨大而且复杂的子字段，并且经常改变，DDP会通过互联发送不必要的变化。

例如，在 “纯” MongoDB中你也许会设计这样的架构，以至于每个列表文档具有一个叫做  `todos` 的字段，其中存储着todo项的一个数组。

```js
Lists.schema = new SimpleSchema({
  name: {type: String},
  todos: {type: [Object]}
});
```

这个架构的问题是由于刚提到的DDP行为，每次对于某个列表中 *任意* 的todo项发生变化，就会通过网络，发送该列表中 *整个* 一组todos。这是因为DDP对于在被称为 `todos` 第三项中的 `text` 字段的 “变化“ 没有任何的概念，因此它简单地 对于 `todos` 字段进行一个完整的数组的改变。

<h3 id="denormalization">冗余化和多集合</h3>

标题暗指我们需要创建多集合来包含子文档。在Todos应用示例中，我们需要一个 `Lists` 集合与一个 `Todos` 集合用来包含每个列表的todo项。因此我们需要去做一些典型的类似SQL数据库事情，就好像使用外键（`todo.listId`）来关联某个文档一样。

In Meteor, it's often less of a problem doing this than it would be in a typical MongoDB application, as it's easy to publish overlapping sets of documents (we might need one set of users to render one screen of our app, and an intersecting set for another), which may stay on the client as we move around the application. So in that scenario there is an advantage to separating the subdocuments from the parent.在流星中，这么做会比典型的MongoDB应用会少些问题，发布部分重叠的几组文档（我们也许需要一组用户在我们应用中渲染一个屏幕，并且对于另一组进行拦截）是很容易的，这些数据可能留在客户端，作为我们驱动该应用的数据。所以，在这场景里将子文档从父文档中分离是有利的。//todo: not confortable with this

然而，对于给定MongoDB先于3.2的版本，并不支持通过多集合的查询（join），相应地我们需要冗余化数据到父级集合中去。冗余化是指在数据库中存储多次相同的信息的实践（作为与“正常”非冗余的形式的反例）。MongoDB是一种鼓励冗余的数据库，并且为此进行过优化。

在Todos应用的例子中，因为我们想要在每个列表的旁边显示没有完成的todos，我们需要冗余化 `list.incompleteTodoCount`。这不方便但却是典型地、有理由地容易的做法，我们可以在以下章节 [抽象冗余器](#abstracting-denormalizers) 进行查看。

另一个冗余化的做法，其架构有时需要能够从父级文档做到子文档。例如，在Todos，由于我们强制了todo列表的隐私，这是通过 `list.userId` 标签来玩成的，但是，我们分开发布todos，所以冗余 `todo.userId` 是有道理的。要完成这个，当创建todo的时候，我们需要从列表中获得 `userId` ，并且无论何时某个列表的 `userId` 发生变化时，更新所有相关的todos。

<h3 id="designing-for-future">为将来做设计</h3>

一个应用，尤其是一个web应用，很少是完全完成的，当设计你的数据架构时，考虑潜在的将来的变化是很有用的。因此，在多数情况下，在你确实需要这些字段（毕竟，经常发生的状况是你所预期的并没有实际发生）之前就添加它们，显然不是个好主意。

但是，提前考虑架构会如何随着时间而变化却是个好主意。例如，你可能在某个文档上有一组字符串字段（也许是一组tag）。虽然可以尝试让它们作为文档上的字段（假设它们并不会有太多的变化）是可以接受的，将来，它们一有机会就会变得相当的复杂（也许tag会有个创建者，或者子tag什么的），所以，一开始就将它们分到单独的集合中要比在运行很长的一段时间后再改要容易些。

这些前瞻性融入到你的架构设计将会依赖于你的应用的个性化约束，并且会需要你自己决定。

<h3 id="schemas-on-write">写入时使用架构</h3>

虽然有不同的方法可以让你通过某个Simple Schema运行数据，在发送到你的集合之前（例如你可以在每个方法调用中检测架构），但是最简单和最可靠的方法就是使用 [`aldeed:collection2`](https://atmospherejs.com/aldeed/collection2) 包，通过该架构来运行每个变化者（`insert/update/upsert` 的调用）。

要这么做，我们使用 `attachSchema()`：

```js
Lists.attachSchema(Lists.schema);
```

What this means is that now every time we call `Lists.insert()`, `Lists.update()`, `Lists.upsert()`, first our document or modifier will be automatically checked against the schema (in subtly different ways depending on the exact mutator).这意味着从现在开始，每次我们调用 `Lists.insert()`, `Lists.update()`, `Lists.upsert()`, 我们的文档或修饰器会自动根据该架构（这种不易察觉的不同方式依赖于实际的那个变形器）进行检测。

<h3 id="default-value">`defaultValue` 以及数据清理</h3>

Collection2所做的一件事情是 [“清理” 该数据](https://github.com/aldeed/meteor-simple-schema#cleaning-data) ，在数据发送到数据库之前。这包括但不限于：

1. 修正类型 － 转换字符串到数字
2. 移除不在该架构中的属性
3. 根据架构定义中的 `defaultValue` 来分配默认值

有时很有用的是，在将数据插入集合之前，做一些复杂的对于文档的初始化。例如，在Todos应用中，我们想要设置新的列表的名字为 `List X` ，其中 `X` 是下一个允许的那个字母。

要达到这样的效果，我们可以子类化 `Mongo.Collection` 并且写我们自己的 `insert()` 方法：

```js
class ListsCollection extends Mongo.Collection {
  insert(list, callback) {
    if (!list.name) {
      let nextLetter = 'A';
      list.name = `List ${nextLetter}`;

      while (!!this.findOne({name: list.name})) {
        // not going to be too smart here, can go past Z
        nextLetter = String.fromCharCode(nextLetter.charCodeAt(0) + 1);
        list.name = `List ${nextLetter}`;
      }
    }

    // Call the original `insert` method, which will validate
    // against the schema
    return super(list, callback);
  }
}

Lists = new ListsCollection('Lists');
```

<h3 id="hooks">在 insert/update/remove 进行挂接</h3>

The technique above can also be used to provide a location to "hook" extra functionality into the collection. For instance, when removing a list, we *always* want to remove all of its todos at the same time.上述的技术也可以被用来在集合中提供一个位置 “挂接” 额外的功能。例如，当移除某个列表时，我们 *一直* 想要同时移除它的所有的todos。

我们也可以在这种情况下使用子类，覆写 `remove()` 方法：

```js
class ListsCollection extends Mongo.Collection {
  // ...
  remove(selector, callback) {
    Package.todos.Todos.remove({listId: selector});
    return super(selector, callback);
  }
}
```

该技术有些不好之处：

1. 变型器会变得非常长，当你想要挂接多次时。
2. 有时，单一功能的片段会被传递到多个变形器上。
3. 用完整地通用方式（穷尽每个可能的选择器和修改器）来完成挂接是很具有挑战性，并且这么做对于你的应用也不是完全必要（因为也许你只是想要在每次调用变形器时都是用同样的方式）。

A way to deal with points 1. and 2. is to separate out the set of hooks into their own module, and simply use the mutator as a point to call out to that module in a sensible way. We'll see an example of that [below](#abstracting-denormalizers).有方法可以应对1和2，就是把一组挂接分离到它们自己的模块中，而且简单地使用变形器作为一个点来调用出那个模块，这是一种明智的方式。[下面](#abstracting-denormalizers)我们会看到一个例子。

Point 3. can usually be resolved by placing the hook in the *Method* that calls the mutator, rather than the hook itself. Although this is an imperfect compromise (as we need to be careful if we ever add another Method that calls that mutator in the future), it is better than writing a bunch of code that is never actually called (which is guaranteed to not work!), or giving the impression that your hook is more general that it actually is.第3点通常可以通过放置在调用该变形器中的方法中放置挂接，而不是放置在挂接的自身，获得解决。虽然这是一种有缺陷的妥协（因为我们会要很小心，如果我们将来要添加另一个方法来调用该变形器），但也好过写一堆从未被实际调用的代码（保证不会工作！）要来得强，或者说，给予一种你的挂接是更通用（它实际上也是）的印象。

<h3 id="abstracting-denormalizers">抽象化冗余器</h3>

冗余可能会发生在不同的几个集合的变形器上。因此，在一个地方定义该冗余，并用一行代码将它挂接到每个变形器中，是明智的选择。这种方法的好处是冗余逻辑集中在一个地方，而不是散落在很多多文件中，但你仍然能够对于每个集合进行代码检查以完全理解每个更新究竟发生了什么。

在Todos示例应用中，我们建立了一个 `incompleteCountDenormalizer` 来抽象列表中的未完成的todos的数量。这段代码需要在每个todo项被插入，更新（勾选或去除勾选），以及移除时进行运行。这段代码如下：

```js
const incompleteCountDenormalizer = {
  _updateList(listId) {
    // 从MongoDB中直接重新计算正确的未完成的数量
    const incompleteCount = Todos.find({
      listId,
      checked: false
    }).count();

    Lists.update(listId, {$set: {incompleteCount}});
  },
  afterInsertTodo(todo) {
    this._updateList(todo.listId);
  },
  afterUpdateTodo(selector, modifier) {
    // 我们在todos上仅支持非常有限的操作
    check(modifier, {$set: Object});

    // We can only deal with $set modifiers, but that's all we do in this app
    if (_.has(modifier.$set, 'checked')) {
      Todos.find(selector, {fields: {listId: 1}}).forEach(todo => {
        this._updateList(todo.listId);
      });
    }
  },
  // Here we need to take the list of todos being removed, selected *before* the update
  // because otherwise we can't figure out the relevant list id(s) (if the todo has been deleted)
  afterRemoveTodos(todos) {
    todos.forEach(todo => this._updateList(todo.listId));
  }
};
```

我们可以将冗余器连接到 `Todos` 集合的变形中，如下所示：

```js
class TodosCollection extends Mongo.Collection {
  insert(doc, callback) {
    doc.createdAt = doc.createdAt || new Date();
    const result = super(doc, callback);
    incompleteCountDenormalizer.afterInsertTodo(doc);
    return result;
  }
}
```

注意我们只处理了那些我们在应用中实际用到的变形器－－－我们不需要应对某个列表上todo数量的所有可能的变化。例如，如果你改变了某个todo项的 `listId`，这会需要改变 *两个* 列表的 `incompleteCount`。既然我们的应用不做这些，我们就不需要在冗余器中处理它。

Dealing with every possible MongoDB operator is difficult to get right, as MongoDB has a rich modifier language. Instead we focus on just dealing with the modifiers we know we'll see in our app. If this gets too tricky, then moving the hooks for the logic into the Methods that actually make the relevant modifications could be sensible (although you need to be diligent to ensure you do it in *all* the relevant places, both now and as the app changes in the future).应对每个可能的MongoDB操作符很困难的，由于MongoDB是具有相当多的修改符的语言。与其专注于应对修改符，不如我们在应用中进行查看。如果这变得太麻烦，那么刻意移动对于这段逻辑的挂接到一些方法中，而这些方法是实际上产生相关修改，这样做很明智（虽然你需要很努力地确保在“所有”相关的位置都这么做，现在以及将来对于该应用的变化）。

能完整地抽象一些通用的冗余技术并实际地尝试去应对所有的可能的改变，对于这样的包的存在是很有意义的。如果你写了这样一个包，请让我们知道！

<h2 id="migrations">迁移到一个新的架构</h2>

就如上面我们所讨论的那样，试图预测对于你的数据架构的所有未来需求是不现实的。相反地，作为一个成熟的项目，有时需要去改变数据库的架构。你需要很小心，知道如何迁移到新的架构，以保证你的应用平滑地工作，在迁移期间以及在迁移之后。

<h3 id="writing-migrations">写迁移</h3>

能用来写迁移的一个有用的包是[`percolate:migrations`](https://atmospherejs.com/percolate/migrations)，它可以提供一种很好的框架用来在你的不同的架构之间进行切换。

假设，作为一个例子，我们想要添加一个 `list.todoCount` 字段，并且保证对于所有已经存在的列表中的该字段都得到设置。那么我们也许只需要在服务器端写下列代码(e.g. `/server/migrations.js`):

```js
Migrations.add({
  version: 1,
  up() {
    Lists.find({todoCount: {$exists: false}}).forEach(list => {
      const todoCount = Todos.find({listId: list._id})).count();
      Lists.update(list._id, {$set: {todoCount}});
    });
  },
  down() {
    Lists.update({}, {$unset: {todoCount: true}});
  }
});
```

这个迁移，它是序列为第一个迁移以便于在数据库中进行运行，将会，当被调用时，为每个列表更新现有todo数量。

To find out more about the API of the Migrations package, refer to [its documentation](https://atmospherejs.com/percolate/migrations).要发现关于迁移包API的更多信息，参见 [它的文档](https://atmospherejs.com/percolate/migrations)

<h3 id="bulk-data-changes">大量改变</h3>

如果你的迁移需要改变很多数据，并且，尤其如果你需要停止你的应用服务器，当其还在运行中，可能使用[MongoDB 大量操作](https://docs.mongodb.org/v3.0/core/bulk-write-operations/) 是一个好主意。

一个大量操作的优势在于，它不仅只需要一个round trip到MongoDB的数据写入，通常以为着这会很快。负面的影响是如果你的迁移很复杂（通常也是这样的，如果你不能仅仅使用一个`.update(.., .., {multi: true})`来完成），它会需要花费大量的时间来准备大量更新。

这意味着如果用户正在进入该站点，而一边该更正在准备中，它就会完全不知道跑到哪天去！而且，一个巨量更新会在当其被应用时锁住整个集合。由于这些原因，你经常需要停止你的服务器并且让你的用户知道你正在执行维护，当该更新运行的时候。

我们可以这样写上述迁移（注意你必须在MongoDB 2.6或以上版本执行巨量更新操作）。我们能够通过 [`Collection#rawCollection()`](http://docs.meteor.com/#/full/Mongo-Collection-rawCollection) 使用原生的MongoDB API。

```js
Migrations.add({
  version: 1,
  up() {
    // This is how to get access to the raw MongoDB node collection that the Meteor server collection wraps
    const batch = Lists._collection.rawCollection().initializeUnorderedBulkOp();
    Lists.find({todoCount: {$exists: false}}).forEach(list => {
      const todoCount = Todos.find({listId: list._id}).count();
      // We have to use pure MongoDB syntax here, thus the `{_id: X}`
      batch.find({_id: list._id}).updateOne({$set: {todoCount}});
    });

    // We need to wrap the async function to get a synchronous API that migrations expects
    const execute = Meteor.wrapAsync(batch.execute, batch);
    return execute();
  },
  down() {
    Lists.update({}, {$unset: {todoCount: true}});
  }
});
```

注意我们能够通过使用一种[聚合](https://docs.mongodb.org/v2.6/aggregation/) 来聚集todo数量的初始集合，来使得该迁移变的快速。

<h3 id="running-migrations">运行迁移</h3>

要在你的开发数据库上运行某个迁移，使用流星shell是很容易的：

```js
// After running `meteor shell` on the command line:
Migrations.migrateTo('latest');
```

如果该迁移的日志打印到console上，你将会在流星服务器的终端窗口上看到它。

要在你的生产环境数据库上运行一个迁移，首先在本地运行你的应用，并使用生产环境模式（通过生产环境设定以及环境变量，包括数据库设定），然后同样适用流星的shell。这就是运行 `up` 函数在所有激活的迁移，在你的生产环境数据库上。在我们的示例中，它可以保证所有的列表具有一个 `todoCount` 字段集。

一个好的方式去做上述的事情就是，启动一个和你的数据库接近的虚拟机，而且在上面安装好流星和SSH（一个特殊的EC2实例，你可以启动和停止是，为此目的设立的，是一个可以接受的选择），并且在shell进去之后运行命令。这种方式可以消除任何你的机器和数据库之间的延时，但你仍然需要非常小心的知道迁移是如何运行的。

**注意你应该一直保持在运行任意迁移之前进行数据库备份**

<h3 id="breaking-changes">断裂性的架构改变</h3>

有时当我们改变了某个应用的架构，我们就处于一种断裂的状态 －－ 以至于旧的架构不能够和新的代码很好地工作在一起。例如，如果我们的一些用户界面代码已经严重地依赖于所有的列表具有一个 `todoCount` 集，那么会有一段时间，在迁移运行之前，在部署之后，我们应用的界面处于断裂的状态中。

一种使该问题能够得到缓解的简单方式是，将该应用在部署和完成迁移之间进行下线。这非常的不理想，尤其考虑到某些迁移会要数个小时进行运行（即使使用[巨量更新](#bulk-data-changes) 带来了很多帮助）。

一个更好的方式是一个多阶段的部署。基本的想法是：

1. 部署一个你的应用的版本，其能够同时处理旧的和新的架构。在我们的例子中，就是代码不会要 `todoCount` （对于已经存在的列表），但可以正确的处理那些新创见的todos。
2. 运行迁移。这个时候，你需要足够确信所有的列表都有了一个 `todoCount`
3. 部署新的代码依赖于新的架构，并且，不在知道如何处理老的架构。现在我们可以很安全的依赖于 `list.todoCount` ，在我们的用户界面里。

应该知晓另一件事情，尤其地，通过这么一种多阶段的部署，准备回滚是非常重要的！为了这个原因，该迁移包允许你能够指定某个 `down()` 函数并且在调用 `Migrations.migrateTo(x)` 来迁移 _回_ 到版本  `x`。

所以如果我们想要回滚我们上面的迁移，我们可以运行
```js
// The "0" migration is the unmigrated (before the first migration) state
Migrations.migrateTo(0);
```

如果你发现你需要回滚你的代码版本，你将会需要小心应对你的数据，并且很小心的反向你的部署步骤。

<h3 id="migration-caveats">警告</h3>

一些上面列出来的迁移策略某些方面或许并不是最理想的方式（虽然在大多数的情况下是也许是适用的）。这里有一些其它需要知道的：

1. 通常最好不要依赖于你的应用的代码在迁移中（因为应用会随着时间改变，但迁移不应该）。例如，让你的迁移通过的Collection2集合进行传递（那么就可以检查架构，设置自动值等等）可能随着时间的变化断裂它们，因为你的架构是随着时间变化的。

  为了避免这样的问题可以简单地不在你的数据上运行旧的迁移。这有些限制但却能工作。

2. 在你的本地机器上运行迁移可能使得它需要花费更多的时间，因为你的机器不是和生产环境数据库相同的配置

Deploying a special "migration application" to the same hardware as your real application is probably the best way to solve the above issues. It'd be amazing if such an application kept track of which migrations ran when, with logs and provided a UI to examine and run them. Perhaps a boilerplate application to do so could be built (if you do so, please let us know and we'll link to it here!).部署一个特殊的 “迁移应用" 到某个和你真实应用一样硬件配置的机器可能是解决上面的问题的最好的方式。

<h2 id="associations">集合之间的关联</h2>

如我们早些时候讨论的，在流星应用中，不同的集合中的文档之间具有关联是很常见的。因此，需要撰写获取关联的文档的查询也是很常见的，一旦你需要某个文档是你想要的（例如获得一个列表中所有的todos）。

为了让这些变得简单，我们能够为某个给定集合的文档的原型链上附加函数，让我们可以在这些文档上具有 “方法” （从面向对象的角度考虑）。我们能够使用这些方法来创建新的查询用来找到关联的文档。

<h3 id="collection-helpers">集合帮助器</h3>

我们可以使用 [`dburles:collection-helpers`](https://atmospherejs.com/dburles/collection-helpers) 包，就可以很简单地附加这样的方法（或者 “帮助器”）到文档。例如：

```js
Lists.helpers({
  // A list is considered to be private if it has a userId set
  isPrivate() {
    return !!this.userId;
  }
});
```

一旦我们附加了这个帮助器到 `Lists` 集合，每次我们从数据库中获取一个列表（在客户端或在服务器端），它都会有一个 `.isPrivate()` 函数可用：

```js
const list = Lists.findOne();
if (list.isPrivate()) {
  console.log('The first list is private!');
}
```

<h3 id="association-helpers">Association helpers</h3>

现在我们可以附加帮助器到文档，那么定义一个帮助器来获取相关的文档就变得容易了。

```js
Lists.helpers({
  todos() {
    return Todos.find({listId: this._id}, {sort: {createdAt: -1}});
  }
});
```

现在我们可以很容易地找到某个列表的所有todos了：

```js
const list = Lists.findOne();
console.log(`The first list has ${list.todos().count()} todos`);
```
