# 如何书写一条Git提交信息

## 引言：为什么一条好的提交信息很重要
如果你随意浏览一个Git仓库的日志，你可能会发现它的提交信息或多或少是一团糟的。举个例子，可以看一下我早期向Spring项目提交的[这些美妙的东西](https://github.com/spring-projects/spring-framework/commits/e5f4b49?author=cbeams)：

```bash
$ git log --oneline -5 --author cbeams --before "Fri Mar 26 2009"
    
e5f4b49 Re-adding ConfigurationPostProcessorTests after its brief removal in r814. @Ignore-ing the testCglibClassesAreLoadedJustInTimeForEnhancement() method as it turns out this was one of the culprits in the recent build breakage. The classloader hacking causes subtle downstream effects, breaking unrelated tests. The test method is still useful, but should only be run on a manual basis to ensure CGLIB is not prematurely classloaded, and should not be run as part of the automated build.
2db0f12 fixed two build-breaking issues: + reverted ClassMetadataReadingVisitor to revision 794 + eliminated ConfigurationPostProcessorTests until further investigation determines why it causes downstream tests to fail (such as the seemingly unrelated ClassPathXmlApplicationContextTests)
147709f Tweaks to package-info.java files
22b25e0 Consolidated Util and MutableAnnotationUtils classes into existing AsmUtils
7f96f57 polishing

```

呀！和来自同一个仓库[近期的提交信息](https://github.com/spring-projects/spring-framework/commits/5ba3db?author=philwebb)相比：

```bash
$ git log --oneline -5 --author pwebb --before "Sat Aug 30 2014"

5ba3db6 Fix failing CompositePropertySourceTests
84564a0 Rework @PropertySource early parsing logic
e142fd1 Add tests for ImportSelector meta-data
887815f Update docbook dependency and generate epub
ac8326d Polish mockito usage
```

你更喜欢阅读哪一种捏？

前者在长度和格式均不一致；后者则简洁统一。一般情况下都会发生前者的情况，而后者绝不会偶然发生。

尽管有很多仓库日志看起来很像前者，但却是例外。[Linux kernel](https://github.com/torvalds/linux/commits/master) 和 [Git itself](https://github.com/git/git/commits/master)都是很好的例子。看一看Sprint Boot，或者任何一个由[Tim Pope](https://github.com/tpope/vim-pathogen/commits/master)管理的仓库。

这些仓库的贡献者深知，向其他共同开发者（甚至是未来的自己）传达有关变更的**_来龙去脉（context）_** 的最佳的方式是通过一条精心设计过的Git提交信息。`diff`会告诉你更改了**_什么（what）_** ，但只有提交信息能够恰当地告诉你**_为什么（why）_** 。Peter Hutterer [很好地阐释了这一点](http://who-t.blogspot.co.at/2009/12/on-commit-messages.html)：

> 重建一段代码的上下文是浪费的。由于我们无法完全避免它，所以我们应该尽最大努力去减少这种情况的发生。提交信息能够让我们做到这一点，并且，一条提交信息可以体现了开发者是否是一个好的协作者。
> Re-establishing the context of a piece of code is wasteful. We can’t avoid it completely, so our efforts should go to [reducing it](http://www.osnews.com/story/19266/WTFs_m) [as much] as possible. Commit messages can do exactly that and as a result, _a commit message shows whether a developer is a good collaborator_.

如果你没有过多地思考是什么构造了一条好的Git提交信息，你可能也没有花太多时间去使用`git log`以及相关工具。这里存在着一个恶性循环：由于提交历史是非结构化和不一致性的，所以人们不会花时间去利用和维护它；又因为提交历史没有被利用和维护，所以它仍然是非结构化的和不一致性的。

然而对日志精心照料是美妙而又有用的事情。这个过程中，`git blame`，`revert`，`rebase`，`log`，`shortlog`以及其他的子命令都会变得很有用。审查其他人的提交和合并请求变成一件值得做的事情，并且，忽然之间这件事就变得能够独立做了。理解几个月或者几年前某些事情为什么会发生比将变得有可能并且高效。

一个项目的长期成功取决于（排除其它因素）它的可维护性，一个维护者几乎没有拥有比他的项目日志更强大的工具（来进行项目维护）。花时间去学习如何恰当地管理一个日志是有意义的。这可能一开始非常麻烦，但之后就会成为一种习惯，并且最终会成为相关开发者成就感以及生产力的源泉。

在这篇博客中，我将仅仅阐释维护一个良好的提交历史的最基本要素：如何书写一条个人的提交信息。还有一些其它的重要实践例如压缩提交，我将不会在这里进行阐述。可能我会在后续的博客中介绍它们。

大多数编程语言对于什么是习惯风格都有着既定的约定，例如，命名、格式等等。当然，这些约定存在一些变体，但是大部分开发者赞同，选择并且坚持使用一种约定要比每个人都按自己方式而造成的混乱要好的多。

一个团队提交日志的方式应该都一致。为了创建一个有用的修订历史，首先团队应该就一个提交信息所定义的至少以下三件事达成一致：

**风格（Style）**：标记语法，换行边距，文法，大小写，标点符号。把这些东西说清楚，消除臆测，让一切变得尽可能简单。这样做的最终结果是，日志将会变得非常地一致，不仅让人们乐意去阅读它，实际上还会让人定期地去阅读它。
**内容（Content）**：提交信息的主体（如果存在）应该包含哪些信息？哪些信息不应该包含？
**元数据（Metadata）**：issue tracking IDs，pull request numbers等应该如何被引用？

幸运的是，已经有许多关于如何创建一个惯用的Git提交信息的既有规定。实际上，它们当中有许多是基于一些切确的Git命令功能作出假设的。不需要你重新再造任何轮子。只要遵循以下七条规则，你就能够走上提交的专业化道路。

## 关于一条好的Git提交信息的七条规则
> _Keep in mind: [This](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html) [has](https://www.git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project#_commit_guidelines) [all](https://github.com/torvalds/subsurface-for-dirk/blob/master/README.md#contributing) [been](http://who-t.blogspot.co.at/2009/12/on-commit-messages.html) [said](https://github.com/erlang/otp/wiki/writing-good-commit-messages) [before](https://github.com/spring-projects/spring-framework/blob/30bce7/CONTRIBUTING.md#format-commit-messages)._

1. [使用空行分隔标题和内容主体](#使用空行分隔标题和内容主体)
3. [标题第一个单词首字母大写](#标题第一个单词首字母大写)
4. [标题不要加句号](#标题不要加句号)
5. [标题使用祈使语气](#标题使用祈使语气)
6. [主体内容每行不超过72个字符](#主体内容每行不超过72个字符)
7. [用主体内容解释What，Why，而不是How](#用主体内容解释What，Why，而不是How)

模板：
```
Summarize changes in around 50 characters or less
关于变更的总结限定在50个字符以内

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. In some contexts, the first line is treated as the
subject of the commit and the rest of the text as the body. The
blank line separating the summary from the body is critical (unless
you omit the body entirely); various tools like `log`, `shortlog`
and `rebase` can get confused if you run the two together.
如果必要，对于解释更多细节的文本，每一行不得超过72个字符。在某些情况
下，第一行会被当作是提交的标题，剩下的文本则被视为是主体。用一个空行
将总结与主体内容分隔是十分关键的（除非你省略了主体）；如果你将两者合
并在一起，很多工具例如`log`，`shortlog`和`rebase`会混淆它们。

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).
Are there side effects or other unintuitive consequences of this
change? Here's the place to explain them.
解释这个提交所解决的问题。重点解释本次变更的动机，而不是描述更改
的过程（代码解释了怎么做）。在这个位置解释，本次更改是否有副作用
或者不直观的后果。

Further paragraphs come after blank lines.
如果还有更多的段落，则用空行隔开。

 - Bullet points are okay, too
 - 使用条目符号也是可以的。

 - Typically a hyphen or asterisk is used for the bullet, preceded
   by a single space, with blank lines in between, but conventions
   vary here
 - 通常用一个连接符或者一个星号开头作为条目符号，条目符号前面有一个
   空格，条目之间用一个空行隔开，不过这里有多种约定。

If you use an issue tracker, put references to them at the bottom,
like this:
如果你使用一个issue tracker，把对它们的引用放在底部，就像这样：

Resolves: #123
See also: #456, #789
```

### 1. 使用空行分隔标题和内容主体
以下内容来自 `git commit` [手册页](https://www.kernel.org/pub/software/scm/git/docs/git-commit.html#_discussion)：
> Though not required, it’s a good idea to begin the commit message with a single short (less than 50 character) line summarizing the change, followed by a blank line and then a more thorough description. The text up to the first blank line in a commit message is treated as the commit title, and that title is used throughout Git. For example, Git-format-patch(1) turns a commit into email, and it uses the title on the Subject line and the rest of the commit in the body.
> 虽然不是强制的，但最好还是在提交信息的开头用较短（不超过50个字符）的一行话来总结一下变更。接着紧跟一个空行，然后再进行更加详尽的描述。提交信息中，第一个空行上方的文本会被当作是提交的标题，并且这个标题会被整个Git使用。例如，Gitformat-patch(1)将一个提交转换成电子邮件，它会使用标题行的文本作为邮件的标题，将提交剩下的内容作为邮件的主体。

首先，不是每一个提交都需要标题和主体。有时候一行就够了，特别是当变更十分简单，不需要进一步解释时。例如：

```
Fix typo in introduction to user guide
```

不需要再过多解释了。如果读者想知道修正的错误是什么，她可以简单地浏览一下更改内容本身，例如，使用`git show`或者`git log`或者`git log -p`。

如果你的提交信息像这个例子一样，最简单的方法就是在命令行里使用带`-m`选项的`git commit`：

```
$ git commit -m"Fix typo in introduction to user guide"
```

但是，当一个提交值得给出解释以及背景时，你便需要书写主体内容。例如：

```
Derezz the master control program
    
MCP turned out to be evil and had become intent on world domination.
This commit throws Tron's disc into MCP (causing its deresolution)
and turns it back into a chess game.
```

使用`git commit -m`书写带有主体内容的提交信息并不是一件很容易的事情。你最好在一个恰当的编辑器里编辑提交信息。如果你还没有为Git在命令行下设置好一个编辑器，阅读[Pro Git的这一节内容](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)。

不管怎么说，标题与主体内容的分离对于阅读日志都是值得的。这里是一条完整的日志：

```
$ git log
commit 42e769bdf4894310333942ffc5a15151222a87be
Author: Kevin Flynn <kevin@flynnsarcade.com>
Date:   Fri Jan 01 00:00:00 1982 -0200

 Derezz the master control program

 MCP turned out to be evil and had become intent on world domination.
 This commit throws Tron's disc into MCP (causing its deresolution)
 and turns it back into a chess game.
```

现在使用命令`git log --oneline`，它只打印标题：

```
$ git log --oneline
42e769 Derezz the master control program
```

或者，使用命令`git shortlog`，它将提交信息以用户为单位进行分组，并且，为了简洁，该命令仅仅打印标题：

```
$ git shortlog
Kevin Flynn (1):
	  Derezz the master control program

Alan Bradley (1):
	  Introduce security program "Tron"

Ed Dillinger (3):
	  Rename chess program to "MCP"
	  Modify chess program
	  Upgrade chess program

Walter Gibbs (1):
	  Introduce protoype chess program
```

Git中还有很多其他的上下文关系，区分标题和主体内容对它们来说很重要——如果标题和主体之间没有用空行分隔，它们就不能正确工作。

### 2. 标题限定在50个字符以内
50个字符并不是一个硬性限制，而仅仅是一个经验法则。将标题限定在这个长度内可以保证它们的可读性，并且，这样可以迫使作者花点时间去思考如何以最简洁的方式解释发生了什么事情。

> _Tip: If you’re having a hard time summarizing, you might be committing too many changes at once. Strive for [atomic commits](https://www.freshconsulting.com/atomic-commits/) (a topic for a separate post)._
> _提示：如果发现总结很困难，你可能一次性提交了过多的更改。努力做到[atomic commits](If you’re having a hard time summarizing)（单独帖子的主题）把！

GIthub的UI完全明白这些规定，如果你超过了50个字符的限制，它会提醒你：

![gh1](https://i.imgur.com/zyBU2l6.png "gh1")

并且它会用一个省略号截断标题中超过72个字符的部分：

![gh2](https://i.imgur.com/27n9O8y.png)

所以，标题尽量控制在50个字符内，最多只能72个字符。

### 3. 标题第一个单词首字母大写
正如它听起来一样简单，标题以大写字母开头。

例如：

```
Accelerate to 88 miles per hour
```

而不要这样：

```
accelerate to 88 miles per hour
```

### 4. 标题不要加句号
标题末尾不需要加标点符号。当你尽可能控制标题在50个字符以内时，空间显得十分宝贵。

例如：

```
Open the pod bay doors
```

而不要这样：

```
Open the pod bay doors.
```

### 5. 标题使用祈使语气
祈使语气指的是“说话和书面表达时好像是在作出命令或者指令”。下面是一些例子：

+ Clean your room
+ Close the door
+ Take out the trash

你正在阅读的七条规则全部都是祈使句。

祈使语气听起来可能有点粗鲁，这也是为什么我们不经常使用它的原因。但用于Git提交的标题时却显得十分完美。其中的一个原因是，**当Git帮你生成一条提交时使用的正式祈使句**。

例如，使用`git merge`时创建的默认消息为：

```
Merge branch 'myfeature'
```

以及，当使用`git revert`时：

```
Revert "Add the thing with the stuff"
    
This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.
```

或者，当点击Github上的pull request按钮时：

```
Merge pull request #123 from someuser/somebranch
```

因此，当你用祈使语气编辑你的提交信息时，应遵循Git的内建约定。例如：

+ Refactor subsystem X for readability
+ Update getting started documentation
+ Remove deprecated methods
+ Release version 1.0.0

刚开始以这种方式书写可能有点不适应。通常，我们说活时使用的是陈述句，而它是用于陈述事实的。这也是为什么我们的提交信息经常用以下句式结束：

+ Fixed bug with Y
+ Changing behavior of X

除此之外，有时候一些提交信息用于描述他们内容：

+ More fixes for broken stuff
+ Sweet new API methods

为了消除你脑子里所有困惑，这里有一个简单的规则让你每次都能够写对：

如果一个Git提交信息标题的格式恰当，则它总能填到下面的句子里：

If applied, this commit will **_your subject line here_**

+ If applied, this commit will **_refactor subsystem X for readability_**
+ If applied, this commit will _update getting started documentation_**
+ If applied, this commit will **_remove deprecated methods_**
+ If applied, this commit will **_release version 1.0.0_**
+ If applied, this commit will **_merge pull request #123 from user/branch_**

注意，这个句子对于非祈使句来说就是语法错误的：

+ If applied, this commit will **_fixed bug with Y_**
+ If applied, this commit will **_changing behavior of X_**
+ If applied, this commit will **_more fixes for broken stuff_**
+ If applied, this commit will **_sweet new API methods_**

> **_请记住：只有在标题中使用祈使句是重要的。当你在书写你的主体内容时，你可以放宽这一限制。_**



### 6. 主体内容每行不超过72个字符
Git永远都不会自动换行。当你编辑提交信息的主体时，你一定要注意它的右边界，手动进行换行。

建议以72个字符为一行，这样Git可以有足够的空间来缩进文本并且还能在缩进后保持在80个字符以下。

在这方面，一个好的文本编辑器能够起到作用。例如，很容易配置Vim，使得你在编辑一条Git提交时，每72个字符就会进行换行。然而，通常IDE对文本智能换行这一点上做得很糟糕（虽然在最近的版本，IntelliJ IDEA关于这点做得更好了）。


### 7. 用主体内容解释What，Why，而不是How
关于用主体内容去解释What和Why，这个[来自于Bitcoin Core的提交](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6)是一个很好的范例：

```
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200

   Simplify serialize.h's exception handling

   Remove the 'state' and 'exceptmask' from serialize.h's stream
   implementations, as well as related methods.

   As exceptmask always included 'failbit', and setstate was always
   called with bits = failbit, all it did was immediately raise an
   exception. Get rid of those variables, and replace the setstate
   with direct exception throwing (which also removes some dead
   code).

   As a result, good() is never reached after a failure (there are
   only 2 calls, one of which is in tests), and can just be replaced
   by !eof().

   fail(), clear(n) and exceptions() are just never called. Delete
   them.
```

浏览一下[full diff](https://github.com/bitcoin/bitcoin/commit/eb0b56b19017ab5c16c745e6da39c53126924ed6)，想想现在作者在这里花时间解释来龙去脉，为协作者以及未来的提交者节省了多少时间。如果他不这么做，它可能会永远消失。

在大多数情况下，你可以省略关于如何更改的细节。代码在这一点上具通常具有自我解释的属性（如果代码过于复杂，就需要用注释来解释一样，这也就是注释的用处）。首要的事情是专注于解释清楚为什么你要作出更改——在更改之前事物是怎么工作的（这样工作会有什么问题），更改之后事物又是如何运作的，以及为什么你决定这么更改。

未来感谢你的维护者可能就是你自己！