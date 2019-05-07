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