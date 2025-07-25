# TRAM1
eventvwr.msc
regsvr32 /s C:\Windows\System32\MyMissing.dll
```
   Get-WinEvent -LogName Application -MaxEvents 20 |
     Where-Object { $_.LevelDisplayName -eq 'Error' } |
     Select-Object TimeCreated, ProviderName, Id, Message |
     Format-List -Property *
```
TimeCreated  : 7/22/2025 12:22:44 PM
ProviderName : Application Error
Id           : 1000
Message      : Faulting application name: ITD.OTS.Supervision.UI.exe, version: 0.0.1.0, time 
               stamp: 0x638ef647
               Faulting module name: KERNELBASE.dll, version: 10.0.19041.6093, time stamp:   
               0x11227201
               Exception code: 0xe0434352
               Fault offset: 0x0013b552
               Faulting process id: 0x28b0
               Faulting application start time: 0x01dbfac8a6b80674
               Faulting application path: C:\c++\HK_HDDT\ITD.OTS.Supervision.UI.exe
               Faulting module path: C:\WINDOWS\System32\KERNELBASE.dll
               Report Id: 68478c4b-f6e6-4d67-bf7b-63ecd440ef31
               Faulting package full name:
               Faulting package-relative application ID:

TimeCreated  : 7/22/2025 12:22:43 PM
ProviderName : .NET Runtime
Id           : 1026
Message      : Application: ITD.OTS.Supervision.UI.exe
               Framework Version: v4.0.30319
               Description: The process was terminated due to an unhandled exception.
               Exception Info: System.Runtime.InteropServices.COMException
                  at ITD.OTS.Supervision.UI.Controller.EmployeeController..ctor()
                  at ITD.OTS.Supervision.UI.MainWindow..ctor()

               Exception Info: System.Windows.Markup.XamlParseException
                  at System.Windows.Markup.WpfXamlLoader.Load(System.Xaml.XamlReader,
               System.Xaml.IXamlObjectWriterFactory, Boolean, System.Object,
               System.Xaml.XamlObjectWriterSettings, System.Uri)
                  at System.Windows.Markup.WpfXamlLoader.LoadBaml(System.Xaml.XamlReader,
               Boolean, System.Object, System.Xaml.Permissions.XamlAccessLevel, System.Uri)        
                  at System.Windows.Markup.XamlReader.LoadBaml(System.IO.Stream,
               System.Windows.Markup.ParserContext, System.Object, Boolean)
                  at System.Windows.Application.LoadBamlStreamWithSyncInfo(System.IO.Stream, 
               System.Windows.Markup.ParserContext)
                  at System.Windows.Application.LoadComponent(System.Uri, Boolean)
                  at System.Windows.Application.DoStartup()
                  at System.Windows.Application.<.ctor>b__1_0(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.DispatcherOperation.InvokeImpl()
                  at
               System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(System.Object) 
                  at
               System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext,    
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object)
                  at MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservi
               ngExecutionContext, System.Threading.ContextCallback, System.Object)
                  at System.Windows.Threading.DispatcherOperation.Invoke()
                  at System.Windows.Threading.Dispatcher.ProcessQueue()
                  at System.Windows.Threading.Dispatcher.WndProcHook(IntPtr, Int32, IntPtr,        
               IntPtr, Boolean ByRef)
                  at MS.Win32.HwndWrapper.WndProc(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)    
                  at MS.Win32.HwndSubclass.DispatcherCallbackOperation(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading
               .DispatcherPriority, System.TimeSpan, System.Delegate, System.Object, Int32)        
                  at MS.Win32.HwndSubclass.SubclassWndProc(IntPtr, Int32, IntPtr, IntPtr)
                  at MS.Win32.UnsafeNativeMethods.DispatchMessage(System.Windows.Interop.MSG       
               ByRef)
                  at System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.Di 
               spatcherFrame)
                  at System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.Dispat
               cherFrame)
                  at System.Windows.Application.RunDispatcher(System.Object)
                  at System.Windows.Application.RunInternal(System.Windows.Window)
                  at System.Windows.Application.Run(System.Windows.Window)
                  at System.Windows.Application.Run()
                  at ITD.OTS.Supervision.UI.App.Main()



TimeCreated  : 7/22/2025 12:17:12 PM
ProviderName : Application Error
Id           : 1000
Message      : Faulting application name: ITD.OTS.Supervision.UI.exe, version: 0.0.1.0, time       
               stamp: 0x638ef647
               Faulting module name: KERNELBASE.dll, version: 10.0.19041.6093, time stamp:
               0x11227201
               Exception code: 0xe0434352
               Fault offset: 0x0013b552
               Faulting process id: 0x32f0
               Faulting application start time: 0x01dbfac7e1385647
               Faulting application path: C:\c++\HK_HDDT\ITD.OTS.Supervision.UI.exe
               Faulting module path: C:\WINDOWS\System32\KERNELBASE.dll
               Report Id: b04ef611-910e-467f-ae5b-de594b49e158
               Faulting package full name:
               Faulting package-relative application ID:

TimeCreated  : 7/22/2025 12:17:12 PM
ProviderName : .NET Runtime
Id           : 1026
Message      : Application: ITD.OTS.Supervision.UI.exe
               Framework Version: v4.0.30319
               Description: The process was terminated due to an unhandled exception.
               Exception Info: System.Runtime.InteropServices.COMException
                  at ITD.OTS.Supervision.UI.Controller.EmployeeController..ctor()
                  at ITD.OTS.Supervision.UI.MainWindow..ctor()

               Exception Info: System.Windows.Markup.XamlParseException
                  at System.Windows.Markup.WpfXamlLoader.Load(System.Xaml.XamlReader,
               System.Xaml.IXamlObjectWriterFactory, Boolean, System.Object,
               System.Xaml.XamlObjectWriterSettings, System.Uri)
                  at System.Windows.Markup.WpfXamlLoader.LoadBaml(System.Xaml.XamlReader,
               Boolean, System.Object, System.Xaml.Permissions.XamlAccessLevel, System.Uri)        
                  at System.Windows.Markup.XamlReader.LoadBaml(System.IO.Stream,
               System.Windows.Markup.ParserContext, System.Object, Boolean)
                  at System.Windows.Application.LoadBamlStreamWithSyncInfo(System.IO.Stream, 
               System.Windows.Markup.ParserContext)
                  at System.Windows.Application.LoadComponent(System.Uri, Boolean)
                  at System.Windows.Application.DoStartup()
                  at System.Windows.Application.<.ctor>b__1_0(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.DispatcherOperation.InvokeImpl()
                  at
               System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(System.Object) 
                  at
               System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext,    
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object)
                  at MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservi 
               ngExecutionContext, System.Threading.ContextCallback, System.Object)
                  at System.Windows.Threading.DispatcherOperation.Invoke()
                  at System.Windows.Threading.Dispatcher.ProcessQueue()
                  at System.Windows.Threading.Dispatcher.WndProcHook(IntPtr, Int32, IntPtr,        
               IntPtr, Boolean ByRef)
                  at MS.Win32.HwndWrapper.WndProc(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)    
                  at MS.Win32.HwndSubclass.DispatcherCallbackOperation(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading
               .DispatcherPriority, System.TimeSpan, System.Delegate, System.Object, Int32)        
                  at MS.Win32.HwndSubclass.SubclassWndProc(IntPtr, Int32, IntPtr, IntPtr)
                  at MS.Win32.UnsafeNativeMethods.DispatchMessage(System.Windows.Interop.MSG       
               ByRef)
                  at System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.Di 
               spatcherFrame)
                  at System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.Dispat
               cherFrame)
                  at System.Windows.Application.RunDispatcher(System.Object)
                  at System.Windows.Application.RunInternal(System.Windows.Window)
                  at System.Windows.Application.Run(System.Windows.Window)
                  at System.Windows.Application.Run()
                  at ITD.OTS.Supervision.UI.App.Main()



TimeCreated  : 7/22/2025 12:16:29 PM
ProviderName : Application Error
Id           : 1000
Message      : Faulting application name: ITD.OTS.Supervision.UI.exe, version: 0.0.1.0, time       
               stamp: 0x638ef647
               Faulting module name: KERNELBASE.dll, version: 10.0.19041.6093, time stamp:
               0x11227201
               Exception code: 0xe0434352
               Fault offset: 0x0013b552
               Faulting process id: 0x2918
               Faulting application start time: 0x01dbfac7c79b393a
               Faulting application path: C:\c++\HK_HDDT\ITD.OTS.Supervision.UI.exe
               Faulting module path: C:\WINDOWS\System32\KERNELBASE.dll
               Report Id: a448b5df-ce38-41b9-92de-849f71bead54
               Faulting package full name:
               Faulting package-relative application ID:

TimeCreated  : 7/22/2025 12:16:29 PM
ProviderName : .NET Runtime
Id           : 1026
Message      : Application: ITD.OTS.Supervision.UI.exe
               Framework Version: v4.0.30319
               Description: The process was terminated due to an unhandled exception.
               Exception Info: System.Runtime.InteropServices.COMException
                  at ITD.OTS.Supervision.UI.Controller.EmployeeController..ctor()
                  at ITD.OTS.Supervision.UI.MainWindow..ctor()

               Exception Info: System.Windows.Markup.XamlParseException
                  at System.Windows.Markup.WpfXamlLoader.Load(System.Xaml.XamlReader,
               System.Xaml.IXamlObjectWriterFactory, Boolean, System.Object,
               System.Xaml.XamlObjectWriterSettings, System.Uri)
                  at System.Windows.Markup.WpfXamlLoader.LoadBaml(System.Xaml.XamlReader,
               Boolean, System.Object, System.Xaml.Permissions.XamlAccessLevel, System.Uri)        
                  at System.Windows.Markup.XamlReader.LoadBaml(System.IO.Stream,
               System.Windows.Markup.ParserContext, System.Object, Boolean)
                  at System.Windows.Application.LoadBamlStreamWithSyncInfo(System.IO.Stream,       
               System.Windows.Markup.ParserContext)
                  at System.Windows.Application.LoadComponent(System.Uri, Boolean)
                  at System.Windows.Application.DoStartup()
                  at System.Windows.Application.<.ctor>b__1_0(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.DispatcherOperation.InvokeImpl()
                  at
               System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(System.Object) 
                  at
               System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext,    
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object)
                  at MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservi
               ngExecutionContext, System.Threading.ContextCallback, System.Object)
                  at System.Windows.Threading.DispatcherOperation.Invoke()
                  at System.Windows.Threading.Dispatcher.ProcessQueue()
                  at System.Windows.Threading.Dispatcher.WndProcHook(IntPtr, Int32, IntPtr,        
               IntPtr, Boolean ByRef)
                  at MS.Win32.HwndWrapper.WndProc(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)    
                  at MS.Win32.HwndSubclass.DispatcherCallbackOperation(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading
               .DispatcherPriority, System.TimeSpan, System.Delegate, System.Object, Int32)        
                  at MS.Win32.HwndSubclass.SubclassWndProc(IntPtr, Int32, IntPtr, IntPtr)
                  at MS.Win32.UnsafeNativeMethods.DispatchMessage(System.Windows.Interop.MSG       
               ByRef)
                  at System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.Di 
               spatcherFrame)
                  at System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.Dispat 
               cherFrame)
                  at System.Windows.Application.RunDispatcher(System.Object)
                  at System.Windows.Application.RunInternal(System.Windows.Window)
                  at System.Windows.Application.Run(System.Windows.Window)
                  at System.Windows.Application.Run()
                  at ITD.OTS.Supervision.UI.App.Main()



TimeCreated  : 7/22/2025 12:10:19 PM
ProviderName : Application Error
Id           : 1000
Message      : Faulting application name: ITD.OTS.Supervision.UI.exe, version: 0.0.1.0, time       
               stamp: 0x638ef647
               Faulting module name: KERNELBASE.dll, version: 10.0.19041.6093, time stamp:
               0x11227201
               Exception code: 0xe0434352
               Fault offset: 0x0013b552
               Faulting process id: 0x2928
               Faulting application start time: 0x01dbfac6eaf0afeb
               Faulting application path: C:\c++\HK_HDDT\ITD.OTS.Supervision.UI.exe
               Faulting module path: C:\WINDOWS\System32\KERNELBASE.dll
               Report Id: ce80c07b-3059-4f58-bc51-e3201f918504
               Faulting package full name:
               Faulting package-relative application ID:

TimeCreated  : 7/22/2025 12:10:19 PM
ProviderName : .NET Runtime
Id           : 1026
Message      : Application: ITD.OTS.Supervision.UI.exe
               Framework Version: v4.0.30319
               Description: The process was terminated due to an unhandled exception.
               Exception Info: System.Runtime.InteropServices.COMException
                  at ITD.OTS.Supervision.UI.Controller.EmployeeController..ctor()
                  at ITD.OTS.Supervision.UI.MainWindow..ctor()

               Exception Info: System.Windows.Markup.XamlParseException
                  at System.Windows.Markup.WpfXamlLoader.Load(System.Xaml.XamlReader,
               System.Xaml.IXamlObjectWriterFactory, Boolean, System.Object,
               System.Xaml.XamlObjectWriterSettings, System.Uri)
                  at System.Windows.Markup.WpfXamlLoader.LoadBaml(System.Xaml.XamlReader,
               Boolean, System.Object, System.Xaml.Permissions.XamlAccessLevel, System.Uri)        
                  at System.Windows.Markup.XamlReader.LoadBaml(System.IO.Stream,
               System.Windows.Markup.ParserContext, System.Object, Boolean)
                  at System.Windows.Application.LoadBamlStreamWithSyncInfo(System.IO.Stream,       
               System.Windows.Markup.ParserContext)
                  at System.Windows.Application.LoadComponent(System.Uri, Boolean)
                  at System.Windows.Application.DoStartup()
                  at System.Windows.Application.<.ctor>b__1_0(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.DispatcherOperation.InvokeImpl()
                  at
               System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(System.Object) 
                  at
               System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext,    
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object)
                  at MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservi
               ngExecutionContext, System.Threading.ContextCallback, System.Object)
                  at System.Windows.Threading.DispatcherOperation.Invoke()
                  at System.Windows.Threading.Dispatcher.ProcessQueue()
                  at System.Windows.Threading.Dispatcher.WndProcHook(IntPtr, Int32, IntPtr,        
               IntPtr, Boolean ByRef)
                  at MS.Win32.HwndWrapper.WndProc(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)    
                  at MS.Win32.HwndSubclass.DispatcherCallbackOperation(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading
               .DispatcherPriority, System.TimeSpan, System.Delegate, System.Object, Int32)        
                  at MS.Win32.HwndSubclass.SubclassWndProc(IntPtr, Int32, IntPtr, IntPtr)
                  at MS.Win32.UnsafeNativeMethods.DispatchMessage(System.Windows.Interop.MSG       
               ByRef)
                  at System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.Di 
               spatcherFrame)
                  at System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.Dispat
               cherFrame)
                  at System.Windows.Application.RunDispatcher(System.Object)
                  at System.Windows.Application.RunInternal(System.Windows.Window)
                  at System.Windows.Application.Run(System.Windows.Window)
                  at System.Windows.Application.Run()
                  at ITD.OTS.Supervision.UI.App.Main()



TimeCreated  : 7/22/2025 12:04:11 PM
ProviderName : Application Error
Id           : 1000
Message      : Faulting application name: ITD.OTS.Supervision.UI.exe, version: 0.0.1.0, time       
               stamp: 0x638ef647
               Faulting module name: KERNELBASE.dll, version: 10.0.19041.6093, time stamp:
               0x11227201
               Exception code: 0xe0434352
               Fault offset: 0x0013b552
               Faulting process id: 0x608
               Faulting application start time: 0x01dbfac60f9c1f5e
               Faulting application path: C:\c++\HK_HDDT\ITD.OTS.Supervision.UI.exe
               Faulting module path: C:\WINDOWS\System32\KERNELBASE.dll
               Report Id: 1691c380-1f59-4e99-bec7-612f106d982d
               Faulting package full name:
               Faulting package-relative application ID:

TimeCreated  : 7/22/2025 12:04:11 PM
ProviderName : .NET Runtime
Id           : 1026
Message      : Application: ITD.OTS.Supervision.UI.exe
               Framework Version: v4.0.30319
               Description: The process was terminated due to an unhandled exception.
               Exception Info: System.Runtime.InteropServices.COMException
                  at ITD.OTS.Supervision.UI.Controller.EmployeeController..ctor()
                  at ITD.OTS.Supervision.UI.MainWindow..ctor()

               Exception Info: System.Windows.Markup.XamlParseException
                  at System.Windows.Markup.WpfXamlLoader.Load(System.Xaml.XamlReader,
               System.Xaml.IXamlObjectWriterFactory, Boolean, System.Object,
               System.Xaml.XamlObjectWriterSettings, System.Uri)
                  at System.Windows.Markup.WpfXamlLoader.LoadBaml(System.Xaml.XamlReader,
               Boolean, System.Object, System.Xaml.Permissions.XamlAccessLevel, System.Uri)        
                  at System.Windows.Markup.XamlReader.LoadBaml(System.IO.Stream,
               System.Windows.Markup.ParserContext, System.Object, Boolean)
                  at System.Windows.Application.LoadBamlStreamWithSyncInfo(System.IO.Stream,       
               System.Windows.Markup.ParserContext)
                  at System.Windows.Application.LoadComponent(System.Uri, Boolean)
                  at System.Windows.Application.DoStartup()
                  at System.Windows.Application.<.ctor>b__1_0(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.DispatcherOperation.InvokeImpl()
                  at
               System.Windows.Threading.DispatcherOperation.InvokeInSecurityContext(System.Object) 
                  at
               System.Threading.ExecutionContext.RunInternal(System.Threading.ExecutionContext,    
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object, Boolean)
                  at System.Threading.ExecutionContext.Run(System.Threading.ExecutionContext,      
               System.Threading.ContextCallback, System.Object)
                  at MS.Internal.CulturePreservingExecutionContext.Run(MS.Internal.CulturePreservi
               ngExecutionContext, System.Threading.ContextCallback, System.Object)
                  at System.Windows.Threading.DispatcherOperation.Invoke()
                  at System.Windows.Threading.Dispatcher.ProcessQueue()
                  at System.Windows.Threading.Dispatcher.WndProcHook(IntPtr, Int32, IntPtr,        
               IntPtr, Boolean ByRef)
                  at MS.Win32.HwndWrapper.WndProc(IntPtr, Int32, IntPtr, IntPtr, Boolean ByRef)    
                  at MS.Win32.HwndSubclass.DispatcherCallbackOperation(System.Object)
                  at System.Windows.Threading.ExceptionWrapper.InternalRealCall(System.Delegate,   
               System.Object, Int32)
                  at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(System.Object,
               System.Delegate, System.Object, Int32, System.Delegate)
                  at System.Windows.Threading.Dispatcher.LegacyInvokeImpl(System.Windows.Threading
               .DispatcherPriority, System.TimeSpan, System.Delegate, System.Object, Int32)        
                  at MS.Win32.HwndSubclass.SubclassWndProc(IntPtr, Int32, IntPtr, IntPtr)
                  at MS.Win32.UnsafeNativeMethods.DispatchMessage(System.Windows.Interop.MSG       
               ByRef)
                  at System.Windows.Threading.Dispatcher.PushFrameImpl(System.Windows.Threading.Di 
               spatcherFrame)
                  at System.Windows.Threading.Dispatcher.PushFrame(System.Windows.Threading.Dispat
               cherFrame)
                  at System.Windows.Application.RunDispatcher(System.Object)
                  at System.Windows.Application.RunInternal(System.Windows.Window)
                  at System.Windows.Application.Run(System.Windows.Window)
                  at System.Windows.Application.Run()
                  at ITD.OTS.Supervision.UI.App.Main()
