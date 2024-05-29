<!--yml
category: 未分类
date: 2024-05-27 14:55:46
-->

# Sandboxing Python with Win32 App Isolation  - Windows Developer Blog

> 来源：[https://blogs.windows.com/windowsdeveloper/2024/03/06/sandboxing-python-with-win32-app-isolation/](https://blogs.windows.com/windowsdeveloper/2024/03/06/sandboxing-python-with-win32-app-isolation/)

Sandboxing Python can be very useful for developers but has been challenging due to the flexibility of CPython implementation. It is particularly useful in many scenarios to have sandboxed Python.  For example, on any website that needs to execute arbitrary code from users, like coding practice sites or online interpreters. It could also be useful to help prevent increasing attacks on LLMs (Large Language Models). 

Win32 App Isolation (https://github.com/microsoft/win32-app-isolation) lights up a different path to sandboxing Python – not on a Python library level, but on an application/OS level. 

Win32 App Isolation is designed to isolate Win32 application by creating a security boundary between the application and the OS, using the combination of AppContainer, virtualized resources and brokering file system, which helps to prevent the application from compromising the operating system. 

Following are the specifics requirements and guidance for sandboxing CPython with Win32 App Isolation on recent Windows Insider releases. It takes about 30-60 minutes and anyone with the insider version of Windows can do it on their local machine. 

To use Win32 App Isolation, you need the insider version of Windows with version number > 25357, and the MSIX Packaging Tool from https://github.com/microsoft/win32-app-isolation/releases/ 

Step One: 

Download the Python installer for Windows from python.org. 3.12.2 was used for this specific example. 

Step Two: 

Package it by following the documentation ([https://github.com/microsoft/win32-app-isolation/blob/main/docs/packaging/msix-packaging-tool.md](https://github.com/microsoft/win32-app-isolation/blob/main/docs/packaging/msix-packaging-tool.md)). You may need a certificate to sign the package to install it (https://learn.microsoft.com/en-us/windows/msix/packaging-tool/create-app-package#signing-preference). 

Add `isolatedWin32-PromptForAccess` capability for demonstration purposes, so you can allow it to access certain resources in prompts. Without the capability, all access to the user files that are not explicitly set to be available to the application would be denied. 

Add an alias `python3.12.exe` to use it from PowerShell. 

Step Three: 

Install the package. Use the defined alias `python3.12.exe` in PowerShell to bring up the Python interpreter. 

When starting the Python interpreter, it will prompt you to ask permission to the current working directory. Python itself would be brought up with either option – but this determines whether Python has access to the files in the current directory. 

For example, if you clicked “No” here, you would not be able to list the files in the current directory anymore: 

The selection will be remembered so the application will only prompt once for any file/folder. The user can reset the file permissions in Settings – Privacy & security – File system: 

The Python interpreter should work well on its own given no code has been changed.  

The more interesting aspects to check out are the “sandboxing” features – network and file system. 

But before that, try to execute a script using isolated Python.  

As you are trying to access and execute a user file (the script), it will prompt for the access. If denied, the script will not run: 

Allow it to review some examples, starting with network access. 

As you did not provide any network capabilities when packaging, Python will not have any access to the network. Confirm this by running this script: 

```
def test_urllib(): 
    with urllib.request.urlopen("http://www.microsoft.com") as response: 
        print(response.read())
```

Also try to access the file system. 

```
def test_open(): 
    with open("../data/test.txt", "w") as f: 
        f.write("test")
```

A prompt was shown to request the access to the file to be written, and you can deny that access by clicking “No” 

It will not work with the native APIs either: 

```
def test_native(): 
    handle = ctypes.windll.kernel32.CreateFileW("../data/test.txt", 0x10000000, 0, None, 1, 0x80, 0) 
    if handle > 0: 
        ctypes.windll.kernel32.CloseHandle(handle) 
    else: 
        raise RuntimeError("Failed to get the handle")
```

You can also test `os.system()` and `subprocess`: 

```
def test_os_system(): 
    os.system("powershell -command cat c:/data/test/bin/data/test.txt") 
def test_subprocess(): 
    subprocess.run(["powershell", "-command", "cat", "c:/data/test/bin/data/test.txt"], shell=True)
```

Importing modules by modifying `sys.path` will trigger the prompt too: 

```
def test_import(): 
    sys.path.append("../data") 
    import example
```

Clicking “no” will deny the access to the module (`example.py`is in the data directory and clicking “yes” would successfully import it): 

From the examples above, by introducing Win32 App Isolation you can help prevent Python from accessing the file system and the network without consent. 

There are two obvious questions: 

1.  How can you “click” the prompts if you want to deploy this on your server? As mentioned above, `isolatedWin32-promptForAccess`is only added for demonstration purposes, and it is not needed for isolation. Without it, your system would simply deny any access to user files, thus helping to prevent attacks like ransomware.
2.  Then how can you grant access to the application for required files like modules, scripts, and data? There are several ways to do it: 
    *   You can package the modules you need into your app, then they will be accessible by default.
    *   You can put the file you need access to into the app’s profile in `%localappdata%` where the app is granted access automatically.
    *   You can put the file to a “publisher directory”, which is any directory with a name ending with your app’s publisher ID in `c:\ProgramData`. ([win32-app-isolation/docs/fundamentals/consent.md at main · microsoft/win32-app-isolation (github.com)](https://github.com/microsoft/win32-app-isolation/blob/main/docs/fundamentals/consent.md))
    *   You can set the access of files and directories with icalcs to allow your app or all apps to access them.

With the technology, the users can create much lighter solutions when they try to execute code from an untrusted source. Isolated Python can live in parallel with normal Python and perform as a low-privilege interpreter on the side. Or, with correct configuration, it can be used on its own which could help protect the system from being compromised while keeping all the functionalities from Python.