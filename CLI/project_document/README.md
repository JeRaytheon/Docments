# 文档信息

|键|值|
|-|-|
|文档版本|0.1.0|
|最后编辑时间|2020-2-9|
|编辑人|fd|

-----

# 目录
- [文档信息](#%e6%96%87%e6%a1%a3%e4%bf%a1%e6%81%af)
- [目录](#%e7%9b%ae%e5%bd%95)
- [一、概述](#%e4%b8%80%e6%a6%82%e8%bf%b0)
  - [1.1 什么是CLI](#11-%e4%bb%80%e4%b9%88%e6%98%afcli)
  - [1.2 背景](#12-%e8%83%8c%e6%99%af)
  - [1.3 开发信息](#13-%e5%bc%80%e5%8f%91%e4%bf%a1%e6%81%af)
- [二、项目结构](#%e4%ba%8c%e9%a1%b9%e7%9b%ae%e7%bb%93%e6%9e%84)
  - [2.1 文件结构](#21-%e6%96%87%e4%bb%b6%e7%bb%93%e6%9e%84)
  - [2.2 逻辑结构](#22-%e9%80%bb%e8%be%91%e7%bb%93%e6%9e%84)
      - [命令](#%e5%91%bd%e4%bb%a4)
      - [resolvers](#resolvers)
      - [validators](#validators)
      - [model层](#model%e5%b1%82)
      - [table渲染](#table%e6%b8%b2%e6%9f%93)
      - [接口层](#%e6%8e%a5%e5%8f%a3%e5%b1%82)
- [三、开发实例](#%e4%b8%89%e5%bc%80%e5%8f%91%e5%ae%9e%e4%be%8b)
  - [3.1 写一个完整的command文件](#31-%e5%86%99%e4%b8%80%e4%b8%aa%e5%ae%8c%e6%95%b4%e7%9a%84command%e6%96%87%e4%bb%b6)
      - [命令项class](#%e5%91%bd%e4%bb%a4%e9%a1%b9class)
      - [添加参数](#%e6%b7%bb%e5%8a%a0%e5%8f%82%e6%95%b0)
      - [默认class](#%e9%bb%98%e8%ae%a4class)
      - [使用定义的参数](#%e4%bd%bf%e7%94%a8%e5%ae%9a%e4%b9%89%e7%9a%84%e5%8f%82%e6%95%b0)
      - [添加一个resolver](#%e6%b7%bb%e5%8a%a0%e4%b8%80%e4%b8%aaresolver)
      - [添加validator](#%e6%b7%bb%e5%8a%a0validator)
- [四、修改日志](#%e5%9b%9b%e4%bf%ae%e6%94%b9%e6%97%a5%e5%bf%97)

# 一、概述

## 1.1 什么是CLI

CLI（command-line interface）是在图形用户界面得到普及之前使用最为广泛的用户界面，它通常不支持鼠标，用户通过键盘输入指令，计算机接收到指令后，予以执行。也有人称之为字符用户界面。

_——摘抄自百度百科_

## 1.2 背景

需要一个基于`NEM-SDK`之上的**纯功能**工具，用命令行的形式实现所有用户可执行的功能。

## 1.3 开发信息

|信息|版本|
|-|-|
|CLI|0.17.1|
|Node|12.15.0|
|TypeScript|3.7.5|


# 二、项目结构

## 2.1 文件结构

- 根目录

|文件夹名|说明|
|-|-|
|bin|当前项目执行脚本。如果想测试当前项目先到当前文件夹下，执行`node nem2-cli`即可|
|build|构建后存放JS文件的目录。仅在执行了`tsc`命令之后会自动生成该文件夹|
|licenses|存放各个库证书的目录。如果添加了新的第三方库，要将第三方库的证书以txt格式存储于该文件夹下。文件名格式为`LICENSE-*.txt`。例如引入了`prompts`第三方库，则文件名为`LICENSE-prompts.txt`|
|src|存放编写的代码文件夹。（下面会对改文件夹进行详细的讲解）|
|test|测试用例|

- src目录

|文件（夹）名|说明|
|-|-|
|commands|命令文件|
|interface|接口层。目前存放抽象出来的命令行基类。|
|models|model层。主要用于存放profile和table的model类。|
|resolvers|存放各类resolver。|
|respositories|存放关于本地存储（主要以文件形式）的方法封装。|
|services|存放服务逻辑代码。如某些命令需要进行逻辑判断或者较为繁杂的逻辑操作，可以把对应的逻辑代码封装到该service层中。|
|validators|存放验证器。验证器是对用户输入内容进行验证的方法。|
|views|存放封装的table方法。在需要显示table（表格）格式的地方，可以将构造table的相关代码存放于此。|
|cli.ts|项目的入口文件。|
|options-resolver.ts|resolver的顶层基类，主要是对用户交互模块的再封装。|

> **提示** 如何查看对应的命令代码在哪？
> 
> 命令的逻辑层级与命令文件的物理层级一致，前提是都要在commands文件夹下。例如`profile create`命令文件的路径就是：`commands/profile/create.ts`

> **警告** 命令文件不能为`default`。因为框架默认会将`default.ts`解释为当前命令的默认文件。为了保证命令调用出现意料之外的问题，务必避免使用`default`作为命令文件名。

## 2.2 逻辑结构

从逻辑上看项目主要分为以下几部分：

- 命令
- resolvers
- validators
- model层
- table渲染
- 接口层

#### 命令

命令是逻辑代码的入口部分，命令主要包含与用户的交互——接收用户的输入和展示给用户信息。

一般来说命令部分可以处理简单的命令逻辑，如果该命令逻辑过于复杂，则建议将逻辑部分分离到service层中。

#### resolvers

用户可以在调用命令时传入对应参数，例如在使用`profile create`命令时有两种方式传入对应的参数：

- 直接传参：

```bash
node nem2-cli profile create -u http://127.0.0.1:3000
```

这里直接使用`-u http://127.0.0.1:3000`传入参数url，值为`http://127.0.0.1:3000`。

- 等待系统提示必要参数

```bash
node nem2-cli profile create
```

如果不传入参数依然可以开始该命令，但是如果某些必须的参数没有时，系统将会与用户交互，提示用户输入该参数，此时再输入同样是可以的。

> **提示** 如何查看某一命令支持哪些参数？
>
> 查看某一命令支持哪些操作可以使用**-h**标记该命令，此时将不会执行命令，而是现实该命令下包含的参数。例如想要查看`profile create`命令支持什么参数，可以这样输入`node nem2-cli profile create -h`

言归正传，怎样在命令执行时与用户进行交互，提示用户需要传入一些参数？这就需要用到resolver。resolvers统一定义在`src/resolvers`文件夹下。resolver的基础都是`src/options-resolver.ts`中封装的方法，只是根据具体的业务需求再细化出一个具体的resolver而已。

#### validators

当与用户进行交互，用户输入了内容，对该内容进行验证的相关代码就属于validator。所以validators的调用基本都是在resolver中，resolver中接收用户的输入之后，调用validators对用户输入的内容进行验证。

#### model层

如果某些业务需要自己定义数据结构，例如贯彻整个CLI的profile概念，我们就可以自己定义一个profile类到model层。

> **提示**
>
> 大部分设计SDK的数据结构都在SDK中有定义，那些数据结构无需自己再进行二次定义，直接使用即可。此处model层主要是用于自己定义CLI业务中涉及的且与SDK无关的model。

#### table渲染

在宣布交易后，会打印该交易的各项信息。为了方便展示，便引入了table的方式对此类信息进行展示。

所以table渲染主要用于给用户展示某类数据集合。如果需要构造一个table，则相关代码需要放置在`src/views/`文件夹下。

#### 接口层

接口层(`src/interfaces`)虽说是定义的接口，但是目前主要用途还是将一些常用的命令、操作抽象出来，作为一个父类放在interface层中。

当然，如果项目中有一些频率不高，但却没有定义的数据类型也可以定义在interface层中。

# 三、开发实例

例如我们要开发一个`profile create`命令。首先，在`src/commands`文件夹下创建`profile`文件夹，再在`profile`文件夹下创建文件`create.ts`文件。完整的路径为`src/commands/profile/create.ts`。本节将以该命令为例详细描述该如何创建一个新命令。

## 3.1 写一个完整的command文件

一个基本的command的文件主要包含两部分：

- 命令项class（命令参数）
- 默认class（命令逻辑代码）

#### 命令项class
首先`命令项class`需要继承自一个基类，最原始的基类为框架自带的`Options`。则此部分代码应该是这样的：

- src/commands/profile/create.ts

```ts
// 引入依赖
import {Options} from 'clime'

// 继承父类
export class CommandOptions extends Options {

}
```

> **注意**
>
> 其中命令项class的名为`CommandOptions`是可以根据自身需要进行取名的，没有什么别的要求，但是一般来说command文件中的命令项class一般都为这个名字。

#### 添加参数

该`CommandOptions`的作用是给命令定义所需要的参数。定义参数的代码格式如下：

- src/commands/profile/create.ts

```ts
// 添加引入option
import {Options, option} from 'clime'

export class CommandOptions extends Options {
	// 定义一个装饰器
	@option({
		flag: 'n',  // 参数的简写。这样可以使用-n进行传参
		description: 'Profile name.',  // 参数的描述，在-h命令中可以看到
	})
	name: string  // 参数名以及类型
}
```

#### 默认class

框架规定每个命令都必须有一个默认class用于命令的逻辑编写。一个默认class的基本格式如下：

- src/commands/profile/create.ts

```ts
// 添加引入command、metadata、Command
import {Options, option, command, metadata, Command} from 'clime'

/* 命令项代码部分（省略） */

// 定义一个command装饰器
@command({
	description: 'Create a new profile',  // 这个描述是对该命令的描述，在-h命令中出现
})
export default class extends Command {
	@metadata
	execute() {
		// 命令逻辑部分
		console.log('hello world')
	}
}
```

保存，在项目根目录下执行命令`tsc`编译TS，切换到`./bin`目录下，运行命令`node nem2-cli profile create`。可以看到，打印出了`hello world`。说明我们的命令已经生效了。

#### 使用定义的参数

如果想使用我们在`CommandOptions`类中定义的参数，可以在默认类中的`execute()`方法中传入参数，参数类型为`CommandOptions`的类名。我们现在打印用户传入的name参数。相关代码：

- src/commands/profile/create.ts

```ts
import {Options, option, command, metadata, Command} from 'clime'

export class CommandOptions extends Options {
	@option({
		flag: 'n',
		description: 'Profile name.',
	})
	name: string
}

@command({
	description: 'Create a new profile',
})
export default class extends Command {
	@metadata
	execute(options: CommandOptions) {
		// 命令逻辑部分
		console.log('The profile name is ' + options.name)
	}
}
```

保存，`tsc`编译。

此时再运行命令，顺便传入参数name值为`testName`。

命令行输入如下命令:

```bash
node nem2-cli profile create -n testName
```

控制台打印结果：`The profile name is testName`。

#### 添加一个resolver

如果该name参数为必须参数，即用户即使忘了在使用命令时使用`-n ***`传入name的值，我们也要在之后的运行过程中询问用户输入该数据。

使用resolver可以帮助你快速的完成该操作。

首先，我们应该为该业务实现一个针对当前具体业务的resolver。

在`src/resolvers`文件夹下创建文件`profileName.resolver.ts`。

一个resolver主要就是自己定义的一个类，类中有一个`resolve()`方法（必须）实现对应的代码。`resolve()`方法必须包含以下几个参数：

|参数名|类型|可省|说明|
|-|-|-|-|
|options|CommandOptions|否|为对应命令文件中定义的命令项class的类名|
|secondSource|Profile|是|定义了一个备选数据，如果传入该参数则优先使用该数据|
|altText|string|是|定义了与用户交互显示的文字|
|altKey|string|是|定义了这个参数对应options中的哪个属性|

> **注意** options类型该如何传？
>
> 上面我们在`src/commands/profile/create.ts`文件中定义了一个命令项class为`class CommandOptions`，所以这里的options类型就为`CommandOptions`。如果我们在`create.ts`文件中定义的命令项class为`class OtherOptions`，则这里options的类型就为`OtherOptions`。

> **注意** altText和altKey有什么用？
>
> 因为resolver可能会在不同地方进行复用，所以为了提高复用率，这里允许用户自定义传入交互文字与具体的options属性名。

> **注意** altKey的值是以什么为依据
>
> altKey是上文我们在`create.ts`文件中定义的命令项class中的属性名。上面代码中`CommandOptions`类中具有属性name与当前resolver对应，所以这里的altKey就应该为`name`。

一个resolver的代码如下：

- src/resolvers/profileName.resolver.ts

```ts
// 引入依赖
import {Resolver} from './resolver'  // 引入resolver接口
import {CommandOptions} from '../commands/profile/create'  // 引入命令项class
import {OptionsResolver} from '../options-resolver'  // 引入封装了的OptionsResolver父类

// 这里可以继承接口，该接口定义了一个Resolver的规范必须要有一个resolve()方法
export class ProfileNameResolver implements Resolver {
	async resolve(options: CommandOptions, secondSource?: Profile, altText?: string, altKey?: string): Promise<string> {
		// 与用户交互，询问用户输入name属性值
		const resolution = await OptionsResolver(options,
            altKey ? altKey : 'name',
            () => undefined,
            altText ? altText : 'Enter the new profile name: ')
						
		// 返回用户输入的name属性值
		return resolution
	}
}
```

我们将这个写好的resolver加入到我们的代码中。在命令执行之后询问用户输入name属性值。

现在`src/commands/profile/create.ts`中的代码：

```ts
import {Options, option, command, metadata, Command} from 'clime'
import {ProfileNameResolver} from '../../resolvers/profileName.resolver.ts'  // 引入resolver

export class CommandOptions extends Options {
	@option({
		flag: 'n',
		description: 'Profile name.',
	})
	name: string
}

@command({
	description: 'Create a new profile',
})
export default class extends Command {
	@metadata
	// 此时execute()方法应该加上async关键字，因为接下来我们将调用async方法
	async execute(options: CommandOptions) {
		// 询问用户输入name属性值
		const name = await new ProfileNameResolver().resolve(options)
		// 打印用户输入的name值
		console.log('The profile name is ' + name)
	}
}
```

#### 添加validator

如果想要判断用户输入的name值是否符合规定，我们需要再添加一个validator。

首先，在`src/validators`文件夹下创建文件`profileName.validator.ts`

一个validator主要部分为：
- 一个自定义类，实现接口`Validator<T>`
- 类中包含主要方法validate

因为validator主要还是沿用框架给出的模板，所以validator基本上模式是一样的：

- 一个validator模板：

```ts
export class MyValidator implements Validator<T> {
	validate(value: T, context?: ValidationContext): void {
		/* validator代码实现部分，如果有错，直接throw一个ExpectedError实例即可 */
	}
}
```

这里我们要判断name值中是否有不合法的字符，则该validator应该这样写：

- src/validators/profileName.validator.ts

```ts
// 引入validator必须的依赖项
import {ExpectedError, ValidationContext, Validator} from 'clime'

export class ProfileNameValidator implements Validator<string> {
	validate(value: string, context?: ValidationContext): void {
		// 验证name值是否合法
		const regexp = /\s/g  // 匹配name值中是否出现空白符，如果出现则不合法
		if (value.match(regexp)) {
			throw new ExpectedError('Invalid name value')
		}
	}
}
```

定义好了一个validator，就可以在resolver收到用户输入之后进行调用，检查用户输入是否合法。

- src/resolvers/profileName.resolver.ts

```
import {Resolver} from './resolver'
import {CommandOptions} from '../commands/profile/create'
import {OptionsResolver} from '../options-resolver'
// 引入validator
import {ProfileNameValidator} from '../validators/profileName.validator.ts'

export class ProfileNameResolver implements Resolver {
	async resolve(options: CommandOptions, secondSource?: Profile, altText?: string, altKey?: string): Promise<string> {
		const resolution = await OptionsResolver(options,
            altKey ? altKey : 'name',
            () => undefined,
            altText ? altText : 'Enter the new profile name: ')
						
		// 使用validator检查用户输入是否合法
		try {
			new ProfileNameValidator().validate(resolution)
		} catch(err) {
			console.log('Error: ' + err)  // 打印错误信息
			process.exit()  // 如果输入错误则结束当前命令
		}
		return resolution
	}
}
```

至此，一个完整的命令就完成了。


# 四、修改日志

[修改日志](./CHANGELOG.md)



















