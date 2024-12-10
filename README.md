# Rust-Complier-Fuzzing--
Fuzzing project based on the rust compiler in the Undergraduate Innovation and Entrepreneurship Program
### 研究目的

Rust编程语言的受欢迎程度正在迅速增长。凭借创新的类型系统，Rust在编译时静态消除了一类内存相关的错误，这些错误是C和C++等不安全语言的顽疾。也正是由于这些安全性保证，Rust被用于关键基础设施的开发，目前像Linux内核、Windows内核等都已开始集成Rust。Rust的性能接近C++，并且有良好的并行性支持，在未来应用场景会逐渐从系统级编程拓展到更广泛的软件开发领域，在数据科学和机器学习领域已有了一些初步的开源库。

编译器不仅负责将代码翻译成机器语言，还在过程中应用各种优化技术，以提升程序的执行效率。可以说，编译器的质量直接决定了语言的性能表现。而编译器的稳定性决定了语言生态的成熟度。编译器的错误可能会导致程序在不同平台上表现不一致，甚至会导致不正确的程序行为。一个成熟的编译器能够为开发者提供信心，从而推动语言的普及。此外，编译器的优化和更新也直接影响语言的发展方向和生命周期。

rustc作为Rust编程语言的官方编译器，尽管它的发展稳步前进，但潜在的问题仍影响其在某些场景下的使用效果。因此，Rust编译器rustc的充分测试至关重要。



### Rust-twins

Rust是一种相对较新的编程语言，以内存安全和众多高级特性著称。近年来，Rust在系统软件中被广泛应用。因此，确保唯一的Rust编译器实现——rustc的可靠性和稳健性至关重要。然而，作为检测错误最有效的技术之一，编译器测试在生成足够多样化的有效Rust程序时面临困难，这是因为Rust具有严格的内存安全机制。此外，现有研究主要关注触发rustc崩溃错误的测试，却忽视了错误编译结果（即误编译）。在缺乏多个Rust编译器实现作为测试基准的情况下，检测误编译仍是一个难题。

本文提出了rust-twins，这是一种新颖且有效的自动化差分测试方法，旨在检测rustc的崩溃和误编译。我们设计了四种针对Rust的特定变异操作符，并为Rust适配了十四种通用变异操作符，以生成语法和语义上有效的Rust程序，从而触发rustc崩溃。此外，我们开发了一种宏化（macroize）方法，将普通Rust程序重写为具有等效行为但不同实现的双宏。我们还设计了一个评估组件，通过比较宏展开结果与简单的宏输入来检查等价性（这些双宏的作用是让代码在不同的实现方式下保持等价的输出，使得编译器可以对两种实现进行比较。如果编译器在处理这些等效实现时出现不同的行为，就可能存在编译器错误。）。最后，rust-twins尝试用大量复杂的输入扩展这两个宏，以检测差异。由于宏展开机制，差异的根本原因可能不仅来自宏展开部分，还可能来自任何其他实现错误的编译器代码。

我们在最新版本的rustc上评估了rust-twins。实验结果表明，经过24小时测试，rust-twins的总行覆盖率是最佳基准技术rustsmith的两倍，并且识别出了更多的崩溃和差异。总体上，rust-twins触发了10个rustc崩溃，生成的宏中有229个暴露了rustc的差异。我们分析并报告了12个此前未知的错误，其中8个已得到确认并修复。



**编译器差分测试**：差分测试是一种检测编译器错误的方法，涉及将相同的代码在不同的编译器版本或优化设置下编译、执行，并比较结果。在RustSmith中，差分测试用于生成符合Rust规则的随机程序，并在不同的Rust编译器或优化级别上运行，确保它们产生一致的输出。差异可能表明编译器的潜在错误。



#### Rustlantis: Randomized Differential Testing of the Rust Compiler https://dl.acm.org/doi/10.1145/3689780

编译器是所有计算机架构的核心。其中间端和后端充满了容易出错的细微代码。同时，编译器错误的后果可能非常严重。因此，开发能够提升对编译器正确性信心的技术，并帮助发现不可避免的错误非常重要。随机差分测试是一种有前景的技术，它是一种模糊测试方法，通过使用不同的编译器或不同的编译器设置执行相同程序来检测行为中的任何意外差异。我们提出了Rustlantis，这是第一个能够在官方Rust编译器中发现新正确性错误的Rust模糊测试工具。为避免处理Rust严格的类型和借用检查，Rustlantis直接生成MIR，即Rust编译器优化的核心中间表示。Rustlantis的程序生成策略结合了静态跟踪程序状态、模糊编译器的程序状态，以及诱导编译器错误优化的伪代码块。这使得我们能够识别Rust编译器中22个此前未知的错误，其中大部分已经得到修复。

### 差分测试

差分测试的核心思想是利用两个看似相似但实际不同的输入，通过编译器对这两个输入的处理结果进行比较，从而揭示编译器在处理这些输入时可能存在的缺陷或错误。

差分测试的有效性在于它能够揭示那些单一输入可能无法触发的深层次问题。例如，如果一个编译器在处理两个看似相似的程序时表现不同，这可能表明编译器在处理某些特定情况时存在逻辑错误或优化不当。通过这种方式，差分测试可以帮助开发者和研究人员发现那些难以通过传统测试方法发现的深层次问题。

### RustSmith

RustSmith 随机生成符合 Rust 语言语法和语义的程序代码，然后将这些代码输入给 Rust 编译器，检查编译结果。如果 Rust 编译器在处理这些程序时出现崩溃、不一致或非预期行为，RustSmith 就能帮助定位到编译器中的潜在问题。

**端到端测试**能力，即它能自动生成程序、编译、运行并检查输出的整个过程





### Rust安全性特性

1. **所有权系统（Ownership）**

​	•Rust 通过所有权规则管理内存，避免了手动内存管理。每个值都有唯一的所有者，当所有者超出作用域时，值会自动释放。

​	•编译器在编译时强制检查所有权规则，避免出现内存泄漏或悬空指针。

2. **借用（Borrowing）和生命周期（Lifetimes）**

​	•	允许变量借用数据的引用，分为可变借用（mutable borrow）和不可变借用（immutable borrow），且一个时间点上只能有一个可变借用或多个不可变借用。

​	•	通过生命周期标注（lifetimes），Rust 确保引用在其生命周期内是有效的，避免悬空引用。

​	3.	**编译时内存安全**

​	•	Rust 在编译阶段捕获绝大部分与内存相关的错误（如数据竞争、未初始化变量访问），极大地降低了运行时崩溃的风险。

​	4.	**线程安全**

​	•	Rust 的类型系统确保多线程编程中的数据一致性，防止数据竞争。例如，Send 和 Sync trait 明确规定了如何在线程之间传递或共享数据。
