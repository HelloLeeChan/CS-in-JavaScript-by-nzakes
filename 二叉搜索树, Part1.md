原文地址:[Computer science in JavaScript: Binary search tree, Part 1](https://www.nczonline.net/blog/2009/06/09/computer-science-in-javascript-binary-search-tree-part-1/)


在数据结构与算法课程中可能被提及最多的就是二叉搜索树了（[Binary Search Tree](https://en.wikipedia.org/wiki/Binary_search_tree)以下简称BST）在本系列文章中它是首次被介绍的典型数据结构，它具有非线性的插入算法。二叉搜索树有点类似于双重链表（linked list），他们的每个节点都含有数据，另外有两个指针（pointer）指向其他节点，两者区别在于节点之间的关系，二叉搜索树的节点指针被称为“左（left）”与“右（right）”，分别表示子节点与当前节点的关系。这样一个节点在Javascript中实现如下：

		var node = {
    value: 125,
    left: null,
    right: null
	};
顾名思义，BST由层级树构成，第一个元素作为根节点也就是祖先节点，其他元素都是后来加入该树状结构的。BST的特点是其节点顺序基于节点值的大小，所有左节点的值都小于右节点的值。这样的话在BST上查找一个值将变得非常简单，如果要搜索的值小于当前节点那么只管向左查找，反之则向右。在BST上没有重复值，因为那样的话就会破坏二叉树的结构。下图展示了一个简单的BST：

![image](https://www.nczonline.net/images/wp-content/uploads/2009/06/500px-Binary_search_tree.svg_-300x250.png)

如图所示根节点值为8，当加入3的时候，由于3小于8，所以3就变成8的左子节点，当加入1时，1小于8同时又小于3，所以1就变成3的左子节点，当加入10时，10变成根节点的右节点，因为10大于8.依次类推地处理6，4，7，14以及13。BST的深度为3，表示最远的值离根节点有3个节点的距离。

基于这种数据结构， BST自然就是有序排列，这样对于数据查找十分有用，因为每步查询可以立即跳过不可能的值。通过限制要查询的节点数量，查询过程可以变得更为高效。假设要搜索的值为6（以上图为例），从根节点开始，6小于8，所以向左遍历，由于6大于3，然后向右遍历，很快就找到了目标值。使用穷举的查找需要查询9个值，而BST只查询3个值就可以了。

使用javascript构建BST，首先需要定义基础接口，如下：

		function BinarySearchTree() {
    this._root = null;
	}

	BinarySearchTree.prototype = {

    //存储构造函数
    constructor: BinarySearchTree,

    add: function (value){
    },

    contains: function(value){
    },

    remove: function(value){
    },

    size: function(){
    },

    toArray: function(){
    },

    toString: function(){
    }

	};
	
类似于其他数据结构，基础接口有添加和移除方法，我在里面又加了一些方法便于使用，`size()`,`toArray()`以及`toString()`

处理一个BST，最好的方法是从`contains()`方法开始，`contains()`接受一个值作为参数并返回一个布尔值判断该值是否在BST中。`contains()`方法使用基础的二分查找算法来查询要搜索的值是否在BST中：

		BinarySearchTree.prototype = {

    //more code

    contains: function(value){
        var found       = false,
            current     = this._root

        //判断要搜索的值是否存在
        while(!found && current){

            //小于当前节点值向左遍历
            if (value < current.value){
                current = current.left;

            //大于当前节点值向右遍历
            } else if (value > current.value){
                current = current.right;

            //值相等，则查找成功
            } else {
                found = true;
            }
        }

        //只查找存在的节点
        return found;
    },

    //more code

	};

查询从根节点开始，由于如果没有数据加入的话，就不存在根节点，所以必须先检验。遍历树的算法很简单在前面已经讲过了：变量`current`每次都被重写（overwritten），直到查询结束或者该方向没有后续节点。

另外，当往BST中插入新值的时候，也可用到这里的`contains()`方法。不同的是插入新值所查找的是新值位置。

		BinarySearchTree.prototype = {

    //more code

    add: function(value){
        //create a new item object, place data in
        var node = {
                value: value,
                left: null,
                right: null
            },

            //用于遍历时保存当前节点的变量
            current;

        //特殊情况: 空树
        if (this._root === null){
            this._root = node;
        } else {
            current = this._root;

            while(true){

                //新值小于当前值向左遍历
                if (value < current.value){

                    //不存在左子节点，则就作为新节点位置
                    if (current.left === null){
                        current.left = node;
                        break;
                    } else {
                        current = current.left;
                    }

                //新值大于当前值向右遍历
                } else if (value > current.value){

                    //不存在右子节点，则就作为新节点位
                    if (current.right === null){
                        current.right = node;
                        break;
                    } else {
                        current = current.right;
                    }       

                //新值与当前值相当，则忽略
                } else {
                    break;
                }
            }
        }
    },

    //more code

	};
	
往BST中插入新值的时候，在不存在根节点的特殊情况下，只需要把新值作为根节点就行了。其他情况的基础算法与之前的茶盏类似，主要的区别在于不存在子节点的时候，该位置就是要插入新值的地方。因为BST中不能有重复值，所以碰到值相等时，就停止操作且自动忽略。

在讨论`size()`方法前，我想先说下关于遍历树的方法。要计算出树的大小，需要遍历树的每个节点。通常BST带有不同的遍历方法，最常用的是中序遍历（in-order traversal），中序遍历从左子树最左一个节点开始查询每个节点，然后是当前节点，再之后右节点，因为BST以从左到右的顺序构建，所以就有了这样的遍历顺序。一般对于`size()`方法来说，遍历顺序无关紧要，可对于`Array()`，遍历顺序就有影响了。由于这两个方法都需要用到遍历，我就写了一个`traverse()`方法：

		BinarySearchTree.prototype = {

    //more code

    traverse: function(process){

        //辅助函数
        function inOrder(node){
            if (node){

                //traverse the left subtree
                if (node.left !== null){
                    inOrder(node.left);
                }            

                //在此节点上调用process方法                process.call(this, node);

                //遍历右子树
                if (node.right !== null){
                    inOrder(node.right);
                }
            }
        }

        //从根节点开始
        inOrder(this._root);
    },

    //more code

	};
	
该方法接受一个单独的`process`函数作为参数，该函数在每个节点都会执行。该方法还定义了一个辅助函数`inOrder()`用于递归地遍历BST，该递归执行必须在非null的节点上。然后`traverse()`方法从根节点开始，`process()`函数在每个节点执行。之后这个方法将被用于实现`size()`，`toArray()`以及`toString()` :

		BinarySearchTree.prototype = {

    //more code

    size: function(){
        var length = 0;

        this.traverse(function(node){
            length++;
        });

        return length;
    },

    toArray: function(){
        var result = [];

        this.traverse(function(node){
            result.push(node.value);
        });

        return result;
    },

    toString: function(){
        return this.toArray().toString();
    },

    //more code

	};

`size()`和`toArray()`都调用了`traverse()`方法并且传入一个函数在每个节点执行。`size()`使用这个执行函数递增长度变量，而`toArray()`使用执行函数把节点值添加进数组中。之后`toString()`方法调用`toArray()`返回一个字符串。

在 part2的文章中，我们将讨论从BST中删除节点。删除节点较为复杂，很多因素需要考虑。