# Dynamic promise packages

Source animation: `svg-code-examples/qcobjects-dynamic-promise-packages.svg`.

```javascript
// QCObjects - Dynamic Promise Packages
// Every dependency package is loaded on demand. 
// Every class is declared but loaded only when needed
Import ("org.mycompany.dependency1.controllers")
.then (()=>
Import ("org.mycompany.dependency2.controllers")
).then (()=>
  Package ("org.mycompany.controllers",[
    Class ("MyClass", 
      ClassFactory("org.mycompany.dependency1.controllers.NestedDependency1Class"),
    {
       prop1: "some value",
       prop2: 7, // some numeric value
       someMethod1 () {
         // some custom behaviour
       }
    }),
    Class ("AnotherClass",NestedDependency2Class, {
       prop1: "some another value",
       prop2: 10, // some numeric value
       someMethod2 () {
         let myobj1 = New(MyClass);
         logger.debug(myobj1.prop1);
         // shows "some value"
         logger.debug(this.prop1)
         // shows "some another value"
         logger.debug(myobj1.prop3)
         // shows some value comming from NestedDependency1Class
    })
  ])
)
```
