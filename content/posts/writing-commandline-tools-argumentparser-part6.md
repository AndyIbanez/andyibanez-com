---
title: "Writing Command Line Tools in Swift Using ArgumentParser, Part 6: Releasing And Installing Our Command Line Tool"
date: 2020-04-22T07:00:00-04:00
draft: false
originalDate: 2020-04-20T12:02:04-04:00
publishDate: 2020-04-22T07:00:00-04:00
highlightjslanguages:
 - swift
 - objectivec
tags:
 - swift
 - programming
 - apple
 - ArgumentParser
categories:
 - development
description: "Learn how to compile for release and install your ArgumentParser command line tools."
keywords:
 - swift
 - programming
 - apple
 - ArgumentParser
---

I wasn't sure if I should include this article as part of this series. But for the sake of completion, I decided to include it. This article is very short, but it tells us how to actually install our own tool in a system so we can start using it without writing its full path.

To recap, and before I end my series in Swift's `ArgumentParser`, let's give a quick overview of everything we have learned so far:

1. We learned the very basics of ArgumentParser, and we [learned about the basic building blocks](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part1/).
2. We learned how to [validate user input and deal with errors](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part2/).
3. We learned how to [organize our command line tool in subcommands](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part3/).
4. We learned how to customize our pages [to customize help](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part4/).
5. Finally, we learned how to [make use of asynchronous APIs within our tool](https://www.andyibanez.com/posts/writing-commandline-tools-argumentparser-part5/).

While `ArgumentParser` is not ready to be used in production, you can finally start using it for your own projects.

# Compiling for Release and Installation

## Compiling

To compile your command line tool, we need to use Xcode to create an Archive of it.

On Xcode, you can directly go to `Product` > and `Archive`. When the the project is done archiving, you will see the organizer window. Right click your project here and select "Show in Finder".

![Organizer](/img/organizer_window_app6)

![Show in Finder](/img/showinfinder_app6.png)

Right click the `xcarchive` file and select `Show Package Contents`.

![Show Package Contents](/img/packagecontentsshown_app6.png)

Your final binary is located inside the `Products > usr > local > bin > YOUR_BINARY`

## Installing

By default, Terminal programs a set of directories they search the command line tools in. Installing our command line tool is as easy as moving our binary to one of these default directories.

One such directory is actually seen above. UNIX systems have a `/usr/local/bin` directory where they keep their command line tools. This directory is part of the famous `$PATH` variable, which you have undoubtedly heard about. All the directories that are part of the `$PATH` are search paths for command line tools. When you execute a command line tool, such as `cat`, `vim`, `tail`, or others, the Terminal will look for their binaries in these folders.

So copy the binary you obtained from the previous section. Then in Finder, press `Cmd + Shift + G`. It will open a tiny window that lets you go to any directory in your system. Write `/usr/local/bin`, and press Enter.

Then simply copy your command line tool here.

Now we can use our command line from any Terminal window without having to specify its full path.

![/user/local/bin](/img/usrlocalbin+app6.png)

```
andyibanez@Andys-iMac / % MyCommandLineTool 1          
----------------------------------------------------------

INFO FOR POKÉMON: 1

ESPECIES: bulbasaur

----------------------------------------------------------
```

# Conclusion

Our Command-Line tools can be installed by simply pasting them in a directory specified in the user's `$PATH` variable. The user may configure some additional paths, but there's a few default ones we can use. Once our tool is there, our users can use our tools very naturally, without having to specify their full name.

<hr>

If you find any inaccuracies (and that includes typos) or problems in this article please tweet at me ([@AndyIbanezK](https://twitter.com/AndyIbanezK)) or send me an e-mail to andy[at]andyibanez[dot]com. Thank you for helping me improve the quality of my blog!

If there's anything related to Swift, iOS, or another Apple Platform you'd like me to cover, feel free to contact me and I will try to cover it in an upcoming article.


