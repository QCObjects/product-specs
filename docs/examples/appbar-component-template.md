# Appbar component template

Source animation: `svg-code-examples/qcobjects-appbar-component-template-example.svg`.

```html
<!-- QCObjects appbar component template example -->
<style>
  @import url("./css/components/appbar-white.css");
</style>
<nav>
  <span class="topnav">
    <component name="topmenu">
        <routing path="#page1" name="blank"></routing>
        <routing path="#page2" name="blank"></routing>
        <routing path="#signup" name="blank"></routing>
        <routing path="#signin" name="blank"></routing>
        <routing path="#" name="topmenu"></routing>
        <routing path="" name="topmenu"></routing>
    </component>
  </span>
  <button  aria-label="nav button" role="button" tabindex="-1" class="navbtn"
          onclick="global.sideNavController.toggle()"></button>
  <component name="nav" controllerClass="SideNavController"></component>
</nav>
```
