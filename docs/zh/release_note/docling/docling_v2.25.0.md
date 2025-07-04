# docling v2.25.0
### 为什么要使用docling

在当今信息爆炸的时代，如何高效地处理和理解文档成为了一个亟待解决的难题。docling应运而生，它不仅能帮助开发者快速生成和管理文档，还能通过智能化的方式提升文档的可读性和可用性。然而，许多用户在使用传统文档工具时，常常面临着格式不统一、信息冗余和更新困难等矛盾。docling的出现，正是为了打破这些局限，让文档管理变得更加轻松和高效。

### docling是什么

docling是一个开源工具，旨在帮助开发者生成、管理和维护文档。它通过自动化的方式，简化了文档编写和更新的过程，使得团队能够更专注于代码的开发，而不是文档的琐事。docling支持多种格式，能够与现有的开发流程无缝集成，提升团队的工作效率。

### 入门示例

想象一下，你是一名软件开发者，正在为一个新项目编写文档。使用docling，你只需在代码中添加注释，docling便会自动提取这些信息并生成结构化的文档。比如，在一个API项目中，你可以在每个函数上方添加描述，docling会将这些描述整理成API文档，省去了手动编写的繁琐步骤。通过这种方式，团队成员可以随时查看最新的文档，确保信息的一致性和准确性。

### docling v2.25.0版本更新了什么

docling v2.25.0版本引入了实验性的VLM管道，使用HF AutoModelForVision2Seq，推出了SmolDocling模型。CLI工具增加了下载所有模型的选项，并优化了帮助信息。此外，修复了VLM使用工件路径的问题，并将HTML中的div元素文本解析为TextItem。最后，扩展了文档中的分块说明，并新增了关于令牌限制的常见问题解答。

### 更新日志

#### 特性
- [实验性] 引入使用HF AutoModelForVision2Seq的VLM管道，推出SmolDocling模型。
- CLI工具增加了下载所有模型的选项，并优化了帮助信息。

#### 修复
- 修复了VLM使用工件路径的问题。
- 将HTML中的div元素文本解析为TextItem。

#### 文档
- 扩展了分块文档，新增了关于令牌限制的常见问题解答。

### 总结

在docling v2.25.0版本中，新增了实验性的VLM管道和CLI工具的改进，修复了一些关键问题，并扩展了文档内容。这些更新不仅提升了工具的功能性，也为用户提供了更好的使用体验。