---
title: ES6--对象和数组的解构
date: 2020-06-09
tags:
- javascript
- ES6
categories: 
- javascript
- ES6
sidebarDepth: 2
---

# JavaScript 深度剖析

## ES6--对象和数组的解构

  ### 为何使用解构功能  
  在ES5及早期版本中，开发者们为了从对象和数组中获取特定数据并赋值给变量，编写了许多看起来同质化的代码，如下：  
  ```javascript
    let options = {
      repeat: true,
      save: false
    };
    //从对象中取数据
    let repeat = options.repeat,
        save = options.save;
  ```
  这段代码从 <code>options</code> 对象中提取了 <code>repeat</code> 和 <code>save</code> 的值并将其存储为同名局部变量，提取的过程极为相似，想象一下，如果你要提取更多变量，则必须依次编写类似的代码来为变量赋值，如果其中还包含嵌套结构，只靠遍历是找不到真实信息的，必须要深入挖掘整个数据结构才能找到所需数据。

  ### 对象解构  
  对象解构的语法形式是在一个赋值操作左边放置一个对象字面量，如：
  ```javascript
    let node = {
      type: 'Identifier',
      name: 'foo'
    };
    let {type, name} = node;
    console.log(type);   // Identifier
    console.log(name);   // foo
  ```
  在这段代码中，node.type的值被存储在名为type的变量中；node.name的值被存储在名为name的变量中。

  *注意：如果使用var、let、const结构声明变量，必须要提供初始化程序（也就是等号右侧的值），否则会导致抛出语法错误。*

  **解构赋值**  
  同样可以在给变量赋值时使用解构语法，如下，在定义变量之后想要修改他们的值，可以这样：

  ```javascript
  let node = {
    type:"Identifier",
    name:"foo"
  },
  
  type = "Literal",
  name = 5;
  
  //使用解构语法为多个变量赋值
  ({type,name} = node);
  
  console.log(type);   //"Identifier"
  console.log(name);   //"foo"
  ```
  在这个实例中，声明变量type和name时初始化了一个值，在后面的几行中，通过解构赋值的方法，从node对象读取相应的值重新为这两个变量赋值。注意： **一定要用一对小括号包裹解构赋值语句，JavaScript引擎将一对开放的花括号视为一个代码块，而语法规定，代码块不能出现在赋值语句的左侧，添加小括号后可以将块语句转化为一个表达式，从而实现整个解构赋值的过程。**

  **默认值**  
  使用解构赋值表达式时，如果指定的局部变量名称在对象中不存在，那么这个局部变量会被赋值为 <code>undefined</code> 如下：

  ```javascript
  let node = {
    type:"Identifier",
    name:"foo"
  };
  let {type,name,value} = node;
  console.log(type);    //"Identifier"
  console.log(name);    //"foo"
  console.log(value);   //undefined
  ```

  当指定的属性不存在时，可以随意定义一个默认值，在属性名称后添加一个等号（=）和相应的默认值即可：

  ```javascript
  let node = {
    type:"Identifier",
    name:"foo"
  };
  let {type,name,value = true} = node;
  console.log(type);    //"Identifier"
  console.log(name);    //"foo"
  console.log(value);   //true
  ```

  **为非同名局部变量赋值**  
  到目前为止的每一个实例中，解构赋值使用的都是与对象属性同名的局部变量，列如，<code>node.type</code> 的值被存储在了变量 <code>type</code> 中。但如果你希望使用不同命名的局部变量来存储对象属性的值，ES6中的一个扩展语法可以满足：

  ```javascript
  let node = {
    type:"Identifier",
    name:"foo"
  };
  let {type:localType, name:localName} = node;
  console.log(localType);   //"Identifier"
  console.log(localName);   //"foo"
  ```

  当使用其他变量名进行赋值时也可以添加默认值，只需在变量名后添加等号和默认值即可：

  ```javascript
  let node = {
    type:"Identifier"
  };
  let {type:localType,name:localName = "bar"} = node;
  console.log(localType);     //"Identifier"
  console.log(localName);     //"bar"
  ```

  **嵌套对象解构**  
  ```javascript
  let node = {
    type: 'Identifier',
    name: 'foo',
    loc: {
      start: {
        line: 1,
        column: 1
      },
      end: {
        line: 1,
        column: 4
      }
    }
  };
  
  let {loc:{start}} = node;
  console.log(start.line);      //1
  console.log(start.column);    //1
  ```

  在这个示例中，我们在解构模式中使用了花括号，其含义为在找到node对象中的loc属性后，应当深入一层继续查找start属性。更进一步，也可以使用一个与对象属性名不同的局部变量名：

  ```javascript
  let node = {
    type:"Identifier",
    name:"foo",
    loc:{
        start:{
          line:1,
          column:1
        },
        end:{
          line:1,
          column:4
        }
    }
  };
  //提取node.loc.start
  let {loc:{start:localStart}} = node;
  console.log(localStart.line);     //1
  console.log(localStart.column);   //1
  ```

  ### 数组解构  
  与对象解构的语法相比，数组解构就简单多了，它使用的是数组字面量，且解构操作全部在数组内完成，而不是像对象字面量语法一样使用对象的命名属性：

  ```javascript
  let colors = ["red","green","blue"];
  let [firstColor,secondColor] = colors;
  console.log(firstColor);    //"red"
  console.log(secondColor);   //"green"
  ```

  在这段代码中，我们从colors数组中解构出了"red"和"green"这两个值，并分别存储在变量firstColor和变量secondColor中。在数组解构语法中，我们通过**值在数组中的位置进行选取**，且可以存储在任意变量中，未显式声明的元素都会直接被忽略。在这个过程中，数组本身不会发生任何变化。  

  在解构模式中，也可以直接省略元素，只为感兴趣的元素提供变量名。比如，如果你只想取数组中的第三个值，则不需要提供第一个和第二个元素的变量名称：

  ```javascript
  let colors = ["red","green","blue"];
  let [ ,,thirdColor] = colors;
  console.log(thirdColor);//"blue"
  ```

  **解构赋值**

  数组解构也可用于赋值上下文，但不需要用小括号包裹表达式，这一点与对象解构的约定不同。

  ```javascript
  let colors = ["red","green","blue"],
    firstColor = "black",
    secondColor = "purple";
  [firstColor,secondColor] = color;
  console.log(firstColor);    //"red"
  console.log(secondColor);   //"green"
  ```

  数组解构还有一个独特的用例：交换两个变量的值。在ES5中交换两个变量的值需要引入第三个临时变量，但在ES6的数组解构中，就不再需要额外的变量了，如下：

  ```javascript
  let a = 1,
      b = 2;
 
  [a,b] = [b,a];
  
  console.log(a);//2
  console.log(b);//1
  ```

  **默认值**

  也可以在数组解构赋值表达式中为数组中的任意位置添加默认值，当指定位置的属性不存在或其值为undefined时使用默认值：

  ```javascript
  let colors = ["red"];
  let [firstColor,secondColor = "green"] = colors;
  console.log(firstColor);    //"red"
  console.log(secondColor);   //"green"
  ```

  **嵌套数组解构**

  ```javascript
  let colors = ["red",["green","lightgreen"],"blue"];
 
  let [firstColor,[secondColor]] = colors;
  
  console.log(firsrColor);    //"red"
  console.log(secondColor);   //"green"
  ```

  在此示例中，变量 <code>secondColor</code> 引用的是 <code>colors</code> 数组中的值 <code>green</code>,该元素包含在数组内部的另一个数组中，所以<code>secondColor</code> 两侧的方括号是一个必要的解构模式。同样，在数组中也可以无限深入去解构，就像在对象中一样。

  **不定元素**

  在数组中，可以通过 <code>...语法</code> 将数组中的其余元素赋值给一个特定的变量，如下：

  ```javascript
  let colors = ["red","green","blue"];
 
  let [firstColor,...restColors] = colors;
  
  console.log(firstColor);          //"red"
  console.log(restColors.length);   //2
  console.log(restColors[0]);       //"green"
  console.log(restColors[1]);       //"blue"
  ```
  **注意：在被解构的数组中，*不定元素必须为最后一个条目*，在后面继续添加逗号会导致程序抛出语法错误**

  **混合解构**  
  可以混合使用对象解构和数组解构来创建更多复杂的表达式，如此一来，可以从任何混杂着对象和数组的数据解构中提取你想要的信息：  
  ```javascript
  let node = {
    type:"Identifier",
    name:"foo",
    loc:{
        start:{
          line:1,
          column:1
        },
        end:{
          line:1,
          column:1
        }
     },
     range:[0,3]
  };
  
  let {
      loc:{start},
      range:[startIndex]
  } = node;
  
  console.log(start.line);    //1
  console.log(start.column);  //1
  console.log(startIndex);    //0
  ```  

  这段代码分别将 <code>node.loc.start</code> 和 <code>node.range[0]</code> 提取到变量 <code>start</code> 和 <code>startIndex</code> 中。记住：解构模式中的<code>loc:</code> 和 <code>range:</code> 仅代表他们在对象中所处的位置，也就是该对象的属性。
