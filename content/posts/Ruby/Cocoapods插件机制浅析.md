---
title: Cocoapods插件机制浅析
date: 2020-03-04  23:16:48
categories:
  - Ruby
tags: 
  - Cocoapods
---

> [原文链接](http://blog.samwei12.cn/2020/03/04/Ruby/Cocoapods%E6%8F%92%E4%BB%B6%E6%9C%BA%E5%88%B6%E6%B5%85%E6%9E%90/)

## 背景

虽然做iOS开发的过程中使用过 **[Cocoapods](https://cocoapods.org/)**， 但是对里面的细节了解其实不算太多，直到这两年做织女项目时，通过对Cocoapods进行Qt支持改造才开始深入了解部分细节，这个过程中，网上没有找到太多相关资料，本文就简单介绍下我对Cocoapods提供的插件机制的一个简单了解，希望能给大家带来一些帮助。

### Ruby Open Classes

在此之前，我们简单看下 `Ruby Open Classes` ，这部分是为未接触过Ruby的同学准备的，熟悉的同学可以直接略过。

在Ruby中，类永远是开放的，你总是可以将新的方法加入到已有的类中，除了你自己的代码中，还可以用在标准库和内置类中，这个特性被称为`Ruby Open Classes`。下面我们通过一个示例简单看下。

<!--more-->

首先，我们自定义一个类`Human`，放在`human.rb`文件中:
```ruby
class Human
    def greeting 
        puts "hello everybody"
    end
    
    def hungry 
        puts "I am hungry"
    end
end
```
接着，我们新增一个`main.rb`：
```ruby
require_relative 'human'

john = Human.new

john.greeting 
# hello everybody

john.hungry
# I am hungry
```
之后，我们在`main.rb`中重新定义`hungry`方法，
```ruby
class Human
    def hungry 
        puts "I could eat a horse"
    end
end

john.hungry
# I could eat a horse
```
可以看到，这里在我们新增`hungry`方法之后，所有的`Human`类的实例均调用我们的新实现了，即使是已经创建好的实例，这里故意放到两个文件中是想说明这个特性是可以跨文件甚至跨模块的，对Ruby内置方法的替换也是可以的（谨慎使用）
```ruby
puts "hello".size

class String 
    def size
        puts "goodbye"
    end
end

# 5
# goodbye
puts "hello".size
```

这个特性是十分强大的，让我们可以很容易的对三方模块进行扩展，也是`Cocoapods`的插件体系所依赖的基础。

## 流程分析

`Cocoapods`的插件体系整体流程还是比较清晰的，下面我们就来逐步看下。

### CLAide

首先，`Cocoapods` 提供了一个便捷的命令行工具库 [CLAide](https://github.com/CocoaPods/CLAide)，这个库包含很多功能，例如，一套命令基类，一套插件加载机制等。

### Command基类

Command基类在`lib/claide/command.rb`中，这里提供了大量基础功能，包括 `run` 、 `options`、 `help`等等。

首先，当我们每次执行 `pod xxx` 命令时候，会执行 `bin`目录下的可执行文件`pod`。

```ruby
require 'cocoapods'

if profile_filename = ENV['PROFILE']
    # 忽略不相关内容... 
else
    Pod::Command.run(ARGV)
end
```

这里实际上是 `Pod` 模块从`CLAide`继承了子类`Command < CLAide::Command`，我们执行Pod命令时候，就会调用

```ruby
def self.run(argv)
    help! 'You cannot run CocoaPods as root.' if Process.uid == 0

    verify_minimum_git_version!
    verify_xcode_license_approved!

    super(argv)
ensure
    UI.print_warnings
end
```

实际上只是扩展了一些检测git版本、xcode证书等，真正核心部分还是调用的`CLAide`的实现：

```ruby
def self.run(argv = [])
    plugin_prefixes.each do |plugin_prefix|
        PluginManager.load_plugins(plugin_prefix)
    end

    argv = ARGV.coerce(argv)
    command = parse(argv)
    ANSI.disabled = !command.ansi_output?
    unless command.handle_root_options(argv)
        command.validate!
        command.run
    end
rescue Object => exception
    handle_exception(command, exception)
end
```

可以看到这里真正执行命令之前会遍历所有的插件前缀，并进行插件加载，回过头来再查看 `cocoapods/command.rb` 会发现，这里指定了约定的插件前缀
```ruby
self.plugin_prefixes = %w(claide cocoapods)
```
可以看到这里的插件分为两种，我们目前只关心文件名为`cocoapods`前缀的插件。

### PluginManager

我们深入`PluginManager`的具体实现看下，

```ruby
def self.load_plugins(plugin_prefix)
    loaded_plugins[plugin_prefix] ||=
    plugin_gems_for_prefix(plugin_prefix).map do |spec, paths|
        spec if safe_activate_and_require(spec, paths)
    end.compact
end

def self.plugin_gems_for_prefix(prefix)
    glob = "#{prefix}_plugin#{Gem.suffix_pattern}"
    Gem::Specification.latest_specs(true).map do |spec|
        matches = spec.matches_for_glob(glob)
        [spec, matches] unless matches.empty?
    end.compact
end

def self.safe_activate_and_require(spec, paths)
    spec.activate
    paths.each { |path| require(path) }
    true
    # 不相关代码略去
    # ...
end
```

为了减小篇幅，这里只贴了核心相关代码，整体的流程大致是：

1. 调用`PluginManager.load_plugins`并传入插件前缀
2. `PluginManager.plugin_gems_for_prefix`对插件名进行处理，取出我们需要加载的文件，例如`cocoapods`前缀在这里会转换为所有包含`cocoapods_plugin.rb`的gem spec 信息及文件信息，例如`~/cocoapods-qt/lib/cocoapods_plugin.rb`
3. 调用`PluginManager.safe_activate_and_require` 进行对应的 gem spec 检验并对每个文件进行加载

至此，基本的插件加载流程大致梳理清楚了。

## 实操

下面我们看下如何自己扩展一个插件，关于这部分，Cocoapods其实也基本已经帮我们做了很多事情了，主要是 [cocoapods-plugins](https://github.com/CocoaPods/cocoapods-plugins)， 它提供了一个插件创建的完整生命周期，包括新增、发布、检索等。

### Cocoapods-plugins

执行 `pod plugins create cocoapods-test` 之后，发现自动帮我们创建了一个gem工程，其中的 lib 文件夹下果然存在了一个 `cocoapods_plugin.rb` 文件，整体的目录结构如下：

```bash
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── cocoapods-test.gemspec
├── lib
│   ├── cocoapods-test
│   │   ├── command
│   │   │   └── test.rb
│   │   ├── command.rb
│   │   └── gem_version.rb
│   ├── cocoapods-test.rb
│   └── **cocoapods_plugin.rb**
└── spec
    ├── command
    │   └── test_spec.rb
    └── spec_helper.rb
```

这里最核心的就是`cocoapods_plugin.rb` ，我们前面分析过，执行pod命令时候会主动加载所有`cocoapods_plugin.rb`文件，那么只要我们将需要扩展的类加到这里面，执行命令时候就会生效。

```ruby
class Test < Command
    self.summary = 'Short description of cocoapods-test.'

    self.description = <<-DESC
        Longer description of cocoapods-test.
    DESC

    self.arguments = 'NAME'

    def initialize(argv)
        @name = argv.shift_argument
        super
    end

    def validate!
        super
        help! 'A Pod name is required.' unless @name
    end

    def run
        UI.puts "Add your implementation for the cocoapods-test plugin in #{__FILE__}"
    end
end
```

可以看到这里其实只是新增了一个 `Test` 命令，并加了一些描述信息。为了让我们的扩展能生效，我们可以通过几种方式，

1. 本地gem源码依赖
2. 安装gem产物

为了更贴近实际生产发布流程，这里我们采用第二种方式。

首先，我们编译生成gem产物，

```bash
gem build cocoapods-test.gemspec
```

其次，本地安装

```bash
gem install ~/CocoapodsQt/cocoapods-test/cocoapods-test-0.0.1.gem  --local
```

此时，我们再执行 pod 命令

![](https://img.alicdn.com/tfs/TB1I3Ova8Cw3KVjSZR0XXbcUpXa-2116-1076.jpg)

可以看到我们扩展的命令已经生效，接下来就可以开始愉快的coding了。

## 结语

至此，我们对Cocoapods的整体插件流程应该有了一个比较清晰的认识了，希望能给大家带来一些帮助。
