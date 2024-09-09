<a href="http://www.boost.org/LICENSE_1_0.txt" target="_blank">![Boost Licence](http://img.shields.io/badge/license-boost-blue.svg)</a>
<a href="https://github.com/boost-ext/di/releases" target="_blank">![Version](https://badge.fury.io/gh/boost-ext%2Fdi.svg)</a>
<a href="https://github.com/boost-ext/di/actions/workflows/build.yml" target="_blank">![Linux](https://github.com/boost-ext/di/actions/workflows/build.yml/badge.svg)</a>
<a href="https://codecov.io/gh/boost-ext/di" target="_blank">![Coveralls](https://codecov.io/gh/boost-ext/di/branch/cpp14/graph/badge.svg)</a>
<a href="http://github.com/boost-ext/di/issues" target="_blank">![Github Issues](https://img.shields.io/github/issues/boost-ext/di.svg)</a>
<a href="https://godbolt.org/z/5qTKhf">![Try it online](https://img.shields.io/badge/try%20it-online-blue.svg)</a>

---------------------------------------

# [Boost::ext].DI

Your C++14 **one header only** Dependency Injection library with no dependencies

<p align="center"><a href="https://www.youtube.com/watch?v=yVogS4NbL6U"><img src="doc/images/di_concept.png" alt="Dependency Injection"/></a></p>

> https://www.youtube.com/watch?v=yVogS4NbL6U

---

### Quick start

#### Download
> [Boost::ext].DI requires only one file. Get the latest header [here!](https://raw.githubusercontent.com/boost-ext/di/cpp14/include/boost/di.hpp)

#### Include
```cpp
#include <boost/di.hpp>
namespace di = boost::di;
```

#### Compile

* **GCC/Clang**
  ```sh
  $CXX -std=c++14 -O2 -fno-exceptions -fno-rtti -Wall -Werror -pedantic-errors file.cpp
  ```
* **MSVC**
  ```sh
  cl /std:c++14 /Ox /W3 file.cpp
  ```

#### Quick guide - Create object graph (创建对象图)
Boost.DI 库被设计为一个便于创建对象图的工具，而创建对象图是依赖注入设计模式的核心内容。

一个**对象图**是一个由各种对象实例和这些实例之间的构造器或方法引用所构成的图。在一个复杂的应用程序中，这些对象之间的关系也可能会非常复杂，这些关系可能会包括一些关键的运行时行为。

这就是`boost::di::create<T>()`方法的核心作用——创建这种类型的对象实例，并递归地解析和满足其所有的依赖，然后返回新创建的对象实例。

例如，当你调用 `auto object = injector.create<MyClass>()`，Boost.DI 库会：

1. 查看 `MyClass` 的构造函数需要哪些参数。
2. 如果这些参数类型也已经在注入器配置中定义，就会创建它们的实例，并且也会递归地解决这些类型的依赖。
3. 使用解决的依赖创建 `MyClass` 的实例。

重要的是，Boost.DI 会自动处理对象之间的依赖关系，并且能够调整对象的创建顺序以确保依赖的正确解析。

因此，Boost.DI 的 `create` 方法可以帮助我们创建和管理复杂的对象图，将我们从大量繁琐的手动依赖管理工作中解放出来。

```cpp
class ctor {
public:
  explicit ctor(int i) : i(i) {}
  int i;
};

struct aggregate {
  double d;
};

class example {
 public:
  example(aggregate a, const ctor& c) {
    assert(87.0 == a.d);
    assert(42 == c.i);
  };
};

int main() {
  const auto injector = di::make_injector(
    di::bind<int>.to(42),
    di::bind<double>.to(87.0)
  );

  injector.create<example>();
}
```
[Run this example on Wandbox](https://wandbox.org/permlink/Dpz6Isld1wxp3p5X).

<p align="center">
<table>
  <tr>
    <th></th>
    <th>Clang-3.8</th>
    <th>GCC-6</th>
    <th>MSVC-2015</th>
  </tr>

  <tr>
    <td>Compilation Time</td>
    <td>0.102s</td>
    <td>0.118s</td>
    <td>0.296s</td>
  </tr>

  <tr>
    <td>Binary size (stripped)</td>
    <td>6.2kb</td>
    <td>6.2kb</td>
    <td>105kb</td>
  </tr>

  <tr>
    <td>ASM x86-64</td>
    <td colspan="3">
      <pre><code>
xor eax, eax
retq
      </code></pre>
    </td>
  </tr>
</table>
</p>

#### Quick guide - Bind interfaces（绑定接口）
Boost.DI库允许你在创建注入器（injector）时定义一些**绑定规则**，这些规则告诉注入器当需要创建特定接口或类的实例时，应该怎样去创建它。

这是通过`di::bind<Interface>().to<Implementation>()`形式的语句来完成的。我们会将一个抽象类型（如接口或基类）绑定到一个具体的实现类型。这样，当注入器需要创建一个指定的抽象类型对象时，会创建绑定的实现类型的实例。

例如，假设我们有一个接口`ICar`和两个实现这个接口的类:`Sedan`和`Truck`。如果我们想在请求`ICar`时获取`Sedan`类的实例，我们可以构造一个注入器并绑定这个接口：

```cpp
auto injector = boost::di::make_injector(
    di::bind<ICar>().to<Sedan>()
);
```

随后，当我们使用`create()`方法请求一个`ICar`实例时，注入器会给我们一个`Sedan`的实例：

```cpp
auto car = injector.create<std::unique_ptr<ICar>>();
```

这种将接口或基类与其实现类型的绑定功能，非常重要，它使得在保持高层代码不变的情况下，可以随时更改或者调整底层的实现，这就是所谓的依赖反转原则。因此，绑定接口在依赖注入模式，甚至在整个面向对象编程中都起着关键的作用。

```cpp
struct interface {
  virtual ~interface() noexcept = default;
  virtual int get() const = 0;
};

class implementation : public interface {
public:
  int get() const override { return 42; }
};

struct example {
  example(std::shared_ptr<interface> i) {
    assert(42 == i->get());
  }
};

int main() {
  const auto injector = di::make_injector(
    di::bind<interface>.to<implementation>()
  );

  injector.create<std::unique_ptr<example>>();
}
```
[Run this example on Wandbox](https://wandbox.org/permlink/4yBTlH1jvdrsRPes).

<p align="center">
<table>
  <tr>
    <th></th>
    <th>Clang-3.8</th>
    <th>GCC-6</th>
    <th>MSVC-2015</th>
  </tr>

  <tr>
    <td>Compilation Time</td>
    <td>0.102s</td>
    <td>0.118s</td>
    <td>0.296s</td>
  </tr>

  <tr>
    <td>Binary size (stripped)</td>
    <td>6.2kb</td>
    <td>6.2kb</td>
    <td>105kb</td>
  </tr>

  <tr>
    <td>ASM x86-64 (same as `make_unique<example>`)</td>
    <td colspan="3">
      <pre><code>
push   %rbx
mov    %rdi,%rbx
mov    $0x8,%edi
callq  0x4008e0 <_Znwm@plt>
movq   $0x400c78,(%rax)
mov    %rax,(%rbx)
mov    %rbx,%rax
pop    %rbx
retq
      </code></pre>
    </td>
  </tr>
</table>
</p>

#### Quick guide - Bind templates
Boost.DI 也可以处理模板类型的绑定。当你有一组使用了模板的类型时，这种特性使得你能够更加灵活的处理这些类型之间的依赖关系。

假设你有一个模板类，如接口 `IProcessor<T>`，你可以用 Boost.DI 绑定不同的实现到模板的不同实例，如将 `StringProcessor` 绑定到 `IProcessor<std::string>`，将 `IntProcessor`  绑定到 `IProcessor<int>`：

```cpp
auto injector = di::make_injector(
    di::bind< IProcessor<std::string> >().to< StringProcessor >(),
    di::bind< IProcessor<int> >().to< IntProcessor >()
);
```

在这个例子中，当注入器需要一个 `IProcessor<std::string>` 或 `IProcessor<int>` 实例时，它就会创建一个 `StringProcessor` 或 `IntProcessor` 实例。

此外，Boost.DI 也支持自动绑定模板。这意味着，如果你没有显式的为一个模板实例定义绑定规则，Boost.DI 会尝试为该模板实例自动创建一个对象。这种行为可以通过提供特定的策略来定制。

总的来说，绑定模板的功能在许多情况下都相当有用，例如，当你正在处理一个有很多模板类的大型代码库，或者你想在运行时动态的决定使用哪个模板实例时。

```cpp
template<class ErrorPolicy = class TErrorPolicy>
class simple_updater {
public:
  void update() const {
    ErrorPolicy::on("update");
  }
};

template<class Updater = class TUpdater>
class example {
public:
  explicit example(const Updater& updater)
    : updater(updater)
  { }

  void update() {
    updater.update();
  }

private:
  const Updater& updater;
};

int main() {
  struct throw_policy {
    static void on(const std::string& str) {
      throw std::runtime_error(str);
    }
  };

  const auto injector = di::make_injector(
    di::bind<class TErrorPolicy>.to<throw_policy>(),
    di::bind<class TUpdater>.to<simple_updater>()
  );

  injector.create<example>().update();
  // Terminates with an uncaught exception because of our bound error policy
}
```
[Run this example on Wandbox](https://wandbox.org/permlink/mAzJQD8cTwboIxIx).

<p align="center">
<table>
  <tr>
    <th></th>
    <th>Clang-3.8</th>
    <th>GCC-6</th>
    <th>MSVC-2015</th>
  </tr>

  <tr>
    <td>Compilation Time</td>
    <td>0.102s</td>
    <td>0.118s</td>
    <td>0.296s</td>
  </tr>

  <tr>
    <td>Binary size (stripped)</td>
    <td>6.2kb</td>
    <td>6.2kb</td>
    <td>105kb</td>
  </tr>

  <tr>
    <td>ASM x86-64</td>
    <td colspan="3">
      <pre><code>
xor eax, eax
retq
      </code></pre>
    </td>
  </tr>
</table>
</p>

#### Quick guide - Bind concepts
在 Boost.DI 中，"binding concepts" 概念主要是绑定语法，用于把接口或抽象类和具体类关联起来。

不过，那样的表述涉及 C++ 的 Concepts 特性，这个特性主要用于约束模板参数的类型。而在 Boost.DI 库中，我们通常不显式地对“绑定概念（Concepts）”进行操作，这可能会导致一些混淆。

如果你的问题是关于在 Boost.DI 中如何操作类似于 C++ Concepts 这样的抽象概念的，那么应该注意 Boost.DI 的绑定语法同样适用于模板化的代码。例如：

```cpp
template <typename T>
di::injector<T> get_injector() {
    return di::make_injector(
        di::bind<Interface<T>>().to<Implementation<T>>()
    );
}
```

在上例中，我们创建了一个模板函数 `get_injector`，这个函数返回一个依赖注入器。这个注入器是根据类型 `T` 来配置的，将 `Interface<T>` 绑定到 `Implementation<T>`。因此，对于不同的 `T` 类型，我们可能获取到不同配置的注入器。

总结一下，Boost.DI 通过使用基于模板的绑定语法，提供了一种在模板代码中绑定类型的方式，这在某种程度上和 C++ 的 Concepts 特性有类似的效果。

```cpp
struct Streamable {
 template<class T>
 auto requires(T&& t) -> decltype(
   int( t.read() ),
   t.write(int)
 );
};

template<class Exchange = Streamable(class ExchangeStream)
         class Engine   = Streamable(class EngineStream)>
class example {
public:
  example(Exchange exchange, Engine engine)
    : exchange(std::move(exchange)), engine(std::move(engine))
  { }
  
private:
  Exchange exchange;
  Engine engine;
};

int main() {
  const auto injector = di::make_injector(
    di::bind<Streamable(class ExchangeStream)>.to<exchange>(),
    di::bind<Streamable(class EngineStream)>.to<engine>()
  );

  injector.create<example>();
}
```
[Run this example on Wandbox](https://wandbox.org/permlink/9QZTRsLkPNholmpj).

<p align="center">
<table>
  <tr>
    <th></th>
    <th>Clang-3.8</th>
    <th>GCC-6</th>
    <th>MSVC-2015</th>
  </tr>

  <tr>
    <td>Compilation Time</td>
    <td>0.102s</td>
    <td>0.118s</td>
    <td>0.296s</td>
  </tr>

  <tr>
    <td>Binary size (stripped)</td>
    <td>6.2kb</td>
    <td>6.2kb</td>
    <td>105kb</td>
  </tr>

  <tr>
    <td>ASM x86-64</td>
    <td colspan="3">
      <pre><code>
xor eax, eax
retq
      </code></pre>
    </td>
  </tr>
</table>
</p>

---------------------------------------

### Documentation

[](GENERATE_TOC_BEGIN)

* [Introduction](https://boost-ext.github.io/di/index.html)
    * [Do I use a Dependency Injection already?](https://boost-ext.github.io/di/index.html#do-i-use-a-dependency-injection-already)
    * [Do I use Dependency Injection correctly?](https://boost-ext.github.io/di/index.html#do-i-use-dependency-injection-correctly)
    * [Do I need a Dependency Injection?](https://boost-ext.github.io/di/index.html#do-i-need-a-dependency-injection)
    * [STUPID vs SOLID - "Clean Code" Uncle Bob](https://boost-ext.github.io/di/index.html#stupid-vs-solid-clean-code-uncle-bob)
    * [Do I need a DI Framework/Library?](https://boost-ext.github.io/di/index.html#do-i-need-a-di-frameworklibrary)
    * [Manual DI - Wiring Mess (Avoid it by using [Boost].DI)](https://boost-ext.github.io/di/index.html#manual-di-wiring-mess-avoid-it-by-using-boostdi)
    * [Real Life examples?](https://boost-ext.github.io/di/index.html#real-life-examples)
    * [Why [Boost].DI?](https://boost-ext.github.io/di/index.html#why-boostdi)
    * [[Boost].DI design goals](https://boost-ext.github.io/di/index.html#boostdi-design-goals)
    * [Articles](https://boost-ext.github.io/di/index.html#articles)
    * [Videos](https://boost-ext.github.io/di/index.html#videos)
    * [[Boost::ext].DI](https://boost-ext.github.io/di/index.html#boostextdi)
    * [Dependency Injection In General](https://boost-ext.github.io/di/index.html#dependency-injection-in-general)
    * [Acknowledgements](https://boost-ext.github.io/di/index.html#acknowledgements)
* [Overview](https://boost-ext.github.io/di/overview.html)
    * [Quick Start](https://boost-ext.github.io/di/overview.html#quick-start)
    * [Dependencies](https://boost-ext.github.io/di/overview.html#dependencies)
    * [Supported/Tested compilers](https://boost-ext.github.io/di/overview.html#supportedtested-compilers)
    * [Configuration](https://boost-ext.github.io/di/overview.html#configuration)
    * [Exception Safety](https://boost-ext.github.io/di/overview.html#exception-safety)
    * [Thread Safety](https://boost-ext.github.io/di/overview.html#thread-safety)
    * [Error Messages](https://boost-ext.github.io/di/overview.html#error-messages)
    * [Performance](https://boost-ext.github.io/di/overview.html#performance)
* [Tutorial](https://boost-ext.github.io/di/tutorial.html)
    * [0. [Pre] Refactor towards DI](https://boost-ext.github.io/di/tutorial.html#0-pre-refactor-towards-di)
    * [1. [Basic] Create objects tree](https://boost-ext.github.io/di/tutorial.html#1-basic-create-objects-tree)
    * [2. [Basic] First steps with bindings](https://boost-ext.github.io/di/tutorial.html#2-basic-first-steps-with-bindings)
    * [3. [Basic] Decide the life times](https://boost-ext.github.io/di/tutorial.html#3-basic-decide-the-life-times)
    * [4. [Basic] Annotations to the rescue](https://boost-ext.github.io/di/tutorial.html#4-basic-annotations-to-the-rescue)
    * [5. [Basic] Split your configuration](https://boost-ext.github.io/di/tutorial.html#5-basic-split-your-configuration)
    * [6. [Advanced] Dump/Limit your types](https://boost-ext.github.io/di/tutorial.html#6-advanced-dumplimit-your-types)
    * [7. [Advanced] Customize it](https://boost-ext.github.io/di/tutorial.html#7-advanced-customize-it)
    * [8. [Advanced] Extend it](https://boost-ext.github.io/di/tutorial.html#8-advanced-extend-it)
* [Try It Online!](https://boost-ext.github.io/di/try_it.html)
* [Benchmarks](https://boost-ext.github.io/di/benchmarks.html)
    * [Performance](https://boost-ext.github.io/di/benchmarks.html#performance)
    * [C++ Libraries](https://boost-ext.github.io/di/benchmarks.html#c-libraries)
    * [C++ vs Java vs CLibraries](https://boost-ext.github.io/di/benchmarks.html#c-vs-java-vs-clibraries)
    * [Usage of C++ vs Java vs CLibraries](https://boost-ext.github.io/di/benchmarks.html#usage-of-c-vs-java-vs-clibraries)
* [User Guide](https://boost-ext.github.io/di/user_guide.html)
    * [Injector](https://boost-ext.github.io/di/user_guide.html#injector)
    * [Bindings](https://boost-ext.github.io/di/user_guide.html#bindings)
    * [Injections](https://boost-ext.github.io/di/user_guide.html#injections)
    * [Annotations](https://boost-ext.github.io/di/user_guide.html#annotations)
    * [Scopes](https://boost-ext.github.io/di/user_guide.html#scopes)
    * [Modules](https://boost-ext.github.io/di/user_guide.html#modules)
    * [Providers](https://boost-ext.github.io/di/user_guide.html#providers)
    * [Policies](https://boost-ext.github.io/di/user_guide.html#policies)
    * [Concepts](https://boost-ext.github.io/di/user_guide.html#concepts)
    * [Configuration](https://boost-ext.github.io/di/user_guide.html#configuration)
* [Examples](https://boost-ext.github.io/di/examples.html)
    * [Hello World](https://boost-ext.github.io/di/examples.html#hello-world)
    * [Bindings](https://boost-ext.github.io/di/examples.html#bindings)
    * [Dynamic Bindings](https://boost-ext.github.io/di/examples.html#dynamic-bindings)
    * [Forward Bindings](https://boost-ext.github.io/di/examples.html#forward-bindings)
    * [Is Creatable](https://boost-ext.github.io/di/examples.html#is-creatable)
    * [Multiple Bindings](https://boost-ext.github.io/di/examples.html#multiple-bindings)
    * [Binding Non-owning Pointer](https://boost-ext.github.io/di/examples.html#binding-non-owning-pointer)
    * [Binding Templates](https://boost-ext.github.io/di/examples.html#binding-templates)
    * [Binding To Constructor](https://boost-ext.github.io/di/examples.html#binding-to-constructor)
    * [Automatic Injection](https://boost-ext.github.io/di/examples.html#automatic-injection)
    * [Constructor Signature](https://boost-ext.github.io/di/examples.html#constructor-signature)
    * [Constructor Injection](https://boost-ext.github.io/di/examples.html#constructor-injection)
    * [Multiple Interface](https://boost-ext.github.io/di/examples.html#multiple-interface)
    * [Annotations](https://boost-ext.github.io/di/examples.html#annotations)
    * [Deduce Scope](https://boost-ext.github.io/di/examples.html#deduce-scope)
    * [Custom Scope](https://boost-ext.github.io/di/examples.html#custom-scope)
    * [Eager Singletons](https://boost-ext.github.io/di/examples.html#eager-singletons)
    * [Modules](https://boost-ext.github.io/di/examples.html#modules)
    * [Modules (hpp/cpp)](https://boost-ext.github.io/di/examples.html#modules-hppcpp)
    * [Custom Policy](https://boost-ext.github.io/di/examples.html#custom-policy)
    * [Custom Provider](https://boost-ext.github.io/di/examples.html#custom-provider)
    * [Pool Provider](https://boost-ext.github.io/di/examples.html#pool-provider)
    * [Configuration](https://boost-ext.github.io/di/examples.html#configuration)
    * [Polymorphism](https://boost-ext.github.io/di/examples.html#polymorphism)
    * [Inheritance](https://boost-ext.github.io/di/examples.html#inheritance)
    * [Type Erasure](https://boost-ext.github.io/di/examples.html#type-erasure)
    * [Function](https://boost-ext.github.io/di/examples.html#function)
    * [Variant](https://boost-ext.github.io/di/examples.html#variant)
    * [Templates](https://boost-ext.github.io/di/examples.html#templates)
    * [Concepts](https://boost-ext.github.io/di/examples.html#concepts)
* [Extensions](https://boost-ext.github.io/di/extensions.html)
    * [Injector](https://boost-ext.github.io/di/extensions.html#injector)
    * [Constructor Bindings](https://boost-ext.github.io/di/extensions.html#constructor-bindings)
    * [Contextual Bindings](https://boost-ext.github.io/di/extensions.html#contextual-bindings)
    * [Assisted Injection](https://boost-ext.github.io/di/extensions.html#assisted-injection)
    * [Extensible Injector](https://boost-ext.github.io/di/extensions.html#extensible-injector)
    * [Concepts](https://boost-ext.github.io/di/extensions.html#concepts)
    * [Factory](https://boost-ext.github.io/di/extensions.html#factory)
    * [Shared Factory](https://boost-ext.github.io/di/extensions.html#shared-factory)
    * [Lazy](https://boost-ext.github.io/di/extensions.html#lazy)
    * [Named Parameters](https://boost-ext.github.io/di/extensions.html#named-parameters)
    * [XML Injection](https://boost-ext.github.io/di/extensions.html#xml-injection)
    * [Serialize](https://boost-ext.github.io/di/extensions.html#serialize)
    * [Types Dumper](https://boost-ext.github.io/di/extensions.html#types-dumper)
    * [UML Dumper](https://boost-ext.github.io/di/extensions.html#uml-dumper)
    * [Heap Provider](https://boost-ext.github.io/di/extensions.html#heap-provider)
    * [Mocks Provider](https://boost-ext.github.io/di/extensions.html#mocks-provider)
    * [Runtime Provider](https://boost-ext.github.io/di/extensions.html#runtime-provider)
    * [Scoped Scope](https://boost-ext.github.io/di/extensions.html#scoped-scope)
    * [Session Scope](https://boost-ext.github.io/di/extensions.html#session-scope)
    * [Shared Scope](https://boost-ext.github.io/di/extensions.html#shared-scope)
* [FAQ](https://boost-ext.github.io/di/FAQ.html)
* [CHANGELOG](https://boost-ext.github.io/di/CHANGELOG.html)
    * [[1.3.0] - 2023-05-23](https://boost-ext.github.io/di/CHANGELOG.html#130-2023-05-23)
    * [[1.2.0] - 2020-07-21](https://boost-ext.github.io/di/CHANGELOG.html#120-2020-07-21)
    * [[1.1.0] - 2019-01-07](https://boost-ext.github.io/di/CHANGELOG.html#110-2019-01-07)
    * [[1.0.2] - 2018-01-04](https://boost-ext.github.io/di/CHANGELOG.html#102-2018-01-04)
    * [[1.0.1] - 2016-05-04](https://boost-ext.github.io/di/CHANGELOG.html#101-2016-05-04)
    * [[1.0.0] - 2016-02-22](https://boost-ext.github.io/di/CHANGELOG.html#100-2016-02-22)
    * [[0.5.0] - 2015-01-12](https://boost-ext.github.io/di/CHANGELOG.html#050-2015-01-12)
    * [[0.1.0] - 2014-08-15](https://boost-ext.github.io/di/CHANGELOG.html#010-2014-08-15)
[](GENERATE_TOC_END)

---

**Disclaimer** `[Boost::ext].DI` is not an official Boost library.
