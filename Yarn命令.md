Yarn命令
---

## 安装 

```npm install -g yarn```

## 常用命令

|npm命令                           |yarn命令                           |说明                                    |
|:---------------------:|:--------------------:|:-----------------------:|
|```npm install ```                  | ```yarn```                         |install 安装是默认行为              |
|```npm install taco --save ```| ```yarn add taco```            |taco 包立即被保存到 package.json 中| 
|```npm uninstall taco --save```| ```yarn remove taco```  |在 yarn 中，在package.json 中添加（add）和移除（remove）等行为是默认的 |       
|```npm install taco --save-dev ``` | ```yarn add taco --dev```||
|```npm update --save``` | ```yarn upgrade```||
|```npm install taco --global``` | ```yarn global add taco```|请谨慎使用 global 标记|
|```npm init ``` | ```yarn init```||
|```npm link``` | ```yarn link```||
|```npm outdated``` | ```yarn outdated```||
|```npm publish``` | ```yarn publish```||
|```npm run``` | ```yarn run```||
|```npm cache clean``` | ```yarn cache clean```||
|```npm login``` | ```yarn login```|运行命令|
|```npm test``` | ```yarn test```||

## Yarn独有命令

|yarn命令                           |说明                                      |
|:---------------------:|:-----------------------:|
| ```yarn licenses ls```           |允许你检查依赖的许可信息|
| ```yarn licenses generate``` |自动创建依赖免责声明 license|
| ```yarn why taco```             |检查为什么会安装 taco，详细列出依赖它的其他包|




