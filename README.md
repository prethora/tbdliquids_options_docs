# TBDLiquids Options Documentation

## Structure

Options can be defined on three levels:

* **Global Options**

    &nbsp;
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

* **Collection Options**

    &nbsp;
    Accessible through the `Edit Options` button of a collection in the *Collection Editor*: Actual options are defined here and are simply applied to all products in the collection. Multiple `definitions` can also be defined at this level for collections with products that do not all have the same options profile - this is used in the `Retired Products` collection, which has `Retired Salts` and `Retired E-Juice` products, each requiring their own set of options.
    &nbsp;


* **Product Options**

  &nbsp;
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

## Pricing & Inventory

I made a somewhat bold design choice that will give us maximum flexibility with pricing and is simultaneously the cleanest possible solution for inventory keeping. The possible trade offs are ultimately manageable and not really an issue.

We are completely moving away from Shopify's pricing management with it's limitations and potential clutter when it comes to variants and inventory tracking. We are completely separating the concerns of pricing and inventory management.

What this means is that all variants that track inventory (well that is, all product variants) have a price of \$0. Essentially, these line items contribute to defining the order, decrementing the inventory count, but they do not at all affect the price. The price component of the order is simply defined by a single variant from a product called "Cent" which is priced at, you guessed it, $0.01. If we want to set the price to \$50.95, we set the quantity of this line item to 5095. 

Since we completely control the visual of how items show from the cart to the notification emails, we can do this. This means we can manage any pricing scheme we want, without having to mess with variants, we can entirely manage sales and deals ourselves without having to mess with scripts and whatnot. Total flexibility.

It also means there is only ever a single $0 variant to represent a physical product/quantity that you want to track - and of course, from whatever product, bundle or pack it is ordered from, there are no duplicate products, it always points to the right variant - thus perfect and accurate inventory tracking. 

Now the aforementioned trade off is order editing - you won't be able to do that like you do now using the Shopify order editing function. Because obviously, you wouldn't see clearly what the price of each item is, you would just see all zeros and the entirety of the order price in the "Cent" product line item. If you wanted to process a partial refund, you would have to calculate the price to refund yourself, edit the quantity of the "Cent" product to lower the price appropriately and remove the other line items as necessary, for accurate tracking purposes. You'd have to do something similar to increase any line item quantities.

Not to worry though, this was not a good solution now either with bold options, because if you had a complicated order, with a whole lot of interconnected line items lying around, and you had to remove a bundle from the order, while there is another pack or something on the order, it could be headache to remove the correct line items. 

So the solution is that when you want to edit the order, you will instead click on "Edit Order" from the "More actions..." menus and it will be managed from a page in the app. This will essentially give you the same view the customer had on their cart, so you can remove items, or adjust down or up the item quantities - and the app would manage everything seamlessly. Problem solved.


