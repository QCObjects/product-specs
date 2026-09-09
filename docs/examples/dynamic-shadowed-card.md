# Dynamic shadowed card

Source animation: `svg-code-examples/qcobjects-dynamic-card.svg`.

```html
<!-- QCObjects Dynamic Shadowed Card Component example -->
<component name="shadowed-card" 
           shadowed="true" 
           data-title="{{title}}"
           data-description="{{description}}" 
           data-image="{{image}}">
  <img slot="logo" src="img/{{image}}" alt="{{title}}" style="width:100%">
  <b slot="card_title">{{title}}</b>
  <div slot="card_description">{{description}}</div>
</component>
```
