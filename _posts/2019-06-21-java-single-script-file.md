---
title: running a single Java (11) source code file as a script (shebang support)
excerpt: A new feature of Java 11 is the possibility to run a single Java source code file as a script.
toc: true
author: Eimert Vink
tags:
  - tech
---
This cool new feature allows to run a single java source file as a shell or python script, without the need to compile
the source code first. This opens up the possibility to run simple (single file) Java programs fairly easily. You'll need
an UNIX based operating system for this guide.

##1) Create a 'HelloWorld' file (**don't use .java as the extension**)
```java
#!/usr/bin/java --source 11

public class HelloWorld {
    public static void main(String... args) {
      System.out.println("Running Java 11 as script");
    }
}
```
##2) Make sure JDK 11 is installed. Java 11 does not have to be the systems default.
```bash
eimert@EIM ~ $ sudo update-alternatives --config java
Er zijn 5 beschikbare keuzemogelijkheden voor het alternatief java (dat voorziet in /usr/bin/java).

  Keuze        Pad                                             Prioriteit Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      automatische modus
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      handmatige modus
  2            /usr/lib/jvm/java-12-oracle/bin/java             1091      handmatige modus
  3            /usr/lib/jvm/java-8-openjdk-amd64/bin/java       1090      handmatige modus
  4            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      handmatige modus
  5            /usr/lib/jvm/java-9-openjdk-amd64/bin/java       1091      handmatige modus

Druk <enter> om de huidige keuze[*] te behouden, of typ een keuzenummer in: 
```

##3) Run the file!
Despite `#!/usr/bin/java --source 11` as the first line, simply running `java HelloWorld` does not work:
```bash
eimert@EIM ~ $ java HelloWorld 
Error: Could not find or load main class HelloWorld
Caused by: java.lang.ClassNotFoundException: HelloWorld
```
Java does compile and execute when specifying `--source 11`: `java --source 11 HelloWorld`:
```bash
eimert@EIM ~ $ java --source 11 HelloWorld 
Running Java 11 as script
```

The file executes with the default 0644 permissions:
```bash
eimert@EIM ~ $ ls -al HelloWorld 
-rw-r--r-- 1 eimert eimert 164 jun 21 10:24 HelloWorld
eimert@EIM ~ $ java --source 11 HelloWorld 
Running Java 11 as script
```

##Conclusion
Running 'Java as a script' gives developers the ability to easily run small utilities on a UNIX system. A single Java 
source script could be invoked using a (shell script) serverless function.

##References
[https://github.com/sandermak/java-next-labs](https://github.com/sandermak/java-next-labs){:target="_blank"}

