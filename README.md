# DOM-diff
 ### DOM DIFF算法简析
 #### diff策略
 #### diff 粒度
 #### diff 算法题
  - 请用递归的方式遍历树形数据结构中的每一个节点
  - 将类似以下JSON表示的树状结构(可以无限层级)通过parseDOM函数(使用document.createElement,document.createTextNode,appendChild等方法)生成一颗DOM树(返回一个element元素)
  - 运用递归的方式进行算法 用自己调用自己的思想
  
  - 在日常的开发过程中经常会遇到这种类型的数据。主要考我们对递归算法的熟练程度。具体的知识点就是题中列出的3个DOM操作的知识。
  - eg：
  ```const JsonTree = {
            "tagName": "ul",
            "props": {
                "className": "list",
                "data-name": "jsontree"
            },
            "children": [{
                "tagName": "li",
                "children": [{
                    "tagName": "a",
                    "props": {
                        "href": "https://www.aliyun.com",
                        "target": "_blank"
                    },
                    "children": "阿里云"
                }]
            }, {
                "tagName": "li",
                "children": [{
                    "tagName": "img",
                    "props": {
                        "src": "./1.jpg",
                        // "width": "16px"
                    }
                }]
            }]


        }
  ```
  说到DOM-diff那一定要清楚其存在的意义，给定任意两棵树，采用先序深度优先遍历的算法找到最少的转换步骤

   DOM-diff比较两个虚拟DOM的区别，也就是在比较两个对象的区别。

   作用： 根据两个虚拟对象创建出补丁，描述改变的内容，将这个补丁用来更新DOM
