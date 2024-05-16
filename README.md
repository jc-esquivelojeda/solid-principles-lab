# solid-principles-lab

## This repository contains the solution to the solid principles lab

Tarea 1: analizar el código 
- Se identifica mas de una responsabilidad en las clase por lo que se debe hacer separacion de responsabilidades acorde a su funcion (SRP)
- Se identifican 2 tipos de procesamientos de ordenes, se puede modificar el código para que permita  implementar mas escenarios de ordenes (OCP, LSP)
- la notificacion de la orden hacia el cliente se puede abrir para mas escenarios mediante OCP

### Initial System Code
```
class SystemManager {
    processOrder(order) {
        if (order.type == "standard") {
            verifyInventory(order);
            processStandardPayment(order);
        } else if (order.type == "express") {
            verifyInventory(order);
            processExpressPayment(order, "highPriority");
        }
        
        updateOrderStatus(order, "processed");
        notifyCustomer(order);
    }
    verifyInventory(order) {
        // Checks inventory levels
        if (inventory < order.quantity) {
            throw new Error("Out of stock");
        }
    }

    processStandardPayment(order) {
        // Handles standard payment processing
        if (paymentService.process(order.amount)) {
            return true;
        } else {
            throw new Error("Payment failed");
        }
    }

    processExpressPayment(order, priority) {
        // Handles express payment processing
        if (expressPaymentService.process(order.amount, priority)) {
            return true;
        } else {
            throw new Error("Express payment failed");
        }
    }

    updateOrderStatus(order, status) {
        // Updates the order status in the database
        database.updateOrderStatus(order.id, status);
    }

    notifyCustomer(order) {
        // Sends an email notification to the customer
        emailService.sendEmail(order.customerEmail, "Your order has been processed.");
    }
}
```


### Tarea 2 y 3 : Codigo refactorizado y cambios documentados

Se define un enum para identificar las prioridades de las ordenes

```
    enum PRIORITY {
        STANDARD, EXPRESS
    }
```

Se define un enum para identificar los estatus de las ordenes

```
     enum STATUS{
        PROCCESED
    }
```
Se abstrae el servicio de notificacion para permitir si se desea ampliar los tipos de notificaciones permitir otros OCP

```
     interface NotificationService {
        void notify(Order order, String message);
    }
```
Se realiza la implementacion de la notificacion por correo

```
    class EmailNotificationService implements NotificationService {
                            
         void notify(Order order){        
            sendEmail(order.customerEmail, "Your order has been processed.");
         }
         
        void  sendEmail(order.customerEmail msg){
            // logica de envio de correo
         }         
      }
      
    }
```
 Con el fin de permitir agregar mas tipos de servicios de pago este se abstrae y se aplica OCP
```
    interface PaymentService{
        boolean process(Double amount, PRIORITY  priority);
    }
    
    // Se implementa la logica de procesamiento para pago standard
    class StandardPaymentService implements PaymentService{
        boolean process(Double amount, PRIORITY priority){
            // logica faltante
        }
    }
    
    // Se implementa la logica de procesamiento para pago express
     class ExpressPaymentService implements PaymentService{
        boolean process(Double amount, PRIORITY priority){
         // logica faltante
        }
    }
```

Se extrae el metodo de verifyInventory dentro de otra clase que refleje mas su funcion (SRP)

```
    class InventoryService {
   
        void verifyInventory(Order order) throws InventoryException {
            if (inventory < order.getQuantity()) {
                throw new InventoryException("Out of stock");
            }
         }
    
```    
Se define una clase abstracta que permitirá el llevar la logica de negocio de las ordenes
```
    abstract class OrderProcessor {
    
        private PaymentService paymentService;
        private NotificationService notificationService;
        private Database database;
        
        // se define el metodo que permitira el ocp para mas tipos de ordenes futuras
        abstract void processOrder(Order order) throws PaymentException, InventoryException;
        
        // Se aplica el principio de inversion de dependencias
        OrderProcessor(PaymentService paymentService, NotificationService notificationService,Database database, InventoryService inventoryService) {
            this.paymentService = paymentService;
            this.notificationService = notificationService;
            this.database = database;
            this.inventoryService = inventoryService;
        }
                 
                 
         updateOrderStatus(Order order, Status status) {
            database.updateOrderStatus(order.id, status);
         }
         
}

```
La clase StandardOrderProcessor solo manejara los pagos standard

```
class StandardOrderProcessor extends OrderProcessor {
    void processOrder(Order order) throws PaymentException, InventoryException {
        inventoryService.verifyInventory(order);
        if (paymentService.process(order.getAmount(),PRIORITY.STANDARD)) {
            return;
        } else {
            throw new PaymentException("Standard Payment failed");
        }
        updateOrderStatus(order, "processed");
        notificationService.notify(order);
    }
    
}
```

 La clase ExpressOrderProcessor solo manejara los pagos express
```
class ExpressOrderProcessor extends OrderProcessor {
    void processOrder(Order order) throws PaymentException, InventoryException {
        inventoryService.verifyInventory(order);
         if (paymentService.process(order.getAmount(), PRIORITY.EXPRESS)) {
            return;
        } else {
            throw new PaymentException("Express payment failed");
        }
        updateOrderStatus(order, "processed");
        notificationService.notify(order);
    }    
}
```

La clase SystemManager class se simpliffico para cumplir con el principio SRP
    su funcion sera recibir ordenes y procesarlas

```
class SystemManager {
    private OrderProcessor orderProcessor;
    
    // El principio de inversion de dependencia es aplicado para hacer uso de un procesador de ordenes 
    SystemManager(OrderProcessor orderProcessor) {
        this.orderProcessor = orderProcessor;
    }

   /*  El metodo processOrder internamente delegara la responsabilidad al OrderProcessor que en base a
     la necesidad se han implementado las clases deseadas*/
    void processOrder(Order order) {
        orderProcessor.processOrder(order);
    }
}

```
