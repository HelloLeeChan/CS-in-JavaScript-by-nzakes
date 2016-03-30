原文地址:[Computer science in JavaScript: Base64 encoding](https://www.nczonline.net/blog/2009/12/08/computer-science-in-javascript-base64-encoding/)

不久前， 我写了关于[data URI](https://www.nczonline.net/blog/2009/10/27/data-uris-explained/)的博客并放出了一组创建他们的[工具](http://www.nczonline.net/blog/2009/11/03/automatic-data-uri-embedding-in-css-files/) 这其中的一个关键部分就是URI方程 － base64 编码。 Base64编码 在[RFC3548](http://tools.ietf.org/html/rfc3548)中与base16以及base32一起都有详细的描述。这些方式都被用作在有限的字符集中表示单字节数据。对于base64一个很常见的误解就是将其误认为是一种用来隐藏底层数据的加密(encryption)算法，实际上Base64编码（encoding）并不包含任何加密，它只是简单地将一个数据格式转换成另一种的简单算法。

所有以上三种编码，base16，base32，以及base64都是为了在7位系统下（7-bit systems）安全地传输数据，避免数据损失的传输方式。传统Email系统就是一个7位系统，MIME的base64编码也是被设计成在系统间安全地传输数据的方法。总之，这三种编码方式都是为了避免8位数据在7位系统下传输失真而设计的。

###工作原理

Base64编码直接作用于底层二进制数据。使用base64编码并不是直接编码字符串，而是编码字符串中代表字符的字节。字符串中的每个字节都是一个8位子节（byte），base64中编码的每个字符却是由6位构成的，除了将8位数据转换成6位，Base64编码并不做其它的操作。

在base64的字母表中有65中可能的字符， 字母A－Z，a－z，数字0-9以及加号（＋）和左斜杠（／）第65个字符是等于号（＝）它被用作表示填充（padding），关于填充的概念将在稍后讨论。因此在base64编码过的字符串中，6位（6-bit）数字0由字母A表示，相应地1由B表示，依次类推。

要使数据可被base64编码， 至少需要24位有效数据（被6和8整除的最小整数） 所以任何3个字符的ASCII序列（three-character ASCII sequence）都可以很好地在base64中编码。假设一个字符串“hat”。首字母“h”用104或者二进制01101000表示，“a”则是97或者01100001，而“t”则是116或者01110100，将他们放在一起，则是:  

    01101000-01100001-01110100
    
在base64编码中转换后，将会被重新定义边界，也就是转换成6位数据，如下：

    011010-000110-000101-110100
    
这之后， 将每个6位数据转换成数字则是：

    26-6-5-52
然后，用base64码表中的字符代替数字，则变成：

    a-G-F-0
所以在base64中编码后“hat”就是“aGFo”了。这样的运算可以很好地工作因为这里正好有24位有效数据，或者3个ASCII字符。由于并非所有的字符串都可以被3整除，所以base64编码需要填充不足位从而便于编码。


###填充

编码过程中将转换每个24位数据，知道没有梗长的数据可被转换。这时会出现以下三种情况：

- 没有更多的位可被转换（原字符串被3整除的情况下）。
- 剩下8位有效数据。这种情况将右填充0到12位。
- 剩下16位有效数据。这种情况将右填充0到18位。

注意以上第二和第三种情况，右填充只在最靠近被6整除的位数据地方。每个6位片段（segment）都被转换成一个字符，然后两个或一个等号（＝）就被分别添加到其后面。每个等号都表示右两个额外的位填充被添加。这些字符无法表示所有原始的ASCII字符串。他们只是简单表示处理位置的标识，以便解码器处理base64编码的字符串。

比如，单词“hatch”，字母“h”使用104或二进制01101000表示，“a”是97或者01100001，“t”则是116或者0110100，“c”是99或者01100011，“h”是104或者01101000，连起来就是：

    01101000-01100001-01110100-01100011-01101000
转换成base64编码，创建6位分组：

    (011010-000110-000101-110100)(011000-110110-1000)
注意该序列中只有第一部分是完整的24位数据（第一个小括号里的数据），序列的第二部分只有16位数据。这种情况下，第二个分组应该填充两个0位以便创建18位分组：

    (011010-000110-000101-110100)(011000-110110-100000)
然后6位分组转换成字符：

    (a-G-F-0)(Y-2-g)
所以结果字符串就是“aGFoY2g”。但这并非最终的base64编码字符串，由于存在两个填充位，一个等号(=)必须添加到序列末尾，则结果为“aGFoY2g＝”

###JavaScript编码
很多语言中的base64编码可以直接处理字节或者字节数组（bytes arrays）由于js的原生数据无法处理这两者（译者疑问：ArrayBuffer算不算？）于是位运算变得非常重要。位运算将直接作用在表示数字的底层位数据上。尽管js的数字实际上存储成64位，无论位运算是否参与，整型值的处理都把他们当成32位。最复杂的地方是把三个8位数字转换成4个6位数字，这也是位运算所做的工作：

####位运算
假设有三个8位数字(字母表示数字，比如A是67)：

		AAAAAAAA-BBBBBBBB-CCCCCCCC
与其相等的6位数字是：

		AAAAAA-AABBBB-BBBBCC-CCCCCC
		
注意，第一组6位数字由第一组8位数字的前六个数字组成。实际上你需要去除第一组8位数字中的后两位。这正好可以由位运算中的右移（>>）运算实现。比如数字`240`或者`11110000`，右移两位后就变成了`60`或者`00111100`。所有位向右移动两位，原来的末两位则删去，左移的话右边空出的位置被填充零。从第一组的8位数字中获取第一组6位数字可以这样做:

		var first6bitNum = first8bitNum >> 2;    //右移两位

第二组6位数字有点复杂，因为它由第一和第二组8位数字共同组成。第二组6位数字中，获取后4位的数字比较简单，只是从第二组8位数字中右移4位，就可以确定第二组6位数字中的后四位。但获取前2位就需要两步操作。

因为从第一组8位数字中我们只需要取后两位，所以可以将第一组8位数字与数字`3`（`00000011`）进行位与（AND）运算，位与运算只有在两个运算数值相同的情况下才会保留值，运算后结果将变成：

		    01100001
		AND 00000011
		------------
            00000001
            
可以看到后两位数字保留下来了。原来是`97`，现在变成了`1` 为了让这两位保留下来的位数字在第二组6位数字中待在正确的位置，需要将它们左移4位（以便腾出空间与第二组8位数字中的前4位组成第二个6位数字），然后使用位或运算合并，就生成了第二组6位数字：

		var second6bitNum = (first8bitNum & 3) << 4 | (second8bitNum >> 4);
		
对于第三个6位数字，处理过程也几乎一样。只是进行位与运算的对象变成了`15`（`00001111`）因为上一个（第二组）8位数字右移出了4位，所以第三个6位数字的前4位需要就需要取该数字的后四位，然后第三组8位数字右移出6位，与前面生成的数字进行位或运算：

		var third6bitNum = (second8bitNum & 0x0f) << 2 | (third8bitNum >> 6); 

最后一个6位数字也很简单，你只需要处理从最后一组8位数字钟移出的位，这一步只需要与`63`（`00111111`）进行位与运算就行了：

		var fourth6bitNum = third8bitNum & 0x3f; 

这样所有的6位数字都已经生成，然后可以分配一个base64的数字索引来表示他们的值，实际上就是将所有base64表示的数据罗列在一个字符串中，每个6位数字都在这个字符串中有一个索引：

		var digits = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
	var firstBase64Digit = digits.charAt(first6bitNum);   //获取第一个数字
	
这样就完成了ASCII到base64编码的基本转换。

####base64编码函数

在使用base64编码字符串之前，应该先检查该字符串是否只包含ASCII字符。因为base64编码需要8位的输入字符，也就是说任何字符代码超过255将无法准确表示，这种情况就需要抛出一个错误：

		function base64Encode(text){

    if (/([^\u0000-\u00ff])/.test(text)){
        throw new Error("Can't base64 encode non-ASCII characters.");
    } 

    //more code here
    }
这里使用一个简单的正则来校验是否存在0-255的字符，如果有的话将无法编码且抛出一个错误。

下一步的主要工作就是使用位运算将每个由三组8位数据组成的序列转换成由四组6位数据组成的序列。由于字符串（string）中每个字符都由一个8位数据表示，所以我们可以遍历整个字符串逐个处理其中的字符（character by character）

		function base64Encode(text){

    if (/([^\u0000-\u00ff])/.test(text)){
        throw new Error("Can't base64 encode non-ASCII characters.");
    } 

    var digits = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
        i = 0,
        cur, prev, byteNum,
        result=[];      

    while(i < text.length){

        cur = text.charCodeAt(i);
        byteNum = i % 3;

        switch(byteNum){
            case 0: //first byte
                result.push(digits.charAt(cur >> 2));
                break;

            case 1: //second byte
                result.push(digits.charAt((prev & 3) << 4 | (cur >> 4)));
                break;

            case 2: //third byte
                result.push(digits.charAt((prev & 0x0f) << 2 | (cur >> 6)));
                result.push(digits.charAt(cur & 0x3f));
                break;
        }

        prev = cur;
        i++;
    }

    //more code here

    return result.join("");
	}
	
由于每个三字节序列（three-byte sequence）字节的处理有点区别，变量`byteNum`存储当前被处理的三字节字符。当`byteNum`为0时，则为三字节序列中的第一个字节，依次类推，1就是第二个。这里使用了取模运算来计算。

该算法使用两个变量来追溯字符串的遍历过程，`cur`存储（原文是track）当前字符，`prev`存储之前的字符。这是很必要的，因为转成base64编码需要前一个字节的数据。`switch`语句用来选择使用哪种位运算（三种不同的情况罗列在前）。base64的值计算完成后，就在`digits`变量中查找出对应的base64字符。`digits`中顺序存储了素偶有base64可以表示的字符。所以可以把它看成一个查询表，通过`charAt()`找出对应位置。然后`result`数组再被合并(joined)。

最后一步工作则是完成一些字符串的填充：

		function base64Encode(text){

    if (/([^\u0000-\u00ff])/.test(text)){
        throw new Error("Can't base64 encode non-ASCII characters.");
    } 

    var digits = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
        i = 0,
        cur, prev, byteNum,
        result=[];      

    while(i < text.length){

        cur = text.charCodeAt(i);
        byteNum = i % 3;

        switch(byteNum){
            case 0: //first byte
                result.push(digits.charAt(cur >> 2));
                break;

            case 1: //second byte
                result.push(digits.charAt((prev & 3) << 4 | (cur >> 4)));
                break;

            case 2: //third byte
                result.push(digits.charAt((prev & 0x0f) << 2 | (cur >> 6)));
                result.push(digits.charAt(cur & 0x3f));
                break;
        }

        prev = cur;
        i++;
    }

    if (byteNum == 0){
        result.push(digits.charAt((prev & 3) << 4));
        result.push("==");
    } else if (byteNum == 1){
        result.push(digits.charAt((prev & 0x0f) << 2));
        result.push("=");
    }

    return result.join("");
	}
这一步多亏有了`byteNum`变量，因为while循环结束后如果`byteNum`是2，就意味着恰好完成了所有字节的编码。如果`byteNum`是其他数值，那么就需要填充了（padding）。所以，如果`byteNum`是0，那么就还剩下一个8位字节，转换后就需要左移运算（left-shift）填充4位（8位转换成两个6位）。如果`byteNum`是1，那么就意味着有两个8位字节，那么需要填充2位。

最后，`result`数组合并后返回，就完成了原始字符串的base64编码转换。

###JavaScript解码
待续……
