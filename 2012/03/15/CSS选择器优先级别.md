# CSS选择器优先级别

此文回iOpenV的侯先生.关于CSS使用ID选择器的好处.

<!-- more -->

原文出处W3C:

http://www.w3.org/TR/CSS21/cascade.html

资料6.4.3节关于选择器优先级别是这么说的.

```
6.4.3 Calculating a selector's specificity

A selector's specificity is calculated as follows:

count 1 if the declaration is from is a 'style' attribute rather than a rule with a selector, 0 otherwise (= a) (In HTML, values of an element's "style" attribute are style sheet rules. These rules have no selectors, so a=1, b=0, c=0, and d=0.)
count the number of ID attributes in the selector (= b)
count the number of other attributes and pseudo-classes in the selector (= c)
count the number of element names and pseudo-elements in the selector (= d)
The specificity is based only on the form of the selector. In particular, a selector of the form "[id=p33]" is counted as an attribute selector (a=0, b=0, c=1, d=0), even if the id attribute is defined as an "ID" in the source document's DTD.

Concatenating the four numbers a-b-c-d (in a number system with a large base) gives the specificity.

Some examples:

 *             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
 li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
 li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
 h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
 ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
 li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
 #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
 style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
<HEAD>
<STYLE type="text/css">
  #x97z { color: red }
</STYLE>
</HEAD>
<BODY>
<P ID=x97z style="color: green">
</BODY>
In the above example, the color of the P element would be green. The declaration in the "style" attribute will override the one in the STYLE element because of cascading rule 3, since it has a higher specificity.
```

顺便一说,有兴趣的话,以前有一个用JS创建表格分别使用id和class进行操作的例子.后者明显十分慢...


