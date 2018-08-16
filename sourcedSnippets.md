Snippets from Csharp forums:-

using System.Diagnostics;

class Program
{
    static void Main()
    {
        Process.Start("C:\\");
    }
}
If your application needs cmd arguments, use something like this:

using System.Diagnostics;

class Program
{
    static void Main()
    {
        LaunchCommandLineApp();
    }

    /// <summary>
    /// Launch the legacy application with some options set.
    /// </summary>
    static void LaunchCommandLineApp()
    {
        // For the example
        const string ex1 = "C:\\";
        const string ex2 = "C:\\Dir";

        // Use ProcessStartInfo class
        ProcessStartInfo startInfo = new ProcessStartInfo();
        startInfo.CreateNoWindow = false;
        startInfo.UseShellExecute = false;
        startInfo.FileName = "dcm2jpg.exe";
        startInfo.WindowStyle = ProcessWindowStyle.Hidden;
        startInfo.Arguments = "-f j -o \"" + ex1 + "\" -z 1.0 -s y " + ex2;

        try
        {
            // Start the process with the info we specified.
            // Call WaitForExit and then the using statement will close.
            using (Process exeProcess = Process.Start(startInfo))
            {
                exeProcess.WaitForExit();
            }
        }
        catch
        {
             // Log error.
        }
    }
}

===========
===========

MSDN example:

using System;
using System.Diagnostics;
using System.ComponentModel;

namespace MyProcessSample
{
    class MyProcess
    {
        public static void Main()
        {
            Process myProcess = new Process();

            try
            {
                myProcess.StartInfo.UseShellExecute = false;
                // You can start any process, HelloWorld is a do-nothing example.
                myProcess.StartInfo.FileName = "C:\\HelloWorld.exe";
                myProcess.StartInfo.CreateNoWindow = true;
                myProcess.Start();
                // This code assumes the process you are starting will terminate itself.
                // Given that is is started without a window so you cannot terminate it
                // on the desktop, it must terminate itself or you can do it programmatically
                // from this application using the Kill method.
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }
        }
    }
}

========
========

Welcome to MSDN Forums, Julio.

I prepared an example which works fine and I hope it can be a roadmap for this issue:

Create a new solution name ConsoleApplication1 and add a ConsoleApplication named ConsoleApplication1 to it as below
using System;
using System.Diagnostics;
using System.IO;

namespace ConsoleApplication1
{
 class Program
 {
  static void Main(string[] args)
  {
   Console.Write("Enter File Name:");
   string fileName = Console.ReadLine();
   using (StreamWriter sw = File.CreateText(fileName))
   {
    sw.WriteLine("I have been created by " + Process.GetCurrentProcess().ProcessName + " @ " + DateTime.Now);
    sw.Close();
   }
  }
 }
}
Add another ConsoleApplication named ConsoleApplication2 to the solution as below
using System.Diagnostics;

namespace ConsoleApplication2
{
 class Program
 {
  static void Main(string[] args)
  {
   Process myProcess = new Process();

   myProcess.StartInfo.FileName = @"ConsoleApplication1.exe";
   myProcess.StartInfo.UseShellExecute = false;
   myProcess.StartInfo.RedirectStandardOutput = true;
   myProcess.StartInfo.RedirectStandardInput = true;

   myProcess.Start();

   string redirectedOutput=string.Empty;
   while ((redirectedOutput += (char)myProcess.StandardOutput.Read()) != "Enter File Name:") ;

   myProcess.StandardInput.WriteLine("passedFileName.txt");

   myProcess.WaitForExit();

   //verifying that the job was successfull or not?!
   Process.Start("explorer.exe", "passedFileName.txt");
  }
 }
}

Right click ConsoleApplication2 project, click Properties and in Debug tab set Working directory to the debug folder of ConsoleApplication1 (it causes that the ConsoleApplication1.exe and ConsoleApplication2.exe in same folder which reduces the file path)
Right click ConsoleApplication2 and click Set as startup project
Rebuild the solution and run.
With best regards,

Yasser.

========
========

