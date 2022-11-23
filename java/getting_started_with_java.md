# Getting Started With Java

To build things using Java, you need the Java Development Kit (JDK). It is one of the three core technology packages used in Java programming. The other two are the Java Virtual Machine (JVM) and the Java Runtime Environment (JRE).

- The **JVM** is an abstract machine used to run Java programs as well as programs written in other languages that are also compiled to Java bytecode.
- The **JRE** is the on-disk part of Java that creates the JVM and loads programs into them.
- The **JDK** provides the tools necessary to write Java programs that can be executed and run by the JVM and JRE.

### Table of Contents

- [Download JDK](#download-jdk)
- [Install Your IDE](#install-your-ide)
- [Install Java In Your Ubuntu OS](#install-java-in-your-ubuntu-os)
- [Quickstart](#quickstart)


## Download JDK

I will show you how to install the open-source JDK (OpenJDK) from Amazon called the [Amazon Corretto](https://aws.amazon.com/corretto/?filtered-posts.sort-by=item.additionalFields.createdDate&filtered-posts.sort-order=desc). As of this writing, I will be showing you how to download [Amazon Corretto 17](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html).

![Amazon Corretto JDK](/images/java/getting_started/download_jdk.png)

You can choose to either use the terminal commands to download the JDK or you can click on the highlighted link to download directly from the site. Ensure that you download the file to a location you want, say in the home directory, for example.


## Install Your IDE

We will need to use an Integrated Development Environment when working with Java. You can choose whatever you prefer. I'd recommend you try out [IntelliJ IDE](https://www.jetbrains.com/idea/) or [VS Code](https://code.visualstudio.com/docs/languages/java). In this tutorial, I will be using VS Code for Java. 

- [Install VS Code](https://code.visualstudio.com/download) if you haven't already.
- Add java support by installing the [Extension pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack). 


## Install Java in Your Ubuntu OS

- To use the Corretto Apt repositories on Debian-based systems, such as Ubuntu, import the Corretto public key and then add the repository to the system list by using the following commands:

    ```java
    $  wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add - 
    sudo add-apt-repository 'deb https://apt.corretto.aws stable main'
    ```

- After the repo has been added, you can install Corretto 17 by running this command:

    ```java
    $  sudo apt-get update; sudo apt-get install -y java-17-amazon-corretto-jdk
    ```

- Before you install the JDK, install the java-common package.

    ```java
    $ sudo apt-get update && sudo apt-get install java-common
    ```

- Download the Linux `.deb` file (if you haven't already) from the [Downloads](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/downloads-list.html) page.

- Navigate to the location where the downloaded file is.

- Install the `.deb` file by using `sudo dpkg --install`.

    ```java
    $ sudo dpkg --install java-17-amazon-corretto-jdk_17.0.0.35-1_amd64.deb
    ```

- Verify the installation:

    ```java
    $ java -version


    // Output

    openjdk version "17.0.5" 2022-10-18 LTS
    OpenJDK Runtime Environment Corretto-17.0.5.8.1 (build 17.0.5+8-LTS)
    OpenJDK 64-Bit Server VM Corretto-17.0.5.8.1 (build 17.0.5+8-LTS, mixed mode, sharing)
    ```

For more installation details, you can refer to the [Corretto documentation](https://docs.aws.amazon.com/corretto/latest/corretto-17-ug/generic-linux-install.html).


## Quickstart

Let us create a new Java program as follows:

```java
// Create project folder
$ mkdir hello_world && cd hello_world

// Create java file
$ touch QuickStart.java

// Open project file in VS Code
$ code .
```

This will open `QuickStart.java` file in VS Code. Update the file with this code:

```java
// QuickStart.java

class QuickStart {
    public static void main (String[] args) {
        System.out.println("Hello World");
    }
}
```

To run the program, press `f5`. You  should see text "Hello World" displayed in the terminal.

When working with Java Projects, VS Code does not offer core support the way IntelliJ IDE or NetBeans do. You must have the necessary extensions installed to work with those project files.