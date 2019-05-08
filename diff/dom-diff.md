# dom diff
## DOM DIFF算法简析
###  diff策略
 - dom节点跨层级的操作特别少，所以可以忽略不计
 - 拥有相同累的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形
 - 同一层级的一组子节点，他们可以通过uuid进行区分
 ### diff 粒度
   #### 1、Tree Diff 
 - 对树的 每一层进行遍历，如果组件不存在了则会直接销毁
   #### 2、 Component Diff 
    React是基于组件构建应用的，对于组件间的比较所采用的策略也是非常简洁和高效的。
    -  如果是同一个类型的组件，则按照原策略进行Virtual DOM比较。
    - 如果不是同一类型的组件，则将其判断为dirty component，从而替换整个组价下的所有子节点。

    - 如果是同一个类型的组件，有可能经过一轮Virtual DOM比较下来，并没有发生变化。如果我们能够提前确切知道这一点，那么就可以省下大量的diff运算、时间。因此，React允许用户通过shouldComponentUpdate()来判断该组件是否需要进行diff算法分析。

   #### 3、Element Diff 
      当节点属于同一层级时，diff提供了3种节点操作，分别为INSERT_MARKUP(插入)，MOVE_EXISTING(移动),REMOVE_NODE(删除)。

        - INSERT_MARKUP:新的组件类型不在旧集合中，即全新的节点，需要对新节点进行插入操作。
        - MOVE_EXISTING:旧集合中有新组件类型，且element是可更新的类型，这时候就需要做移动操作，可以复用以前的DOM节点。
        - REMOVE_NODE:旧组件类型，在新集合里也有，但对应的element不同则不能直接复用和更新，需要执行删除操作，或者旧组件不在新集合里的，也需要执行删除操作。

    - 建议
        -  uuid不要设置为数组的index，因为会变动，即使元素并未发生变化
        
      eg:
      ```js
    
       var element = {
        tagName: 'A',
        props: {
          id: 'A'
        },
        children: {
          {
            tagName: 'B',
            props: {
              class: 'B'
            },
            children: ['D','E']
          }, 
          {
            tagName: 'C',
            props: {
              class: 'C'
            },
            children: ['F','G']
          }
        }
      }
      ```
      diff算法是react中经典之作，他很巧妙，该算法是react整个界面渲染的基础和保障。


将之前先介绍一下传统的DIff的弊端：传统的Diff算法通过循环递归依次对节点进行对比，效率比较低下，算法的时间复杂度为O(n^3)，n为树的节点数，当节点比较多的时候，这种搜索次数将会急剧增加，计算机的负荷开销将会十分巨大。



### DIFF的三种策略：DOM间的diff、组件间的diff、元素间的diff

#### 1. tree diff（这种情况比较少）

react对树的算法进行了简单的优化：对树进行分层的比较，两棵树只会对同层的节点进行比较。

react通过updateDepth对Virtual DOM树进行层级的控制，只会对相同层级的DOM节点进行比较，即同一个父节点下的所有的子节点。当发现节点已经不存在的时候，则该节点即子节点会被完全的删除，不会进行进一步的比较了，这样只需要对树遍历一次即可。

（注意：开发组件的时候，保持组件的DOM结构的稳定有助于提高性能，例如：可以通过CSS来隐藏节点，而不是真正的删除或者增加DOM节点）

##### 2.component diff

组件间的比较算法：

·如果是相同类型的组件，按照tree diff的策略进行比较；

·如果不是，将该组件判断为dirty component，然后将该节点下面的所有的子节点全部替换掉；

·对于相同类型的组件，可能他的Virtual DOM没有任何的变化，如果知道这一点，将极大的节省计算diff的时间，因此，react允许用户通过shouldComponentUpdate()来判断该组件是否需要diff算法。



以上两个都是比较简单的情况：

##### 3.element diff

当节点在同一层级的时候，diff提供三种操作：INSERT_MAPKUP（插入）、MOVE_EXISTING（移动）、REMOVE_NODE（删除）。

此处引进key值：这是由于，当层级中的节点元素没有发生变化的时候，而是仅仅是位置的变化，如果按照上述的原则位置不同将会重新绘画，这将影响渲染的性能，因此，为每个节点添加唯一的key值，当发现有相同的节点的时候，不需要进行重新的创建或删除节点的操作。



如图所示：diff的操作的过程（就是旧节点更新为新节点的过程）

首先对新节点逐一进行遍历，通过唯一的key值来判断旧的集合中是否存在相同的节点，图中，先遍历新图中的节点，首先取到节点B，发现B节点在旧图中存在（B在旧图中的index=1），此时比较lastIndex（访问过的节点的最右的位置，初始值为0）与B在旧图中的index比较，if(index<lastIndex)，则进行移动操作，此时，index=1>lastIndex=0，因此B节点不移动，接下来更新lastIndex的值：lastIndex= max(lastIndex,index),此时lastIndex值更新为1；

接着取新节点A，A在旧节点中的位置index=0，此时，lastIndex= 1，满足（index<lastIndex）的条件，因此，将A的值更新为新节点中的位置，此时lastIndex仍然为1；

接下来取到新节点的D，D在旧节点中的位置为index=3，此时lastIndex=1，不满足（index<lastIndex）条件，因此，D的位置不更新，此时lastIndex = max(lastIndex,index) = 3;

最后取到节点C，C在旧节点的位置为index= 2，此时lastIndex=3，满足（index<lastIndex）条件，所以对，C节点进行移动操作。



上述仅仅是，一种情况，如果出现遍历一遍新节点过程中，在旧节点中不存在该节点，应该添加该节点，最后遍历完新节点后，还要遍历一遍 旧节点，将旧节点中多余的节点删除掉

eact是facebook开发的用来构造UI界面的JS库。它被设计的时候就从底层去考虑解决性能问题。这篇文章里我将阐述react的diff算法和渲染机制，以此来帮助读者优化自己的应用。

#### diff算法
在我们深入到实现细节之前，我们很有必要先看一下React是怎样工作的。
```
var MyComponent = React.createClass({ 
    render: function() { 
        if (this.props.first) { 
            return <div className="first"><span>A Span</span></div>; 
        } else { 
            return <div className="second"><p>A Paragraph</p></div>; 
        } 
    }
 });
```
在任何时候，你都是这样去描述你想获得的UI界面。理解render方法的结果并不是实际的DOM这一点很重要。那些结果只是一些轻量的JavaScript对象，我们可以把它们称为虚拟DOM。

##### React想利用这样的表示方法来寻找上一次渲染到下一次渲染之间能够执行的最少步骤。例如，如果你想先挂载<MyComponent first={true} />，然后用<MyComponent first={false} />来替换掉它，最后把这个组件卸载，操作DOM的指令可能是这样的：

从无到有：

创建节点：```<div className="first"><span>A Span</span></div>```
从一到二：

将节点属性className="first"替换成className="second"
将子节点 ```<span>A Span</span>替换成<p>A Paragraph</p>```
从二到无：

移除节点：```<div className="second"><p>A Paragraph</p></div>```
#### 层级对比
计算一棵树形结构转换成另一棵树形结构的最少操作是一个O(n^3)问题。可以想象，传统解法对我们的实际用例并不友好。React使用了一种简单却强大的技巧，使算法的复杂度接近O(n)。
![](https://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/1.png)
React只会比较两棵树之间的同级节点。这样就彻底的降低了复杂度，并且不会带来什么损失。因为在web应用中不太可能把一个组件在DOM树中跨层级地去移动。它们通常只会在子节点中平级的移动组件，如下图： 
level by level

### 列表
假设我们有一个组件，需要循环渲染5个相同的组件，然后在这5个组件组成的列表的中间位置插入一个新的组件。根据这些仅有的信息，我们很难去在这两个新旧列表之间做好映射关系。

默认的，React会把前一个列表的第一个组件跟下一个列表的第一个组件做对比，以此类推。你可以在组件中设置key属性，来帮助React更好的做出映射比对。实际上，通常在子节点中找到一个唯一的key是非常容易的。 
list and key

### 组件
一个React应用通常是由多个用户自定义组件组合而成，最终会转换成一个主要有div节点构成的树。React的diff算法处理这些额外的信息时，它只会去比较那些拥有相同类名的组件。

例如，一个<Header>组件被<ExampleBlock>组件替换时，React会直接移除header组件，然后创建一个example block组件。我们不需要去浪费宝贵的时间来比较两个完全没有相似点的组件。 
components

### 事件代理机制
在DOM节点上挂载事件监听器，响应又慢又吃内存。与此相反，React实现了一种非常流行的叫“事件代理”的技术。React甚至在未来打算重新实现一个兼容W3C标准的事件系统。这意味着IE8的事件处理bug成为了过去时，并且在所有的浏览器中事件名可以得到统一。

让我们来解释一下这是怎么实现的。它会在document的根节点上注册一个事件监听器。当一个事件被触发，浏览器会告诉我们目标DOM节点。为了能够通过DOM层级来传播事件，React不会再虚拟dom上迭代层级。

取而代之的是，我们利用了每一个React组件都会使用唯一的id来编码层级这一事实。我们可以通过简单的字符串操作来获取所有父级的id。通过把注册地事件监听器放在一个hashMap中，我们发现这样做的性能远比把它们关联到虚拟DOM要好。下面的例子展示了事件如何在虚拟DOM上进行分发：

```// dispatchEvent('click', 'a.b.c', event)
clickCaptureListeners['a'](event);
clickCaptureListeners['a.b'](event);
clickCaptureListeners['a.b.c'](event);
clickBubbleListeners['a.b.c'](event);
clickBubbleListeners['a.b'](event);
clickBubbleListeners['a'](event);
```

浏览器会为每一个事件和事件监听器创建一个事件对象。这个事件对象有一个很不错的属性就是你可以维护每一个事件对象的引用，甚至修改它们。但这也意味这很高的内存开销。React会在应用启动的时候为这些对象分配一个内存池。任何需要用到事件对象的时候，都可以从这个内存池获得一个可复用的对象。这样可以显著的减轻垃圾回收的负担。

### 渲染
#### 批量处理
任何时候你在一个组件中调用setState，React都会将这个组件标记为dirty。在一次事件循环结束后，React会搜索所有被标记为dirty的组件，并对它们进行重新渲染。

这一批量处理意味着在一次事件循环中，DOM只会被更新一次。这个特性是打造高性能应用的关键，通常在编写JavaScript代码时难以实现。然而在React应用中，这一特性是默认实现的。

batching

#### 子树渲染
当setState被调用时，组件会为了更新子节点而重新构建虚拟DOM。如果你在根元素上执行setState，则整个React应用都会被重新渲染，所有组件的render方法都会被调用，即使它们没有发生任何改变。这听起来既吓人又低效，但实际上还好，因为我们并没有去改变真实的DOM。

首先，我们来关注UI界面的展示。根据屏幕大小的限制，通常是需要顺序渲染数千到上万不等的元素。对于可管理的整个界面，JavaScript对于业务逻辑的处理已经足够快了。

另一个很重要的点在于，编写React代码时，你通常不需要每次都在根节点上执行setState来改变视图。你可以在接受变更事件的一个或几个组件上来执行setState。你很少需要一直在根节点上调用setState，这就意味着可以把变化限制在与用户发生交互的组件上。

subtree

选择性的子树渲染
最后，如果你在组件中实现了如下方法的话，你还可以阻止一些子树的重新渲染：

```boolean shouldComponentUpdate(object nextProps, object nextState)```

根据该组件前后state/props的对比，你可以通知React不需要改变或者没必要重新渲染。合理地使用这个方法，可以极大提升应用性能。

为了能够使用它，你必须要能够比较JavaScript对象。这里有许多issues值得探讨，比如应该是浅比较还是深比较。如果是深比较，我们是应该使用不可变数据结构还是执行深拷贝？

你还需要记住的是，这个函数会一直执行，所以必须确保它的计算耗时要小于重新渲染组件的耗时，即使这个重新渲染不是必须的。 


### 总结
这种让React变快的技术并不新鲜。长久以来，我们就知道操作DOM十分费时，你应该对读写操作进行合并，使用事件代理技术性能更好等等…

人们始终在谈论它们，是因为在实践中使用常规的JavaScript代码很难实现它们。而是React如此出众的原因就是，所有这些优化手段都是默认的。这使你很难让自己的应用变慢，就像你不会搬起石头砸自己的脚。

React的性能消耗模型也很容易理解：每一次setState都会重新渲染所有子树。如果你想继续压榨性能，尽量减少setState的使用频率，并且使用shouldComponentUpdate来阻止大型子树的重新渲染