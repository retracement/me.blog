+++
date = '2015-11-30T01:01:00+01:00'
draft = false
title = 'Bitesize SSIS - Passing SSIS variables into script components'
categories = ['Technology']
tags = ['Dotnet','SQL','SSIS']
+++

I hate SSIS. It seems to me that it is full of certain nuances and unless you are regularly developing SSIS packages, they are easy to forget or it is easy to miss specific important steps. I first started using SSIS back in 2005 when it was directly introduced to replace DTS, but even today I am constantly going around in circles whenever I have to return to write certain functionality.Therefore I have decided to put together a "Bitesize" series of posts that encapsulate simple operations in order to help not just you, but more importantly remind me! Hopefully this will save me time in the long run...

---

Script Task components are incredibly useful if you (like me) have spent any time in a development background. My language of choice is C#, but Script Task components also support Visual Basic (for the purposes of my explanation I will assume C# is your preferred language too). Sooner or later you will require the ability to pass in dynamic values into the Script Task so that you may consume or manipulate them through code.

There are essentially three operations required in order to use SSIS variables in your scripts, they are:

- Define your (SSIS) variable/s
- Declare (SSIS) variable/s to Script Task as *ReadOnlyVariables* or *ReadWriteVariables*
- Assign (SSIS) variables/s to Script variables (or visa-versa)

---

# Define your SSIS variable/s

Your SSIS variables can be scoped as necessary and assuming that the Script Task has visibility, the scope is irrelevant for their consumption. Ensure that you create them using the correct data type so that you do not have to perform type checks in the script to allow simple casts to the correct required type.

To view SSIS variables, from the menu simply click *SSIS/ Variables* to display the Variables pane. It can be useful to click the *Variable Grid Options* button on the Variable pane and select *Show variables of all scopes*.

![MyVar](/images/2015/myvar.jpg)

Now you have the Variables pane visible, simply click on the *Add variable* button to define your new variable/s, their data type and their scopes. It is usually advisable to assign a data value regardless of whether this will be overwritten or not simply because doing so, allows you to evaluate variable expressions and have useful output returned at design time.

*Tip: If you do not create the SSIS variables up front, it is sometimes possible to create them directly from the container or task editor. For instance, when in the Foreach Loop Editor we can create new package (or scoped) variables when attempting to define a Variable Mapping. We shall be discussing Variable Mappings further in a future post.*

---

# Declare SSIS variables to Script Task

This is one of those operations which might not be obvious to anyone returning back to SSIS after any length of time but SSIS variables are not available to Script Tasks unless you explicitly declare them to the Script Task (yes I know they are declared to the package, but the script task needs to know that you want to use them and how).

If you don't and attempt to use them you will see something like this...

![error](/images/2015/error.jpg)

DTS Script Task has encountered an exception in user code:

>Project name: ST_a814ca7dfd6244b3a42a81260bb1d4a8
Exception has been thrown by the target of an invocation.
at System.RuntimeMethodHandle.InvokeMethod(Object target, Object[] arguments, Signature sig, Boolean constructor)
at System.Reflection.RuntimeMethodInfo.UnsafeInvokeInternal(Object obj, Object[] parameters, Object[] arguments)
at System.Reflection.RuntimeMethodInfo.Invoke(Object obj, BindingFlags invokeAttr, Binder binder, Object[] parameters, CultureInfo culture)
at System.RuntimeType.InvokeMember(String name, BindingFlags bindingFlags, Binder binder, Object target, Object[] providedArgs, ParameterModifier[] modifiers, CultureInfo culture, String[] namedParams)
at Microsoft.SqlServer.Dts.Tasks.ScriptTask.VSTATaskScriptingEngine.ExecuteScript()

Think of this step as really declaring the variables for interop. It is important that you decide how the SSIS variable/s is/are going to be consumed by the script code, so you must define them as *ReadOnlyVariables* or *ReadWriteVariables*. There is a misconception that placing your SSIS variables in the read only list would prevent write backs in code, but this is not exactly the case and the reason to use them is more driven by the necessity to serialise access to them more than anything else. For more information about those two list types you should read Todd McDermids excellent post [titled Use ReadOnlyVariables and ReadWriteVariables properties in Scripts](https://toddmcdermid.blogspot.co.uk/2010/08/use-readonlyvariables-and.html) and follow his best practice advice when choosing between them. You can always change which list you use afterwards if absolutely necessary.

![scripttask](/images/2015/scripttask.jpg)

Launch the Script Task Editor by selecting the Script Task properties and clicking the ellipses button next to either of those options just mentioned. Select all the variables that you wish to make available to the Script task and click *OK* to close the *Select Variables* dialog.

# Assign SSIS variable/s to Script variables (or visa-versa)

So far, all we have really done is defined the SSIS variables and put in place the plumbing to allow their consumption in code. As mentioned, the way you are allowed to consume them will depend upon how you have declared them to the script task.

Launch the Visual Studio Tools for Applications script editor by (you guessed it) clicking *Edit Script*... while still in the *Script Task properties* dialog.

Scroll to the `public Main` method and you will see:

```ruby
// TODO: Add your code here
```

Obviously this is your place to input your code block, and more importantly, the place where you will consume your SSIS variables!

In order to assign your SSIS variable to a script variable:

```ruby
var myvar = (int)Dts.Variables["User::MyVar"].Value;
```

In the example above we are using [inferred typing](https://msdn.microsoft.com/en-GB/library/bb384061.aspx) and explicitly casting our SSIS variable (and therefore script variable type) to Integer. If you have made a mistake with the SSIS variable type or cast to the wrong script variable type, this might result in a runtime error or unexpected results.

In order to assign your script variable value back to a SSIS variable the operation is pretty much reversed as you would expect:

```ruby
Dts.Variables["User::MyVar"].Value = myvar;
```

Notice that in the example above the assumption is that the SSIS variable datatype is compatible with the script variable type.

Once you have finished writing your code block you may save your code and close the Script Editor. All that is left is to click the OK button to close the Script Task Editor and run your package!

---

To be honest, that is all there really is to using SSIS variables within a script task.  I hope this has been a useful post for you, but at very least I won't keep forgetting!

---

![extendingssis](/images/2015/extendingssis.webp)

Want more Bitesize SSIS tips? Then keep an eye open for the other posts in the series and if you cannot wait until then, you might want to take a look at my good friend Régis Baccaro's new book [Extending SSIS with .NET Scripting](https://www.apress.com/9781484206393) which is out now through [Apress](https://www.apress.com/). And for those who are curious, No he didn't give me a free copy!

