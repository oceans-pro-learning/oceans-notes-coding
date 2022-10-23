要想项目识别`import`和`export`的语法

如果node 13.2.0之前
1. 需要在package.json中添加属性`type = "module"`即可。
2. 在执行命令（一般在package.jso的scripts选项）中添加如下选项
`node --experimental-modules src/index.js`。

如果node13.2.0之后，执行上述的1即可。