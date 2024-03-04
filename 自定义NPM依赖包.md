1. 创建一个空项目，即空文件夹

2. 在命令行中执行初始化命令

   ```shell
   npm init -y
   ```

3. 系统会自动创建`package.json`文件

4. 创建`src`文件夹，之后项目源代码就放在其中，在`src`中创建入口文件`index.ts`

5. 添加`TypeScript`依赖，执行命令

   ```shell
   npm i typescript -s
   ```

6. 在项目根目录下创建`TypeScript`打包配置文件`tsconfig.json`，[编译选项官方文档](https://www.tslang.cn/docs/handbook/compiler-options.html)，以下是一个参考选项

   ```json
   {
     "compilerOptions": {
       "target": "es6",
       "module": "commonjs",
       "declaration": true,
       "outDir": "./dist",
       "strict": true
     }
   }
   ```

7. 在`scr`文件夹中添加项目源代码，并在`index.ts`中导出需要导出的方法或类

8. 命令行中执行`tsc`执行进行编译

9. 修改`package.json`文件，修改`main`字段指向编译后的`index.js`文件，增加字段`types`，指向同目录下的`index.d.js`文件

