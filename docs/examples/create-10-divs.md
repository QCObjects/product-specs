# Create 10 div elements

Source animation: `svg-code-examples/qcobjects-create-10-div-elements.svg`.

```javascript
range(9).map(index => New(
  Component,{
    name:`component-${index}`,
    template:`This is the component element number ${index}`,
    tplsource:"inline",
    body:_DOMCreateElement("div")
  }
)).map(e=>document.body.append(e))
```
