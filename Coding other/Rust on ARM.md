https://www.hackster.io/locnnil/running-rust-on-arm-processor-281d22#:~:text=Rust%20has%20platform%20support%20for%20many%20target%20architectures%2C,type%3A%20%24%20rustc%20--print%20target-list%203.3%20Installing%20Linker

# Running Rust on ARM Processor

Cross-compiling Rust to NL668-LA board using OpenLinux. Prepare your environment, create, compile and run your first aplication.

[Intermediate](https://www.hackster.io/projects?difficulty=intermediate)Protip1 hour1,276

![Running Rust on ARM Processor](https://hackster.imgix.net/uploads/attachments/1544058/_7BfPBYJ2k1.blob?auto=compress%2Cformat&w=900&h=675&fit=min)

## [](https://www.hackster.io/locnnil/running-rust-on-arm-processor-281d22#things)Things used in this project

### Hardware components

![NL668-LA](https://hackster.imgix.net/uploads/attachments/1410752/20200615170755652.png?auto=compress%2Cformat&w=48&h=48&fit=fill&bg=ffffff)

[Fibocom NL668-LA](https://www.hackster.io/fibocom/products/nl668-la?ref=project-281d22)

×

1

[](https://www.fibocom.com/en/products/LTECat-NL668-LA.html "More info")

## [](https://www.hackster.io/locnnil/running-rust-on-arm-processor-281d22#story)Story

A simple "Hello World" Example of a Rust program running on Fibocom NL668-LA module. Covering the basics concepts of cross compilation on Rust language.

### 

1. Introduction

As explained in [this article](https://www.hackster.io/victorffs/nl668-la-openlinux-quickstart-c40508) It's possible to embedded your application on Fibocom's NL668 CAT4 module that runs a Linux based OS called OpenLinux. As you may now, Rust is becoming one of the most popular languages and promising to be the successor of C and C++ languages.

### 

2. Dependencies

-   NL668-LA Module (or any other Fibocom Module)
-   Rust
-   GCC
-   Linux Operating System (can be a virtual machine or WSL)
-   VSCode
-   ADB (Android Debug Bridge)

### 

3. Installing software dependencies

Let's stating installing all software requirements...

### 

3.1 Rust

- First It's necessary to have Rust and Cargo installed, if you don't, as explained in [rust official page](https://www.rust-lang.org/tools/install) it can be done running the command:

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Then after finished run:

```
$ rustc --version && cargo --version
```

If Rust was successfully installed the result would be something like:

![Printing  rustc and cargo version](https://hackster.imgix.net/uploads/attachments/1510612/screenshot_from_2022-10-17_23-35-36_JLKHiWZD3P.png?auto=compress%2Cformat&w=740&h=555&fit=max)

Printing rustc and cargo version

**Rustc** is the official Rust compiler and **Cargo** is the official Rust build tool and package manager.

### 

3.2 Target for Armv7l

Rust has platform support for many target architectures, one of them is the armv7-unknown-linux-gnueabi the platform for the NL668.

To install the target packages run:

```
$ rustup target add armv7-unknown-linux-gnueabi
```

note: to see all available architectures type:

```
$ rustc --print target-list
```

### 

3.3 Installing Linker

The Linker used for the mdm9607 processor in Rust is the same used for the `gcc-arm-linux-gnueabi` so let's install it! For Debian based distros run:

```
$ sudo apt install gcc-arm-linux-gnueabi
```

If it was installed successfully when running:

```
$ arm-linux-gnueabi-gcc --version
```

The result will be something like:

![Output of arm-gcc](https://hackster.imgix.net/uploads/attachments/1510613/screenshot_from_2022-10-18_00-03-41_NxRzaNjLWr.png?auto=compress%2Cformat&w=740&h=555&fit=max)

Output of arm-gcc

### 

3.4 ADB (Android Device Bridge)

ADB (Android Debug Interface) is a tools used for communicate with the module. In most cases ADB is used for Android developmet.

This tools is capable of send commands to device, upload files and provides access to the Linux shell.

### 

3.4.1 For Linux

For Install ADB in **Debian based** Linux distro run:

```
$ sudo apt-get update$ sudo apt-get install adb
```

To enable **driver**  **and**  **permission** follow the [section 3.4.3 of Victor's article](https://www.hackster.io/victorffs/nl668-la-openlinux-quickstart-c40508#toc-3-4-adb-and-fastboot--optional-if-not-installed-6).

### 

3.4.2 For (Windows WSL)

For windows WSL it's possible to install following as explained in this part of [Victor's article](https://www.hackster.io/victorffs/nl668-la-openlinux-quickstart-c40508#toc-3-4-adb-and-fastboot--optional-if-not-installed-6)

### 

3.4.3 Using ADB from WSL on Windows

If you don't like stay switching between Powershell and Bash, it's possible use the WSL directly from your bash terminal creating a symbolic link to your executable in windows directory, if you installed the ADB on `C:/platform-tools` folder the comand will be:

```
$ sudo ln -s /mnt/c/platform-tools/adb.exe /usr/bin/adb
```

WARN: Using this technique, it's not recommended the use of microcom to access the AT-Commands terminal (`/dev/smd7`) inside the ADB because the mobile network in AT commands may not work properly. (I'm speaking from experience).

### 

4. The Project

### 

4.1 Creating project

To start a new project run:

```
$ cargo new cross-arm
```

This command will create a new folder with a cargo project inside. Let's take a look.

```
$ cd cross-arm$ ls -a
```

![](https://hackster.imgix.net/uploads/attachments/1510759/hackester_01_t6nU4pWBbR.png?auto=compress%2Cformat&w=740&h=555&fit=max)

By default the `cargo new` command creates a file structure with some files and folders inside, let's understand they:

-   **.git** - By default cargo initializates a git repo for your project. If you're familiar with git you know that this folder is where git store changes in your project.
-   **.gitignore** - This folder have information to what files or directories should be igonored. By default it ignores only the target folder.
-   **Cargo.toml** - In this file you specify your dependencies and the [crates](https://crates.io/) that you may use latter in your project (Crates are like libraries in C/C++).
-   **Cargo.lock** - You should not edit this file, it's an autogenerated cargo file, using the information on cargo.toml that maps your dependencies.
-   **src** - It's where your main.rs file stay and your other source files that you may create will stay.
-   **target** - This folder is where your compiled binary file stays.

Inside the **src** folder by default, cargo already creates a **main.rs** with a "Hello World" example:

![](https://hackster.imgix.net/uploads/attachments/1510785/hackester_06_rvFXnVwJWu.png?auto=compress%2Cformat&w=740&h=555&fit=max)

### 

4.2 Installing target

For make a cross compile you have to install your desired target. In our case, we are going to install `armv7-unknown-linux-gnueabi:`

```
$ rustup target add armv7-unknown-linux-gnueabi
```

To verify if the target was correctly installed:

```
$ rustup target list --installed
```

![Installed targets](https://hackster.imgix.net/uploads/attachments/1510763/hackester_02_tbpD7YtSzv.png?auto=compress%2Cformat&w=740&h=555&fit=max)

Installed targets

### 

4.3 Specifing the linker

Now it's time to specify the linker for arm-gcc target. It can be done locally for only this project or make a globally config.

To specify locally, inside your project folder create a `.cargo` folder and inside it a `config.toml` file:

```
$ mkdir .cargo$ cd .cargo$ touch config.toml
```

Edit the config.toml file inserting:

```
[target.armv7-unknown-linux-gnueabi]linker = "arm-linux-gnueabi-gcc"
```

If you wish make this changes globals for any other project inset theses same two lines in `~/.cargo/config.toml` file. OBS: If it doesn't exist you can create it.

### 

5. Static Compiling

Another important thing is to make a static compile to your project don't deppend of any other shared libraries. It can be done adding the line in your .cargo/config.toml:

```
rustflags = ["-C", "target-feature=+crt-static"]
```

### 

6. Running

Now, compile your project runing:

```
$ cargo build --target armv7-unknown-linux-gnueabi
```

![building output](https://hackster.imgix.net/uploads/attachments/1510772/hackester_03_PAljOgGipp.png?auto=compress%2Cformat&w=740&h=555&fit=max)

building output

Your compilled binary will be inside `target/armv7-unknown-linux-gnueabi/debug/`

`Let's` push to NL668 and run:

```
$ cd target/armv7-unknown-linux-gnueabi/debug/$ adb push cross-arm /data$ adb shell# cd /data# chmod +x cross-arm# ./cross-arm
```

![Push to module using ADB](https://hackster.imgix.net/uploads/attachments/1510773/hackester_04_2wF2XODy8v.png?auto=compress%2Cformat&w=740&h=555&fit=max)

Push to module using ADB

![Inside ADB shell](https://hackster.imgix.net/uploads/attachments/1510774/hackester_05_bpjWrvYmJH.png?auto=compress%2Cformat&w=740&h=555&fit=max)

Inside ADB shell

And It's done! Congratulations, you have made your first Rust application on NL668 Fibocom module. To the next steps would be try some crate to access some other hardware resources, like serial ports and so on.

Have fun and goodbye!