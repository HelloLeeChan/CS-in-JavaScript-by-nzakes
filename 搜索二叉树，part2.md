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

        //find the node (removed for space)

        //只处理存在的节点
        if (found){

            //计算子节点数量
            childCount = (current.left !== null ? 1 : 0) + 
                         (current.right !== null ? 1 : 0);

            //特殊情况，该节点是根节点
    if (current === this._root){
                switch(childCount){

                    //没有子节点，直接删除                    case 0:
                        this._root = null;
                        break;

                    //存在一个子节点                    case 1:
                        this._root = (current.right === null ? 
                                      current.left : current.right);
                        break;

                    //两个子节点，略复杂                    case 2:

                        //TODO

                    //no default

                }        

            //不存在根节点
            } else {

                switch (childCount){

                    //不存在子节点，直接从父节点删除                    case 0:
                        //如果当前节点小于其父节点                        //左指针指向null                        if (current.value < parent.value){
                            parent.left = null;

                        //if the current value is greater than its
                        //parent's, null out the right pointer
                        } else {
                            parent.right = null;
                        }
                        break;

                    //one child, just reassign to parent
                    case 1:
                        //if the current value is less than its 
                        //parent's, reset the left pointer
                        if (current.value < parent.value){
                            parent.left = (current.left === null ? 
                                           current.right : current.left);

                        //if the current value is greater than its 
                        //parent's, reset the right pointer
                        } else {
                            parent.right = (current.left === null ? 
                                            current.right : current.left);
                        }
                        break;    

                    //two children, a bit more complicated
                    case 2:

                        //TODO          

                    //no default

                }

            }

        }

    },

    //more code here

	};

当处理根节点时，只需要简单地将其重写(overwriting)它。
明儿回公司继续翻...