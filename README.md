# TBDLiquids Options Documentation

## General Design

Options can be defined on three levels:

* **Global Options**
    Accessible from the `Options Editor` menu item of the App: Used to define product aliases, which map placeholders to specific Shopify product IDs for easy and human-readable reference in option configurations. For example:

    ```yaml
    products:
      ...
      - id: countryCrumbleSalt
        product_id: 7372104007871    
    ```

    If a product is defined without a `product_id`, a basic product (with a single $0 variant) will be created for it. These are meant to be used for tracking inventory of products that are not directly for sale. These products don't have to be free, they may add a charge to an order (more on how this is managed below). For example:

    ```yaml
    products:
      ...
    - id: usbCCable
    - id: mysteryFlavor

    ```
    &nbsp;

* **Collection Options**

    Accessible through the `Edit Options` button of a collection in the *Collection Editor*: Actual options are defined here and are simply applied to all products in the collection. Multiple `definitions` can also be defined at this level for collections with products that do not all have the same options profile - this is used in the `Retired Products` collection, which has `Retired Salts` and `Retired E-Juice` products, each requiring their own set of options.
    &nbsp;


* **Product Options**

  Accessible with the "gear" icon on the right of products in the `Collection Editor`. By default products inherit their options configuration from their collection, but in the *Options Editor* for a specific product, you can select `Overwrite` and include a product specific options configuration. It is also possible to overwrite the collection configuration but also inherit it, but setting...

  ```yaml
  inherits: true
  ```

  ...in the product specific configuration. The purpose of this being to customize the collection configuration. For example, the `Retired Products` collection configuration doesn't have any actual options, it only has 2 `definitions`: `retired-salts` and `retired-e-juice`, which are potential configurations. The `Horchata Salt` product options configuration is set to `Overwrite` and looks like this:

  ```yaml
  inherits: true
  load: retired-salts
  ```

  Another example is the `Berry Cool` and `Polar Pear` products, which unlike the other `Signature E-Juice` products has an `Add Extra Cool` option and doesn't have a `+Cool (Menthol)` addon. The `Signature E-Juice` collection configuration defines a variable `withExtraCool` and uses it in a `conditional` field on those options, which is essentially a Javascript function - if it returns `true` the option is included, or ignored otherwise. The `Berry Cool` product options configuration is set to `Overwrite` and looks like this:
  
  ```yaml
  inherits: true
  var:
    withExtraCool: true
  ```