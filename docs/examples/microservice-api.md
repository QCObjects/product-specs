# Microservice API (BFF)

Source animation: `svg-code-examples/qcobjects-microservice-api.svg`.

```javascript
<!-- QCObjects Micro-services API Example -->
Import ("org.mycompany.backend.api.client_services");
Package("org.mycompany.api.products",[
  Class("Microservice",BackendMicroservice,{
    get (data){
      /* You can handle the micro-service instance using this pointer */
      let microservice = this;
      /*
      * This will call a proxy service declared in the client_services package
      * The serviceLoader is promise oriented
      */
      var service = serviceLoader(New(ProxyService,{
        data:null
      })).then(
        (successfulResponse)=>{
          // This will show the service response as a plain text
          logger.debug(successfulResponse.service.template);
          microservice.body = JSON.parse(successfulResponse.service.template);
          try {
            /* 
            * A call to the done method will 
            * finish the task and end the response of the micro-service
            */
            microservice.done();
          } catch (e){
            logger.debug(e.message);
          }
        },
        (failedResponse)=>{
           logger.debug(failedResponse);
        });
    }
  })
]);
```
