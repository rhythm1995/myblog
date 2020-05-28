---
title: 将项目从 GitHub 部署到服务器
tags:
  - linux
  - 运维
url: 21.html
id: 21
categories:
  - 扯淡集
date: 2016-01-23 23:22:35
---

原文转载自[开源中国翻译](http://www.oschina.net/translate/deploying-from-github-to-a-server)，一大半是我翻译的。

* * *

GitHub以及它所依赖的版本控制系统Git，绝对是非常出色的项目管理和协作的工具，不管项目是不是跟代码相关。 本文会讨论有哪些选项可以让Git和Github更好的融入项目的工作流当中，以实现平滑的自动化的过程。 我把这些选项划分到不同的工具集当中，这些集合包括自动运行测试，以及拉取代码部署到服务器上等等。

为何要这样做？
-------

有了这些自动化过程的运行，你和你的团队就可以只关注单纯的编码以及代码的合并，而不是每次build的时候都要花费几个小时去重复的做部署这样的事情。 自动化部署变化的主要问题是变化会自动地被部署。你必须信任你的团队以及他们写的代码。这就是为什么自动化部署和自动化测试的搭配成为典型，而下面提供的工具也反映了这一点。

Git Hooks（钩子）
-------------

Git内置了一套拓展框架叫做钩子（http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks）用来处理自动化部署，并且这些钩子一般在被特定的Git事件 ( certain points)触发后被调用在我们的第一端口用来处理任务。钩子可以被分为服务器端钩子与客户端钩子。 服务器端是用于监听网络操作的事件 ——比如，当存储库接收推送后。而客户端挂钩的触发是因为开发者进行了操作，如提交和合并。 这是在Git文档中hooks的完整列表(http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)。 我会注释一对情侣在这里让你开始。希望这能让你在自己当前手动部署的项目工作流程中中变得非常有用。Hooks可以在任何语言的项目部署中运行，强大而灵活。

pre-commit
----------

此这个钩子运行在其他所有钩子之前，并且在更改提交之前。可以用来在提交前检查代码错误。 我们在这里写一个JavaScript的小项目说明（当然，我故意留了你可以找到的bug）。 重命名hooks/pre-commit.sample 为 hooks/pre-commit，并进行如下测试命令，以这样的内容：

    &lt;br&gt;
    `#!/bin/shjshint index.js`&lt;br&gt;
    试着提交这个变动：
    `git commit -m &quot;adding Javascript 
    你可以看到报错信息：&lt;br&gt;
    `index.js: line 5, col 25, Missing semicolon.1 error`
    

添加缺少的分号后重新提交，不在报错。

post-receive
------------

当推送远程Git仓库完成时，服务器端的该钩子触发。在这个例子中，我们推出一个简单的网站的最新版本到你的Web服务器目录，实际上是一个（最基本的）部署。 我有一个现有的网站包含有一个index.html页 - 以及我们在后面的例子将使用的其他网页。你也可以创建自己的，使用在这里设立仓库。 克隆仓库，通过指定--bear标记来创建一个只包含版本控制信息的存储库，而不是我们的代码仓库：  

    git clone --bare https://github.com/sitepoint-editors/GitHub-Auto-Deploy.git GitHu`&lt;br&gt;
    现在我们添加钩子：
    `cd GitHub-Auto-Deploy.git/hooksvi post-receive`
    添加这些到文件中：&lt;
    `git clone --bare https://github.com/sitepoint-editors/GitHub-Auto-Deploy.git GitHub-Auto-Deploy.git`&lt;
    现在我们添加钩子：
    `cd GitHub-Auto-Deploy.git/hooksvi post-receive`&lt;
    添加这些到文件中：
    `#!/bin/shgit --work-tree=/var/www/html --git-dir=/var/repo/GitHub-Auto-Deploy.git checkout -f`&lt;
    

注意：这些路径是基于Ubuntu环境下完成，所以记得要改变路径，以满足你的路径。 该命令将推出当前仓库到定义的工作目录，但没有任何版本控制数据。 更改文件属性使之可执行： `chmod +x post-receive`<br> 小贴士：这些位置与Ubuntu的安装路径相关，所以一定记得要改变路径，以满足您的设置。该命令将检查当前的存储库到定义的工作目录，但没有任何版本控制数据。 将文件添加可执行的权限：<br> `chmod +x post-receive`<br> 在你的本地端，像平时一样克隆这个库，使用你选择的工具，并添加一个新的远程的实时服务器（记得更改服务器的详细信息到你的Web服务器和用户的详细信息）：<br> `git remote add prod ssh://user@domain.com/var/repo/GitHub-Auto-Deploy.git`<br> 要部署到我们生产环境下的服务器来替代仓库，输入以下命令： `git push prod master`<br> 你可以ls一下服务器的 var/www/html 目录，可以看到index.html文件已经被自动拷贝进你的web文件夹内啦。 如果你使用的是自己的Git仓库，你可以把它配置在同一台服务器上的应用，并实现自动化部署。如果你使用的是GitHub上或其他外部Git的服务，那么这个钩子还没有完全自动化，但它已经降到了一步。这可以进一步简化。 GitHub的post-receive 钩子中有一个可以使用reync或scp的选项。这是另外的一种选择——特别是当你的应用需要构建时（GitHub限制了可能的命令）——是使用post-receive 钩子来触发，然后使用-f选项可以检查出从GitHub的代码库的应用程序服务器上的脚本和运行其他一些必要的命令。这个时候，自动化部署开始变得复杂起来，我们不得不使用下一套工具来更好的完成。

从 GitHub 直接自动部署
---------------

GitHub 有它自己的文档来自动化部署到集成平台，这里包括一些托管提供商。 老实说，大部分文档都有些错误，不准确或者没有起到作用， 在一些主流的主机提供商那儿，我做了一些搜索链接到官方文档，对于其他一些提供商，我建议你使用 post-receiveor 持续集成的方法：<br> Heroku AWS Azure

持续集成(CI)服务
----------

有许多无数的能够查看 GitHub 项目回购变更协议的应用服务，不仅能够为你部署，而且能够执行其他功能，诸如为你运行测试和构建过程。 一旦你移动到一个新的和更复杂的实例时，我们可以使用 CI 自动化构建项目过程。首先，拉伸一个存储库的 Master 分支，然后触发一个运行构建的 bash 脚本，并且部署流程以及对微博更新。CI 与 web 服务能够在同一台服务器上或者在不同的服务器上运行，这一切都取决于你的偏好。

Jenkins
-------

你需要搭建你自己的 Jenkins 服务器，这意味着你可以完全地控制它，但必须要对它进行维护。幸运的是，它提供了多平台支持，如果你只是想要先简单尝试一下的话，这些支持也包括了 Docker。 Jenkins 使用插件实现了自己的大部分功能，并且由于其年代久远、开源的性质以及普及度很广，它拥有很多的插件。例如，有一些 Git、GitHub 和 Twitter 的相关插件。 Jenkins 需要大量的配置，而且有时，若想要将你需要的指令组合到一起来构造你所需的工作流程，可能需要大量的研究。

Travis
------

此外，在 GitHub 文档中，使用 GitHub 的 Travis 集成指令已经过时。现在，它更简单：阅读找出更多的 Travis 文档。 Travis 不需要任何主机与服务器设置，因此你无需投入太多的精力，就可以保持和试用CI，这是一个很好的起点。不过，扩展超出（综合）默认的集成将涉及到一些额外的配置工作。比如，微博请求对 webhooks 的访问。 在回购中，你会注意到 Travis-- 特别是在配置自己的文件中，它有一个习惯，就是更新太慢。当你本身没有对 Travis 服务器进行访问时，那么这些问题就难以解决。

其他商业服务
------

持续集成已经日益流行了，所以已经有了非常多的新的服务和应用程序 – 很多是通过你可能已经在使用的工具的创作者释出的，并且将很和谐的融入到现有的工具链和工作流程当中。这里有些例子： https://buddy.works/ https://www.atlassian.com/software/bamboo/ https://www.jetbrains.com/teamcity/ https://codeship.com/ https://circleci.com/ https://saucelabs.com/ https://about.gitlab.com/gitlab-ci/ http://deploybot.com/

* * *