# nuget-package-fail
Test case that shows nuget update failure


All the projects are setup to depend on 2 nugets which I will represent in json.
It is important to note that the ***projects*** explicitly depend on B and F.  B also depends on F.

```json
  "MyProject":{
    "B.DependsOnSome.Package 1.0.0":{
      "F.DoesNotDependOnAnything.Package 1.0.0":{}
    },
    "F.DoesNotDependOnAnything.Package 1.0.0":{}
  }
```
Imagine that a nuget gets reved up and in the process changes out all nugets it depended on.
In my example, my B nuget dumps its dependency on F and now has a new dependency on G
```
  "B.DependsOnSome.Package 1.0.1":{
    "G.DoesNotDependOnAnything.Package 1.0.0":{}
  }

```
Ok, so lets do an update and see what happens


```powershell
Update-Package -Id B.DependsOnSome.Package -ProjectName M -Version 1.0.1
```


For projects that use packages.config to manage nugets we get the following;

```xml
Pre Update
<packages>
  <package id="B.DependsOnSome.Package" version="1.0.0" targetFramework="net452" />
  <package id="F.DoesNotDependOnAnything.Package" version="1.0.0" targetFramework="net452" />
</packages>
```
```xml
Post Update
<packages>
  <package id="B.DependsOnSome.Package" version="1.0.1" targetFramework="net452" />
  <package id="F.DoesNotDependOnAnything.Package" version="1.0.0" targetFramework="net452" />
  <package id="G.DoesNotDependOnAnything.Package" version="1.0.0" targetFramework="net452" />
</packages>
```
While it may appear that this is correct, it actually is not.  What has happened here is that the G nuget has now been granted first class citizenship.  It is recorded that the project itself requires it, when in fact is is only B 1.0.1. that has that dependency.

It will become more apparent when we uninstall B.
```xml
Post Uninstall of B
<packages>
  <package id="F.DoesNotDependOnAnything.Package" version="1.0.0" targetFramework="net452" />
  <package id="G.DoesNotDependOnAnything.Package" version="1.0.0" targetFramework="net452" />
</packages>
```
***WTF*** is G still doing there?

The good news is that this is fixed in the new VS 2017 projects that do not use packages.config to manage nugets.  I can't imagine how this dependency blunder made it this far, but better late than never.






