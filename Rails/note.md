# Ruby on Rails Tutorial Note

## 模型的继承体系

User 继承自ApplicationRecord 类,而这个类继承自ActionRecord::Base 类，这样对象才能与数据库通讯，才能把数据库中的列看做Ruby中的属性。

## controller 命名

创建的控制器名称会是蛇底式的 `static_pages_controller.rb`, 我们一般使用驼峰式的方式创建 `rails generate controller StaticPages`，这只是一种约定，也可以使用蛇底式 `rails generate controller static_pages`

因为ruby类名使用驼峰式，但是ruby文件名一般使用蛇底式，所以rails生成器使用underscore方式把驼峰式转换成蛇底式

## 撤销操作

当需要更改控制器名字，得删除生成的文件。因为生成控制器的时候还会生成一些辅助文件，这些也是需要撤销的。我们使用`rails destroy`来完成撤销操作

	rails generate controller StaticPages home help
	rails destroy controller StaticPages home help

一般来说上面俩个命令是互相抵消的

迁移命令改变数据库状态：

	rails db:migrate
	
撤销迁移命令：

	rails db:rollback

回归到最开始的状态：

	rails db:roolback VERSION=0

## ApplicationController

	class StaticPagesController < ApplitionController
		def home
		end
	end

因为controller继承了 ApplitionController, 当访问/static_pages/home时，rails会在StaticPages中寻找home动作，然后执行改动作，再渲染相应的视图。

## 什么时候应该先写测试（摘抄）

* 与应用代码相比，测试代码特别简短，倾向先写测试
* 对想要实现的功能不是特别清楚，倾向于先写应用代码，然后再写测试，并改进实现方式
* 为安全相关的功能先编写测试
* 只要发现一个问题，** 就编写一个测试重现重问题 **，避免回归，然后再编写应用代码修改问题
* 尽量不为以后可能修改的代码编写测试（如html结构的细节）
* 重构之前要编写测试， 集中测试容易出错的代码

一般先编写控制器和模型测试，然后再编写集成测试。

## TDD

先编写一个失败测试，然后编写应用代码让测试通过，最后再根据需求重构代码。

> 遇红-变绿-重构

## MiniTest

在测试辅助文件 `test/test_helper.rb` 中引入minitest，可以让Rails应用的测试适时的编程红色和绿色。

	require 'minitest/reporters'
	Minitest::Reporters.use!

## 使用Guard 自动测试

为了避免每次写完代码都要切换到命令行然后手动测试，我们可以使用Guard自动测试。Guard会监测文件系统的变动，并只会运行这个文件中的测试。

* 在gemfile中加入 guard gem
* 初始化：bundle exec guard init
* 编辑生成的Guardfile文件，让Guard在集成测试和视图发生变化后运行正确的测试
* 运行 bundle exec guard 开启自动测试

## Model

控制器名是复数，模型名是单数：控制器是 Users，而模型是 User。

模型表示单个用户，而数据库表中存储了很多用户: 数据库表是 users, 而模型是 User

User <= ApplicationRecord <= ActiveRecord::Base

## 迁移文件

change 方法用来定义要对数据库进行什么操作。

使用create_table 方法来新建一个表。

使用rails db:migrate 命令来向上迁移

迁移都是可逆的，撤销上面的迁移可以使用

	rails db:rollback

这个命令会调用drop_table方法，将新建的users表删除，对应新建时的 create_table方法。

## model验证

验证不能为空,长度,唯一性

	validates :name, presence: true, length: { maximum: 50 }, uniqueness: { case_sensitive: true }

查看错误信息

	user.errors.full_messages

Active Record 中的唯一性验证无法保证数据库层也能实现唯一性

在model层建立的唯一性验证会在拥有一定访问量的时候，出现俩条email一样的数据。

解决方法是在数据库层也加上唯一性验证，为email建立索引，然后为索引添加唯一性验证。

## 索引

建立索引

	rails generate migration add_index_to_users_email

添加邮件唯一性迁移

	add_index :users, :email, unique: true

索引本身不保证唯一性，所以需要添加 unique: true

## 使用安全密码机制

使用Rails 中的一个方法基本就能实现了

	has_secure_password

使用了这个方法后，会自动添加以下的功能

* 在数据库中的password_digest 列存储安全的密码哈希值
* 获得一对虚拟属性，password 和 password_confirmation，而且在创建用户的时候会执行存在性检验和匹配验证
* 获得 authenticate方法，如果密码正确，返回相应的用户对象，否则反回false

使用 has_secure_password的唯一要求是对应的模型中需要有password_digest的属性。

新建迁移为User添加password_digest列

	rails generate migration add_password_digest_to_users password_digest:string

这里迁移的名字以_to_users 结尾，rails 会自动生成一个向users表添加列的迁移

has_secure_password 使用先进的bcrypt哈希算法密码摘要，我们这里引入bcrypt gem

	gem 'bcrypt', '3.1.11'

has_secure_password 本身会验证存在性，但是只会验证没有密码，不能监测到比如"     "这样无效的密码

所以我们在这基础上需要在modle中添加存在性验证

	validates :password, presence: true, length{ minimum: 6 }

