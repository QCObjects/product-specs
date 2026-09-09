# Products grid template

Source animation: `svg-code-examples/qcobjects-products-grid-example.svg`.

```html
<!-- QCObjects Products Grid Component Example -->
<h1>Latest Products</h1>
<p>Find our latest products here </p>
<p></p>
<style>
$layout(portrait, css/components/grid-products-portrait.css)
$layout(landscape, css/components/grid-products-landscape.css)
component[name=grid-products] div.shadowHost {
  overflow-x: scroll;
}
</style>
<component 
 name="grid-products"      
 componentClass="org.qcobjects.components.grid.GridComponent"  subcomponentClass="ProductCardComponent" 
 serviceClass="ProductsService">
</component>
```
