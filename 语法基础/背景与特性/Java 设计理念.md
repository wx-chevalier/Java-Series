# Java 设计理念

如果不看一下 Java 的流行语，对 Java 历史的讨论就不完整。虽然促成 Java 发明的基本力量是可移植性和安全性，但其他因素在塑造 Java 语言的最终形态时也发挥了重要作用。Java 团队将这些关键的考虑因素总结为以下的流行语列表。

- Simple
- Secure
- Portable
- Object-oriented
- Robust
- Multithreaded
- Architecture-neutral
- Interpreted
- High performance
- Distributed
- Dynamic

其中有两个热词已经讨论过了：安全和便携。让我们来看看其他每一个都意味着什么。

## 简单

Java 被设计成易于专业程序员学习和有效使用。假设你有一定的编程经验，你会发现 Java 不难掌握。如果你已经了解面向对象编程的基本概念，学习 Java 将更加容易。最重要的是，如果你是一个有经验的 C++程序员，转到 Java 只需要很少的努力。因为 Java 继承了 C/C++的语法和 C++的许多面向对象的特性，所以大多数程序员学习 Java 几乎没有问题。

## 面向对象

虽然受到前辈的影响，但 Java 的设计并不是为了与任何其他语言的源代码兼容。这使得 Java 团队可以自由地在一片空白的基础上进行设计。这样做的一个结果是对对象采取了一种干净、可用、实用的方法。从过去几十年的许多开创性的对象软件环境中大量借鉴，Java 设法在纯粹主义者的 "一切都是对象 "模式和实用主义者的 "别挡我的路 "模式之间取得了平衡。Java 中的对象模型简单且易于扩展，而整数等原始类型则作为高性能的非对象被保留下来。

## 稳健

Web 的多平台环境对程序提出了非同寻常的要求，因为程序必须在各种系统中可靠地执行。因此，在 Java 的设计中，创建健壮程序的能力被赋予了很高的优先级。为了获得可靠性，Java 在一些关键领域进行了限制，迫使你在程序开发的早期就发现自己的错误。同时，Java 使你不必担心许多最常见的编程错误原因。因为 Java 是一种严格的类型化语言，它在编译时检查你的代码。然而，它也会在运行时检查您的代码。许多难以追踪的 bug 往往在难以重现的运行时情况下出现，在 Java 中根本不可能产生。知道你所写的东西在不同的条件下会以可预测的方式表现出来是 Java 的一个关键特性。

为了更好地理解 Java 是如何健壮的，请考虑程序失败的两个主要原因：内存管理错误和处理不当的特殊情况（即运行时错误）。在传统的编程环境中，内存管理是一项困难、繁琐的任务。例如，在 C/C++中，程序员经常会手动分配和释放动态内存。这有时会导致一些问题，因为程序员要么会忘记释放先前分配的内存，要么更糟的是，试图释放一些代码的另一部分仍在使用的内存。Java 通过为您管理内存分配和 deallocation，几乎消除了这些问题。事实上，deallocation 是完全自动的，因为 Java 为未使用的对象提供了垃圾回收）。传统环境中的异常情况经常出现在诸如除零或 "找不到文件 "的情况下，必须用笨拙而难读的结构来管理它们。Java 在这方面提供了面向对象的异常处理方法，提供了帮助。在一个编写良好的 Java 程序中，所有的运行时错误都可以而且应该由你的程序来管理。

## 多线程

Java 的设计是为了满足现实世界对创建交互式、网络化程序的要求。为了实现这一目标，Java 支持多线程编程，这使得您可以编写同时做许多事情的程序。Java 运行时系统带有一个优雅而又复杂的多进程同步解决方案，使您能够构造出运行流畅的交互式系统。Java 易于使用的多线程方法使您可以考虑程序的具体行为，而不是多任务子系统。

## 架构中立

对于 Java 设计者来说，一个核心问题是代码的持久性和可移植性。在 Java 诞生之时，程序员面临的主要问题之一是，无法保证如果你今天写了一个程序，它明天就能运行--即使是在同一台机器上。操作系统的升级、处理器的升级以及核心系统资源的变化，都会使一个程序出现故障。Java 设计者在 Java 语言和 Java 虚拟机中做出了几个艰难的决定，试图改变这种情况。他们的目标是 "写一次；在任何地方、任何时间、永远运行"。在很大程度上，这个目标已经实现了。

## 解释型和高性能

如前所述，Java 通过编译成称为 Java 字节码的中间表示法，实现了跨平台程序的创建。这种代码可以在任何实现 Java 虚拟机的系统上执行。之前大多数跨平台解决方案的尝试都是以牺牲性能为代价的。正如前面所解释的，Java 字节码经过精心设计，通过使用及时编译器，可以很容易地直接翻译成本地机器代码，以获得非常高的性能。提供这一功能的 Java 运行时系统不会失去任何平台独立代码的好处。

## 分布式

Java 是为 Internet 的分布式环境设计的，因为它能处理 TCP/IP 协议。事实上，使用 URL 访问资源与访问文件并无多大区别。Java 还支持远程方法调用（RMI）。该功能使程序能够跨网络调用方法。

## 动态

Java 程序携带了大量的运行时类型信息，这些信息用于验证和解决运行时对对象的访问。这使得以安全和方便的方式动态链接代码成为可能。这对于 Java 环境的健壮性是至关重要的，因为在 Java 环境中，字节码的小片段可能会在运行系统中动态更新。
