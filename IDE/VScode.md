### 使用VScode file template:

### 例如添加sol的template:

打开User Snippets:

<img src="../img/image-20220424164927842.png" alt="image-20220424164927842" style="zoom:50%;float:left" />

找到solidity.json

```json
{
  // Place your snippets for solidity here. Each snippet is defined under a snippet name and has a prefix, body and 
  // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
  // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
  // same ids are connected.
  // Example:
  // "Print to console": {
  //  "prefix": "log",
  //  "body": [
  //    "console.log('$1');",
  //    "$2"
  //  ],
  //  "description": "Log output to console"
  // }
  "Print to conaole":{
​    "prefix": "sol",    //在新建立的页面中输入sol就会有智能提示，Tab就自动生成好了
​    "body": [
​        "// SPDX-License-Identifier: MIT",
​        "pragma solidity ^0.8.7;",     //这个头文件可以删除，我为了使用方便就加了
​        "", //空行
​        "contract $1 {", //标准命名空间

​        "}",
​    ],
​    "description": "A sol file template."   //用户输入后智能提示的内容（你可以用中文写“生成C++模板”）
	}
}
```

