【hanbin翻译中】

> 原文：[Working a Getting Started guide with STS](https://spring.io/guides/gs/sts/)
>
> 译者：[hanbin](http://github.com/hanbin)
>
> 校对：

# Working a Getting Started guide with STS

This guide walks you through using Spring Tool Suite (STS) to build one of the Getting Started guides.

## What you’ll build

You’ll pick a Spring guide and import it into Spring Tool Suite. Then you can read the guide, work on the code, and run the project.

## What you’ll need

- About 15 minutes
- [Spring Tool Suite (STS)](https://spring.io/tools/sts/all)
- [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later

## Installing STS

If you don’t have STS installed yet, visit the link up above. From there, you can download a copy for your platform. To install it simply unpack the downloaded archive.

When you’re done, go ahead and launch STS.

## Importing a Getting Started guide

With STS up and running, open the **Import Spring Getting Started Content** wizard from the **File** menu.

![Import Getting Started Content](https://spring.io/guides/gs/sts/images/1_open_wizard.png)

A pop-up wizard will offer you the chance to search and pick any of the published guides from the Spring website. You can either skim the list, or enter search words to instantly filter the options.]

> The criteria is applied to both the title and the description when offering instant search results. Wildcards are supported.


You can pick either [Maven](https://spring.io/guides/gs/maven) or [Gradle](https://spring.io/guides/gs/gradle) as the build system to use.

You can also decide whether to grab the **initial** code set, **complete** code set, or both. For most projects, the **initial** code set is an empty project, making it possible for you to copy-and-paste your way through a guide. The **complete** code set is all the code from the guide already entered. If you grab both, you can compare your work against the guide’s and see the differences.

Finally, you can have STS open a browser tab to the guide on the website. This will let you work through a guide without having to leave STS.

For the purpose of this guide, enter **rest** into the instant search box. Then pick [Consuming Rest](https://spring.io/guides/gs/consuming-rest). Pick **Maven** for building, and **initial** and **complete** code sets. Also opt to open the web page as shown below:

![Pick a guide](https://spring.io/guides/gs/sts/images/3_wizard.png)

STS will create two new projects in your workspace, import the [Consuming Rest](https://spring.io/guides/gs/consuming-rest) code base (both initial and complete), and open a browser tab inside STS as shown below:

![View the code and the guide](https://spring.io/guides/gs/sts/images/4_after-import.png)

From here, you can walk through the guide and navigate to the code files.

## Summary

Congratulations! You have setup Spring Tool Suite, imported the Consuming Rest getting started guide, and opened a browser tab to walk through it.

## See Also

The following guides may also be helpful:

- [Working a Getting Started guide with IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
