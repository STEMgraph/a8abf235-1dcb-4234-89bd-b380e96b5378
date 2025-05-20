<!---
{
  "id": "a8abf235-1dcb-4234-89bd-b380e96b5378",
  "depends_on": ["abe720ad-32b4-4303-9b61-5875f592c05c"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-20",
  "keywords": ["CopperSpice", "GUI", "C++"]
}
--->

# A minimal CopperSpice Example

## Introduction
> Before you proceed with this tutorial, make sure, that the [CopperSpice Kitchensink Demo](https://github.com/STEMgraph/abe720ad-32b4-4303-9b61-5875f592c05c) was built correctly on your machine!

After setting up the demo, we are now doing the exact opposite: Setting up a minimal working example. 
Whenever it is time to work with a new library, language or framework, make sure to first build an already existing demo project; and second build your own minimal example. 

The heart of CopperSpice is the so called Event-System. 
It is the interface between the click- and keyboard-events.
On Linux Systems, the X11 library can be used to abstract this. 

Imagine your process running, doing what it can do until it runs out of jobs. 
It is now every GUIs time to wait for a next `click`, `button` or `timerout` event.
On the lowest basis of our process, a specific combination of instructions is used, to indicate to the kernel, that we would like to sleep until any of these events happen. 

In a very trivial way, this could look something like this:
```asm
.section .data
timeout:
    .quad 5         # seconds (tv_sec field of struct timeval)
    .quad 0         # microseconds (tv_usec field of struct timeval)

.section .text
.global _start
_start:
    # ----------------------------------------------------------
    # rdi = nfds (highest file descriptor + 1)
    # Since we are not watching any file descriptors,
    # we set nfds = 0 — the kernel will ignore read/write/except sets.
    # ----------------------------------------------------------
    mov $0, %rdi         # nfds = 0 (no file descriptors to monitor)

    # ----------------------------------------------------------
    # rsi = readfds
    # Set to NULL (0) since we're not watching for readability.
    # Normally, you'd pass a pointer to an fd_set structure.
    # ----------------------------------------------------------
    mov $0, %rsi         # readfds = NULL

    # ----------------------------------------------------------
    # rdx = writefds
    # Set to NULL (0) since we're not watching for writability.
    # ----------------------------------------------------------
    mov $0, %rdx         # writefds = NULL

    # ----------------------------------------------------------
    # r10 = exceptfds
    # Set to NULL (0) since we're not watching for exceptional conditions.
    # ----------------------------------------------------------
    mov $0, %r10         # exceptfds = NULL

    # ----------------------------------------------------------
    # r8 = timeout
    # Load the address of our timeout struct into r8.
    # This tells select() how long to wait.
    # lea (Load Effective Address) calculates the address of `timeout`
    # relative to the instruction pointer (RIP).
    # ----------------------------------------------------------
    lea timeout(%rip), %r8  # timeout = &timeout (5s)

    # ----------------------------------------------------------
    # rax = syscall number
    # The syscall number for select() is 23 on x86-64 Linux.
    # This makes the syscall with all arguments set up above.
    # ----------------------------------------------------------
    mov $23, %rax        # syscall number for select()
    syscall              # invoke the kernel

    # ----------------------------------------------------------
    # Once the select() syscall returns (after timeout),
    # we'll just exit the process cleanly with exit(0).
    # ----------------------------------------------------------
    mov $60, %rax        # syscall number for exit()
    xor %rdi, %rdi       # exit code = 0
    syscall              # exit(0)
```
If you are confused by the syntax, go back to the [basic GAS exercises](https://github.com/orgs/STEMgraph/repositories?q=GAS)

Once we know which events we want to listen to, we can decide which function to run. 
This process builds on the `sends + blocks` idea that was first employed in the **smalltalk** language in the 1970s. 
The **Qt**-Framework (pronounced: "cute") refactored this idea to a mechanic called `signals & slots`. 
Where signals are events, emitted by an object and slots are triggers of methods of other objects. 
CopperSpice, since being a fork of Qt, uses the same concept.
Which means we need to tell the CopperSpice Event-System which signal shall call which method. 
This is done by `connect`ing them:
```cpp
connect(sender, &SenderClass::signalName,
        receiver, &ReceiverClass::slotName);
```

In this exercise we will connect the `::clicked`-signal of a push-button with the `::close` slot of the applications main-window object. 

### Further Readings and Other Sources
- [Video-Tutorial to this Exercise](https://youtu.be/OhyylLAHQHc?si=rKLNYyRgMCm_KtVj&utm_source=STEMgraph)
- [The CopperSpice Event System](https://www.copperspice.com/docs/cs_api/event-system-c.html)

## Tasks

### Task 1: Create a minimal GUI project

Create a new folder and inside it a subfolder `./src`. Add the following `main.cpp`:

```cpp
#include <QtCore>
#include <QtGui>

int main(int argc, char *argv[])
{
   QApplication app(argc, argv);

   QWidget *mainWindow = new QWidget();
   mainWindow->setMinimumSize(700, 350);

   QPushButton *pb = new QPushButton();
   pb->setText("Close");

   QHBoxLayout *layout = new QHBoxLayout(mainWindow);
   layout->addWidget(pb);

   QObject::connect(pb, &QPushButton::clicked,
         mainWindow, &QWidget::close);

   mainWindow->show();

   return app.exec();
}
```

Explanation:

* Includes `QtCore` and `QtGui`, the essential CopperSpice headers.
* Initializes the QApplication which handles GUI control.
* Creates a QWidget as the main window and sets its minimum size.
* Adds a QPushButton labeled "Close".
* Uses a horizontal layout to manage the button.
* Connects the button's click signal to the close slot of the window.
* Displays the window and enters the application loop.

### Task 2: Create a `CMakeLists.txt`

> ⚠️ **Attention:**  
> Don't worry if the `CMakeLists.txt` script looks confusing — CMake is a complete language of its own.  
> This build configuration will work just fine for this and future examples, even if you don't understand every detail right now.  
> If you're curious or want to dive deeper later, check out the dedicated [CMake exercises](#), but it's not necessary at this stage.

Add the following `CMakeLists.txt` to your project root:

```cmake
cmake_minimum_required(VERSION 3.16)
project(PushButton)

include(CheckCXXCompilerFlag)
include(CheckCXXSourceCompiles)
include(CheckIncludeFile)
include(CheckIncludeFiles)

find_package(CopperSpice REQUIRED)

if(CMAKE_SYSTEM_NAME MATCHES "(Linux|OpenBSD|FreeBSD)")
  include(GNUInstallDirs)
  set(CMAKE_INSTALL_RPATH "\$ORIGIN")
elseif()
  message(FATAL_ERROR "Unsupported Operating System. Please use Linux or a compatible Unix variant.")
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_INCLUDE_DIRECTORIES_BEFORE ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

list(APPEND PROJECT_SOURCES
  ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
)

add_executable(${PROJECT_NAME} ${PROJECT_SOURCES})

target_link_libraries(${PROJECT_NAME}
  PRIVATE
  CopperSpice::CsCore
  CopperSpice::CsGui
)

install(TARGETS ${PROJECT_NAME} DESTINATION .)
cs_copy_library(CsCore)
cs_copy_library(CsGui)
cs_copy_plugins(CsGui)
```

From your project root, run the following command:

```sh
cmake -B ./build -DCMAKE_PREFIX_PATH=$CS_LIB_PREFIX
```

Ensure that the environment variable `CS_LIB_PREFIX` is correctly set. If you don't know what that means, revisit the Kitchensink demo setup steps.

### Task 3: Build and run the application

```sh
cmake --build ./build
cmake --install ./build --prefix ./deploy
./build/PushButton
```

## Questions

1. What role does the `QApplication` object play in a CopperSpice application?
2. How does the signal-slot mechanism work behind the scenes?
3. Why do we need to explicitly link `CsCore` and `CsGui`?
4. What would happen if you connect a signal to a slot that does not exist?
5. How can you verify whether a `connect()` call succeeded?

## Advice

You are now entering the realm of reactive, event-driven programming in C++. It's crucial to master these minimal examples before progressing to larger and more complex GUI systems. Take the time to fully understand each command, compile step, and signal-slot binding. If something breaks, break it down even further. Stick with `vim` and build automation with CMake for deeper insight into your toolchain. For additional practice, refer to [this signals and slots exercise](#). If you're struggling to understand the CMake flow or the environment variables, revisit the Kitchensink project setup — understanding your build system is half the battle.
