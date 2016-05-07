原文地址:[Computer science in JavaScript: Binary search tree, Part 2](https://www.nczonline.net/blog/2009/06/16/computer-science-in-javascript-binary-search-tree-part-2/)


在我[之前的博客](https://github.com/HelloLeeChan/CS-in-JavaScript-by-nzakes/blob/master/%E4%BA%8C%E5%8F%89%E6%90%9C%E7%B4%A2%E6%A0%91%2C%20Part1.md)中,我用JavaScript大致创建了一个二叉搜索树(BST),那篇博客中讨论了在BST中增加节点等。但是有一个关键部分没有提到，就是如何删除在BST中删除节点。因为要保持树的平衡(保持左边的值依然小于右边)，所以在BST中删除节点比较复杂。

当删除节点时，需要确定该节点是否为根节点，对于根节点的处理类似于其他特殊情况的处理，一般在最后将其设置为其他值。为了简单起见，它将被当做特殊情况处理。

删除节点的第一步是确定该节点是否存在：

		BinarySearchTree.prototype = {

    //其他操作

    remove: function(value){

        var found       = false,
            parent      = null,
            current     = this._root,
            childCount,
            replacement,
            replacementParent;

        //确定是否存在要搜索的节点
        while(!found && current){

            //节点小于当前节点，往左遍历
            if (value < current.value){
                parent = current;
                current = current.left;

            //反之向右
            } else if (value > current.value){
                parent = current;
                current = current.right;

            //节点相等，表示找到了
            } else {
                found = true;
            }
        }

        //只在找到节点的情况下进行下一步处理
        if (found){
            //continue
        }

    },

    //其他代码

    };
    
`remove()`方法的第一步是在BST中定位要删除的节点，当要删除的节点小于当前节点的时候向左遍历，反之向右。遍历过程中，父节点`parent`始终被记录，因为最终需要从要删除节点的父节点上删除。当`found`为`true`，表示找到要删除的节点。

删除节点的时候有三种情况需要考虑:

 1. 叶子节点
 2. 该节点只有一个子节点
 3. 该节点有两个子节点
 
除叶子节点以外，BST上的删除节点操作都需要保持树的左右平衡。前两种情况相对来说比较容易实现，叶子节点直接删除，只有一个子节点的情况下只需要用其子节点将其替代即可。最后一种情况有点复杂，所以放在最后处理。

在了解如何删除节点前，你需要知道要删除的节点上存在多少子节点。确定子节点后，你需要确定该节点是否根节点，这是个比较简单的决策树：

		BinarySearchTree.prototype = {

    //其他代码

    remove: function(value){

        var found       = false,
            parent      = null,
            current     = this._root,
            childCount,
            replacement,
            replacementParent;

        //找到节点

        //只处理存在的节点
        if (found){

            //计算子节点数量
            childCount = (current.left !== null ? 1 : 0) + 
                         (current.right !== null ? 1 : 0);

            //特殊情况，该节点是根节点
    	if (current === this._root){
                switch(childCount){

                    //没有子节点，直接删除                   
                    case 0:
                        this._root = null;
                        break;

                    //存在一个子节点                                       
                    case 1:
                        this._root = (current.right === null ? 
                                      current.left : current.right);
                        break;

                      //两个子节点，略复杂                    
                      case 2:

                        //TODO

                    //no default

                }        

            //不存在根节点
        } else {

                switch (childCount){

                    //不存在子节点，直接从父节点删除                    
                    case 0:
                        //如果当前节点小于其父节点，父节点指向null或者被删除节点的左节点                      
                        if (current.value < parent.value){
                            parent.left = null;

    			//如果当前节点大于父节点，父节点指向null或者被删除节点的右节点                                               
    			} else {
        			 parent.right = null;
                        	}
                        	break;

                    //一个子节点的情况下只需重置父节点                    
                    case 1:
                        //如果当前节点小于父节点，重置其左指针                        
                        if (current.value < parent.value){
                            parent.left = (current.left === null ? 
                                           current.right : current.left);

                        //如果当前节点大于父节点，则重置其右指针                        
                        } else {
                            parent.right = (current.left === null ? 
                                            current.right : current.left);
                        }
                        break;    

                    //两个子节点的情况下需要更多操作                    
                    case 2:

                        //TODO          

                    //no default

                }

            }

        }

    },

    //其他代码

	};

当处理根节点时，只需要简单地将其重写(overwriting)它。对于非根节点，其父节点的指针指向必须基于要删除的节点的值：当要删除的值小于父节点，父节点的左指针(left)必须被重置为`null`(被删除节点不含子节点的情况下)或者被删除节点的左指针所指向的节点；反之如果要删除节点的值大于父节点，此时父节点的右指针(right)必须被重置为`null`或者被删除节点右指针所指向的节点。

正如之前所提到的，删除包含两个子节点的节点操作最为复杂，考虑以下二叉树：

![image](https://www.nczonline.net/images/wp-content/uploads/2009/06/500px-Binary_search_tree.svg_-300x250.png)

根节点为8，当删除其左子节点3会怎么样？存在两种可能：节点1(3的左子节点，也被称为直接前驱，原文：in-order predecessor)可以取代3，或者右子树的最左节点4可以取代3，译者注：也就是说要么取左边最大的，要么取右边最小的。

以上两种方案都可行， 要找到直接前驱(in-order predecessor)在删除节点以前，找到被删除节点的左子树中最右的节点，要找到直接后继，找到被删除节点右子树中最左的节点。这两个操作都需要额外的遍历来完成：

	BinarySearchTree.prototype = {

    //more code here

    remove: function(value){

        var found       = false,
            parent      = null,
            current     = this._root,
            childCount,
            replacement,
            replacementParent;

        //找到节点

        //只处理存在的节点
        if (found){

            //计算子节点数量
            childCount = (current.left !== null ? 1 : 0) + 
                         (current.right !== null ? 1 : 0);

            //处理根节点
            if (current === this._root){
                switch(childCount){

                    //其他情况

                    //两个子节点的情况下更复杂
                    case 2:

                        //新的根节点是原根节点的左子节点
                        //...maybe
                        replacement = this._root.left;

                        //找到最右节点
                        //新的根节点
                        while (replacement.right !== null){
                            replacementParent = replacement;
                            replacement = replacement.right;
                        }

                        //it's not the first node on the left
                        if (replacementParent !== null){

                            //从其之前的位置删除
                            replacementParent.right = replacement.left;

                            //为新的跟节点设置子节点
                            replacement.right = this._root.right;
                            replacement.left = this._root.left;
                        } else {

                            //分配子节点                            
                            replacement.right = this._root.right;
                        }

                        //officially assign new root
                        this._root = replacement;

                    //no default

                }        

            //非根节点的处理
            } else {

                switch (childCount){

                    //其他情况

                    //删除包含两个子节点的节点的情况最为复杂
                    case 2:

                        //重置指针，为遍历做准备
                        replacement = current.left;
                        replacementParent = current;

                        //找出最右节点
                        while(replacement.right !== null){
                            replacementParent = replacement;
                            replacement = replacement.right;
                        }

                        replacementParent.right = replacement.left;

                        //为替换节点分配节点
                        replacement.right = current.right;
                        replacement.left = current.left;

                        //正确安插替换节点
                        if (current.value < parent.value){
                            parent.left = replacement;
                        } else {
                            parent.right = replacement;
                        }          

                    //no default

                }

            }

        }

    },

    //more code here

	};

以上代码对于包含两个子节点的根节点与非根节点的处理方式几乎是一样的。 其逻辑就是每次都寻找直接前驱(左子树中最右的节点)。在`while`循环中用到了`replacement`和`replacementParent`两个变量。替换节点`replacement`将会取代`current`的位置，`replacement`将从其原来的位置上移除，同时这里需要重新设置其原父节点的`right`指针到它的`left`指针上。对于根节点的处理，因为`replacementParent`是`null`，且替代节点(replacement)是根节点的直接子节点，所以直接将`replacement`的`right`节点设置为根节点的右节点。最后一步是把替代节点安插到正确的位置。 对于删除根节点，替换节点被设置为新的根节点，对于删除非根节点， 替换节点被安插到合适位置。

这里需要注意的是： 在大部分值在树一侧的BST中，总是使用直接前驱替换节点会导致二叉树的不平衡， 不平衡二叉树将导致搜索效率降低， 并且会影响实际中的应用。 有些BST的会判断到底是使用直接前驱还是直接后继作为替换节点来保证BST的平衡(比如所谓的自平衡BST)

关于二叉搜索树的源码可以从我的github下载[Computer Science in JavaScript GitHub project](https://github.com/nzakas/computer-science-in-javascript/) 你也可以看看[Isaac Schlueter](http://foohack.com/)的二叉树实现[GitHub fork](https://github.com/isaacs/computer-science-in-javascript/)
