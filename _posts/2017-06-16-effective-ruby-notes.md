---
title: 《Effective Ruby》读书笔记
categories: programming
tags: Ruby
---

# 《Effective Ruby》读书笔记

* 在比较语句中应将`true`/`false`放在运算符的左边，保证调用的是`TrueClass#==`/`FalseClass#==`。若放在右边的话，调用的是比较的对象的方法，很可能其方法是被覆写过的。

  ```ruby
  class Bad
    def ==(other)
      true
    end
  end

  false == Bad.new #=> false
  Bad.new == false #=> true
  ```

* `=~`匹配出来的不是全局变量，而是局部变量（当前进程的当前方法），Ruby对全局变量的命名没有限制，任何字符都可以作为变量名，所以`$1`会让人以为是全局变量。

  ```ruby
  def extract_error(message)
    if message =~ /~ERROR:\s+(.+)$/
      $1
    else
      "no error"
    end
  end

  $1 #=> nil
  ```

* Ruby中的常量指的是大写字母打头的标识符，所以类名、模块名都是常量。而由于类、模块是可以随时改变的，故其实Ruby中的常量也是可以随时改变的，所以其表现更像全局变量。为了不让常量的值被意外改变，需要对常量进行冻结，而冻结分为三种。

   1. 直接冻结常量的引用

      ```ruby
      A = 'abc'
      def foo
        A.gsub!('a', 'b')
      end
      A #=> 'abc'
      foo
      A #=> 'bbc'

      ######

      A = 'abc'
      A.freeze
      def foo
        A.gsub!('a', 'b')
      end
      A #=> 'abc'
      foo #=> RuntimeError: can't modify frozen String
      A #=> 'abc'
      ```
    
      但这种方法不能防止直接赋值
    
      ```ruby
      A = 'abc'
      A.freeze
      A = 'bbc'
      A #=> 'bbc'
      ```

   2. 冻结定义常量的模块
  
      将常量包含在模块中，然后冻结该模块，可以避免上述赋值改变常量值的发生
  
      ```ruby
      module Foo
        A = 1
      end
      Foo.freeze
      
      Foo::A = 2 #=> RuntimeError: can't modify frozen Module
      Foo::A #=> 1
      ```

   3. 冻结数组及数组元素

      当数组正好是字符串数组，则需在冻结数组的同时冻结数组中的元素
  
      ```ruby
      module Defaults
        NETWORKS = ['192.168.1', '192.168.2']
      end
      def purge_unreachable(networks = Defaults::NETWORKS)
        networks.delete_if do |net|
          !ping(net + '.1')
        end
      end
      purge_unreachable #=> Defaults::NETWORKS值会被改变
      
      # 下面这样可以防止上述情况发生
      
      module Defaults
        NETWORKS = ['192.168.1', '192.168.2'].freeze
      end
      
      # 但防止不了下面

      def host_addresses(host, networks = Defaults::NETWORKS)
        networks.map { |net| net << ".#{host}" }
      end
      
      host_addresses #=> 会改变Defaults::NETWORKS数组中元素的值
      
      # 只有冻结数组引用的同时，冻结数组中所有元素才能阻止上述所有情况的发生，如下
      
      module Defaults
        NETWORKS = [
          "192.168.1", 
          "192.168.2"
        ].map!(&:freeze).freeze
      end
      ```

* `initialize`方法可以手动调用

  ```ruby
  class A
    def initialize
      @a = 1
    end
    
    def foobar
      @a = 2
      initialize
      p @a #=> 1
    end
  end
  ```
  
* `dup`/`clone`会调用钩子方法`initialize_copy`

* 在类中，如果要访问实例的setter方法，需在前面加上消息接受者，也就是`self`，不然仅仅只是定义了一个局部变量

  ```ruby
  class A
    attr_accessor :foo
    
    def initialize
      foo = 1 # 这里仅仅只是定义了一个局部变量，并没有改变@foo的值
      self.foo = 1 # 这里用self作为消息接受者，则调用了setter方法foo=
    end
  end
  ```
  
* 可以用Struct代替Hash，一般会把Struct::new的返回结果赋值给常量，使其能跟类一样使用；并且new方法可以接受一个块。

  ```ruby
  A = Struct.new(:a, :b) do
    def foobar
      a + b
    end
  end
    
  a = A.new(1, 2)
  a.a #=> 1
  a.b #=> 2
  a.foobar #=>3
  ```

* 定义一个包含在命名空间中的对象时，如果用如下语法，必须要其命名空间的模块已经定义好时才能用，不然会报错

  ```ruby
  class A::B
    def foo
      p 'bar'
    end
  end
  #=> NameError: uninitialized constant A
    
  #####
    
  # 以下两种都是对的
  # 1.
  module A
    class B
      def foo
        p 'bar'
      end
    end
  end
    
  # 2.
  module A
    #...
  end
    
  class A::B
    def foo
      p 'bar'
    end
  end
  ```

* `equal?`方法在比较两个对象时会比较其object_id，即当且仅当两个对象完全一样时才会返回true；`eql?`方法需要两个对象的值和类型完全一样，而不会隐式转换类型再比较；`==`方法则只需值一样就行

* Hash类比较对象的键时调用的是`eql?`方法，则当Hash对象元素的键是对象时，需要重载对象的该方法。但仅仅重载该方法也不行，还需重载其`hash`方法，因为当用对象作为Hash对象元素的键时，会调用该方法决定该对象应该存在数据结构的什么地方。

  ```ruby
  class Color
    def initialize(name)
      @name = name
    end
  end
    
  a = Color.new('pink')
  b = Color.new('pink')
  ### 键其实相同，但会出现两个元素
  { a => 'like', b => 'like' } #=> {#<Color @name="pink">=>"like", #<Color @name="pink">=>"like"}
    
  #####
    
  # 重载两方法，将其委派到@name属性上
  class Color
    attr_reader :name
    
    def initialize(name)
      @name = name
    end
    
    def hash
      name.hash
    end
    
    def eql?(other)
      name.eql?(other.name)
    end
  end
    
  a = Color.new('pink')
  b = Color.new('pink')
  ### 此时正确，只有一个元素
  { a => 'like', b => 'like' } #=> {#<Color @name="pink">=>"like"}

  ```
  
* case关键字实际上调用的是`===`操作符

  ```ruby
  case command
  when 'start' then start
  when /^cd\s+(.+)$/ then cd($1)
  when Numeric then timer(command)
  end
    
  ### 上述语句等价于
    
  if 'start' === command then start
  elsif /^cd\s+(.+)$/ === command then cd($1)
  elsif Numeric === command then timer(command)
  end
  ```
  
  如上，case关键字之后的表达式会作为`===`操作符的右操作数，而when后面的作为左操作数。这样就必须要注意一些类，他们在command位置上和在when后面的结果是不一样的，例如Regexp。
  
  ```ruby
  # 下面调用的是Regexp#===，即正则在when后，字符串在case后时，结果是符合预期的
  /a/ === 'a' #=> true
  # 而下面调用的是String#===，即正则在case后，字符串在when后，结果就不太一样了，因为===默认的实现是直接调用==方法，而'a' == /a/的结果也是false
  'a' === /a/ #=> false
  ```
  
  同样需要注意的还有类和模块
  
  ```ruby
  [1, 2, 3] === Array #=> false
  [1, 2, 3].is_a?(Array) #=> true
  # 类和模块对===方法的实现也不一样，其行为类似于is_a?方法，下面调用的就是Class#===(Array::===)
  Array === [1, 2, 3] #=> true
  ```

* 如果要手动创建一个单例类，不仅要private new方法，还要禁止dup/clone方法

  ```ruby
  class Singleton
    private_class_method :new, :dup, :clone

    def self.instance
      @@single ||= new
    end
  end
  ```

* 子类会共享父类中的类变量，而不是重新创建

  ```ruby
  # 接上例
  class Configuration < Singleton
  # ...
  end

  class Database < Singleton
  # ...
  end

  Configuration.instance #=> #<Configuration>
  Database.instalce #=> #<Configuration>
  ```

* `dup`和`clone`方法只会执行浅拷贝，所谓浅拷贝，是指只拷贝容器本身，而不拷贝容器内元素

  ```ruby
  a = ['a']
  b = a.dup << 'b'
  b.each { |x| x.sub!('a', 'c') }
  a #=> ['c']
  ```

  可以使用Marshal来完成深拷贝

  ```ruby
  a = ['a']
  b = Marshal.load(Marshal.dump(a))
  b.each { |x| x.sub!('a', 'c') }
  a #=> ['a']
  ```

  但Marshal会消耗较多的内存，且序列化和反序列化耗时，部分类还不能被序列化，例如IO和File，在序列化时会跑出TypeError异常

* 当集合作为函数参数时，是按引用传递的，也就是说，在函数中对参数的改变会映射到其本身

  ```ruby
  foo = ['foo', 'bar']
  def bar(foo)
    foo.delete_if { |f| f.size > 1 }
  end
  bar(foo)
  foo #=> []
  ```

* Kernel模块中有一个特殊的方法`Array`，可以将不同的对象转换成数组。如果传入的参数是数组的话，则简单的返回原数组；如果传入的参数能相应`to_a`方法，则调用；其他情况下则将参数封装成单一元素的数组返回

  ```ruby
  Array(['a', 'b']) #=> ['a', 'b']
  Array(nil) #=> []
  Array('foobar') #=> ['foobar']
  ```

  但永远不要将Hash对象传入该方法，因为Hash#to_a会生成一个元素为键值对数组的二维数组，这往往不

* `Array#include?`方法的性能很差，某些情况下可以用`Hash#include?`替换，需要用数组中的元素构造一个哈希。一般将数组元素作为键，true作为值，这样，所有元素都引用同一个不可变的全局变量。也可以用`Set`代替Hash，两者性能相当，因为Set类内部使用哈希存储元素的。

  ```ruby
  class Role
    def initialize(name, permissions)
      @name, @permissions = name, permissions
    end
    
    def can?(permission)
      @permissions.include?(permission)
    end
  end

  ### 等价于

  class Role
    def initialize(name, permissions)
      @name = name
      # 将数组构造成哈希
      @permissions = Hash[permissions.map { |p| [p, true] }]
    end
    
    def can?(permission)
      @permissions.include?(permission)
    end
  end

  ### 也可用Set代替Hash

  class Role
    def initialize(name, permissions)
      @name, @permissions = name, Set.new(permissions)
    end
    
    def can?(permission)
      @permissions.include?(permission)
    end
  end
  ```

  但是需要注意的是，因为是将数组元素作为键，就必须遵从[上述所说](#hash_anchor)

  还有需要注意的是，Set是无序列表，如果需要有序集合，则应该考虑用`SortedSet`

* reduce方法是万能的，但需要注意的是，总是要给累加器一个初值，且传递的块总是要返回一个累加器

  ```ruby
  Hash[array.map { |x| [x, true] }]
  # 或
  hash = {}
  array.each do |element|
    hash[element] = true
  end
  ### 等价于
  # 给累加器传递了一个空哈希作为初值
  array.reduce({}) do |hash, element|
    hash.update(element => true)
  end

  users.select { |u| u.age >= 21 }.map(&:name)
  ### 等价于
  # reduce版本的好处是不用创建和遍历两个数组
  users.reduce([]) do |names, user|
    names << user.name if user.age >= 21
    # 返回修改后的names作为下一次的累加器
    names
  end
  ```

* 在用`||=`给哈希赋值时可以考虑使用默认哈希值，用`Hash::new`方法，传入的参数即为默认值

  ```ruby
  def frequency(array)
    array.reduce({}) do |hash, element|
      hash[element] ||= 0
      hash[element] += 1
      hash
    end
  end
  ### 等价于
  def frequency(array)
    array.reduce(Hash.new(0)) do |hash, element|
      hash[element] += 1
      hash
    end
  end
  ```

  访问不存在的键时会返回默认值，但不会修改哈希对象

  ```ruby
  h = Hash.new(42)
  h[:missing_key] #=> 42
  h.keys
  h[:missing_key] += 1 #=> 43
  # 上述语句等价于 h[:missing_key] = h[:missing_key] + 1
  h.keys #=> [:missing_key]
  ```

  但需要注意的是，如果传入的默认值是一个可以修改的参数，则会出现问题

  ```ruby
  h = Hash.new([])
  h[:missing_key] #=> []
  # <<方法并没有改变哈希对象，而是改变了其默认值[]
  h[:missing_key] << 'foobar'
  h.keys #=> []
  h[:missing_key] #=> 'foobar'

  h = Hash.new([])
  # h[:weekdays] / h[:months] 在第一次调用的时候都引用了统一默认值对象
  h[:weekdays] = h[:weekdays] << 'foo'
  h[:months] = h[:months] << 'bar'
  h.keys #=> [:weekdays, :months]
  h[:weekdays] #=> ['foo', 'bar']
  h.default #=> ['foo', 'bar']
  ```

  给Hash::new传递一个块即可解决上述问题，既每次调用不存在的键时都会执行块，并将其返回值作为默认值

  ```ruby
  h = Hash.new { [] }
  h[:weekdays] = h[:weekdays] << 'foo'
  h[:months] = h[:months] << 'bar'
  h[:weekdays] #=> ['foo']
  ```

  Hash::new方法的块可以选择性地接受两个参数，哈希本身和将要访问的键

  ```ruby
  h = Hash.new { |h, k| h[k] = [] }
  h.keys #=> []
  h[:foo] #=> []
  h.keys #=> [:foo]
  ```

* 我们重构代码时很可能碰到如下代码

  ```ruby
  if hash[key]
  # ...
  end
  ```

  这个依赖于哈希在访问不存在的键时默认返回nil，而当这种情况下，永远不要给该哈希赋予默认值，因为这样会导致逻辑跟原来不一致。而正是有这种可能性，所以应该用`has_key?`来检查一个键是否存在，而不是`Hash#[]`

* 相比用默认值，可以使用`Hash#fetch`方法，该方法接受可选的第二个参数，当访问的键不存在时则返回第二个参数的值，类似于默认值但不是全局赋予。且在第二个参数不存在时访问一个不存在的键，该方法丢出异常

  ```ruby
  h = {}
  h[:weekdays] = h.fetch(:weekdays, []) << 'Monday' #=> ['Monday']
  h.fetch(:missing_key) #=> KeyError ...
  ```

* 如果要实现一个类，和原生类有很大交集，应该使用委托而不是继承

  ```ruby
  # 继承的话会出现如下情况
  class LikeArray < Array; end
  x = LikeArray.new([1, 2, 3])
  y = x.reverse
  y.class #=> Array
  ```

  `Forwardable`模块实现了委托，配合其`def_delegators`等方法便能实现

  ```ruby
  require 'forwardable'

  class RaisingHash
    extend Forwardable
    include Enumerable
    
    def initialize
      @oth_hash = Hash.new do |hash, key|
        raise KeyError, "invalid key '#{key}'!"
      end
    end
    
    # def_delegators会将@hash中的方法映射到RaisingHash类的实例上
    def_delegators(:@hash, :[], :[]=, :delete, :each, :keys, :values, :length, :empty?, :has_key?)
    
    # 如果要自定义映射的方法名，可使用def_delegator
    def_delegator(:@hash, :delete, :erase!)
  end
  ```

  但是，并不是所有方法委托后的结果都是跟预想一致的。比如上述例子中，RaisingHash的实例在访问不存在的键时会抛出异常，但如果委托Hash#invert方法后，返回的实例则不会，因为其并不会调用initialize方法。这种情况则需要自己自定义方法

  ```ruby
    def invert
      other = self.class.new
      other.replace!(@hash.invert) # 此处将invert方法生成的新hash作为参数传入
      other
    end
    
    protected
    
    def replace!(hash)
      # default_proc方法返回传给Hash::new的块，此处将原对象的初始化块赋值给新的，然后将新的替换老的
      hash.default_proc = @hash.default_proc
      @hash.hash
    end
  ```

  需要注意的是，委托类的对象在调用`dup`/`clone`方法时，仅克隆对象，但对象的实例变量是指向同一个的，故需要重写initialize_copy方法，该方法在`dup`/`clone`方法调用时调用

  ```ruby
  h = RaisingHash.new # => #<RaisingHash:0x007fe3cc9ae928 @oth_hash={}>
  h2 = h.clone # => #<RaisingHash:0x007fe3cb92c780 @oth_hash={}>
  # 可以发现两者的实例变量引用的是同一对象
  h.oth_hash.object_id # => 70308183504000
  h2.oth_hash.object_id # => 70308183504000

  # initialize_copy仅接受一个参数，改参数为原对象。方法中对实例变量的引用默认接收者为原对象
  def initialize_copy(other)
    @hash = @hash.dup
  end
  ```

  还有就是需要注意在调用`freeze`/`taint`/`untaint`方法后，委托的方法是否还能改变其实例变量，如还能，则需重写`freeze`/`taint`/`untaint`方法，在处理对象本身的同时，处理其实例变量

  ```ruby
  def freeze
    @hash.freeze
    super
  end
  ```

  下面是完整的RaisingHash类

  ```ruby
  require 'forwardable'

  class RaisingHash
    extend Forwardable
    include Enumerable

    def initialize
      @oth_hash = Hash.new do |hash, key|
        raise KeyError, "invalid key '#{key}'!"
      end
    end

    attr_reader :oth_hash

    def_delegators(:@oth_hash, :[], :[]=, :delete, :each, :keys, :values, :length, :empty?, :has_key?)

    def invert
      other = self.class.new
      other.replace!(@oth_hash.invert)
      other
    end

    def initialize_copy(other)
      @oth_hash = @oth_hash.dup
    end

    def freeze
      @oth_hash.freeze
      super
    end

    protected

    def replace!(hash)
      hash.default_proc = @oth_hash.default_proc
      @oth_hash = hash
    end
  end
  ```

