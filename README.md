# TBDLiquids Options Documentation

## Structure

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

* **Collection Options**

    Accessible through the `Edit Options` button of a collection in the *Collection Editor*: Actual options are defined here and are simply applied to all products in the collection. Multiple `definitions` can also be defined at this level for collections with products that do not all have the same options profile - this is used in the `Retired Products` collection, which has `Retired Salts` and `Retired E-Juice` products, each requiring their own set of options.
    &nbsp;


* **Product Options**

  Accessible with the "gear" icon on the right of products in the `Collection Editor`. By default products inherit their options configuration from their collection, but in the *Options Editor* for a specific product, you can select `Overwrite` and include a product specific options configuration. It is also possible to overwrite the collection configuration but also inherit it, by setting...

  ```yaml
  inherits: true
  ```

  ...in the product specific configuration. The purpose of this being to customize the collection configuration. For example, the `Retired Products` collection configuration doesn't have any actual options, it only has 2 `definitions`: `retired-salts` and `retired-e-juice`, which are potential configurations. The `Horchata Salt` product options configuration is set to `Overwrite` and looks like this:

  ```yaml
  inherits: true
  load: retired-salts
  ```

  Another example is the `Berry Cool` and `Polar Pear` products, which unlike the other `Signature E-Juice` products have an `Add Extra Cool` option and don't have a `+Cool (Menthol)` addon. The `Signature E-Juice` collection configuration defines a variable `withExtraCool` and uses it in a `conditional` field on those options, which is linked to that variable. The `Berry Cool` product options configuration is set to `Overwrite` and looks like this:
  
  ```yaml
  inherits: true
  var:
    withExtraCool: true
  ```

## Pricing & Inventory

I made a somewhat bold design choice that will give us maximum flexibility with pricing and is simultaneously the cleanest possible solution for inventory keeping. The trade off is ultimately manageable and not really an issue.

We are completely moving away from Shopify's pricing management with its limitations and potential clutter when it comes to variants and inventory tracking. We are completely separating the concerns of pricing and inventory management.

What this means is that all variants that track inventory (well that is, all product variants) have a price of \$0. Essentially, these line items contribute to defining the order, decrementing the inventory count, but they do not at all affect the price. The price component of the order is simply defined by a single variant from a product called "Cent" which is priced at, you guessed it, $0.01. If we want to set the price to \$50.95, we set the quantity of this line item to 5095. 

Since we completely control the visual of how items show from the cart to the notification emails, we can do this. This means we can manage any pricing scheme we want, without having to mess with variants, we can entirely manage sales and deals ourselves without having to mess with scripts and whatnot. Total flexibility.

It also means there is only ever a single $0 variant to represent a physical product/quantity that you want to track - and of course, from whatever product, bundle or pack it is ordered from, there are no duplicate products, it always points to the right variant - thus we have perfect and accurate inventory tracking. 

Now the aforementioned trade off is order editing - you won't be able to do that like you do now using the Shopify order editing function. Because obviously, you wouldn't see clearly what the price of each item is, you would just see all zeros and the entirety of the order price in the "Cent" product line item. If you wanted to process a partial refund, you would have to calculate the price to refund yourself, edit the quantity of the "Cent" product to lower the price appropriately and remove the other line items as necessary, for accurate tracking purposes. You'd have to do something similar to increase any line item quantities and generate an invoice.

Not to worry though, this was not a good solution now either with bold options, because if you had a complicated order, with a whole lot of interconnected line items lying around, and you had to remove a bundle from the order, while there is another pack or something on the order, I'm guessing it could be a headache to remove the correct line items. 

So the solution is that when you want to edit the order, you will instead click on "Edit Order" from the "More actions..." menu and it will be managed from a page in the app. This will essentially give you the same view the customer had on their cart, so you can remove items, or adjust down or up the item quantities from a high-level, not having to worry about low-level line items - and the app would manage everything seamlessly. Problem solved.

## Option Types

In the current setup you have standard Shopify options and Bold options as an addon to them. In the new system, these are merged into one system and we don't use Shopify options at all. Each product simply has a list of options, and each option has a `type`. The types available are:

* **buttons** (default)

  This looks like the current input for Shopify options, with the first button selected by default. Each item can either have a fixed value, or a custom value with a configurable `caption` and `range`. Only this type of option can be set as *trackable* with the field `track` set to `true`, which automatically generates variants for each combination of trackable choices - more on this below.

  Here is the configuration for the `nicotine` option of the `Handcrafted Salts` collection as an example:

  ```yaml
  - id: nicotine
    caption: Nicotine
    track: true
    items: 
      - id: mg15
        caption: 15MG
        value: 15
      - id: mg30
        caption: 30MG
        value: 30
      - id: mg50
        caption: 50MG
        value: 50
      - id: custom
        caption: CUSTOM
        value:
          type: integer
          caption: Enter Any Value (0-50)
          range: 0-50
  ```

  Note that `type` is not set, and defaults to *buttons*.

* **checkbox**

  Can be on or off, used for addons. Has an `included` field which can be set to `true` in which case the checkbox is selected and cannot be unselected - used for the `mysteryFlavor` option on flavor packs and the `usbCCable` option on the `Caliburn A2 Starter Bundle` product.

  Here is the configuration for the `agentCool` option of the `Handcrafted Salts` collection as an example:

  ```yaml
  - id: agentCool
    type: checkbox
    caption: Include Agent Cool? ({{price}})
    description: Easily add the perfect level of cooling to any flavor
    link: agentCool.size.ml15
  ```

* **checkbox-group**

  Similar to **checkbox**, but a group of checkboxes - the indivual checkboxes act as individual items of the group option - the exception being that for this option type multiple items can be selected instead of just one.

  Here is the configuration for the `addons` option of the `Signature E-Juice` collection as an example:

  ```yaml
  - id: addons
    type: checkbox-group
    caption: Add-ons
    items:
      - id: sweet
        caption: +Sweet (Sucralose)
      - id: sour
        caption: +Sour (Malic)
      - id: cool
        caption: +Cool (Menthol)
        conditional: not withExtraCool
  ```

* **select**

  A drop down menu.

  Here is the configuration for the `extraFlavoring` option of the `Signature E-Juice` collection as an example:

  ```yaml
  - id: extraFlavoring
    type: select
    caption: Add Extra Flavoring
    items:
      - id: perc10
        caption: +10% Flavor
      - id: perc20
        caption: +20% Flavor
      - id: perc30
        caption: +30% Flavor
      - id: perc40
        caption: +40% Flavor
  ```

* **variant-select**

  A drop down menu of variants pulled explicitly from other products - this is used for example for the `deal` option for the `Caliburn A2 Replacement Pods` product. This option automatically connects the selected variant to the cart item. 

  Has a `multiple` field which can be set to `true` - in which case the customer can set the quantity they want of the variant - used for example for the `replacementPods` option for the `Caliburn A2 Device` product.

  You'll notice an ellipsis (...) in front of product IDs (configured placeholders) in the `items` list for options of this type - the ellipsis means "pull all tracked non-custom variants from this product". You'll also notice an `itemCaption` field which is expected to be a Javascript function which given the `product` and `variant` objects, should return an appropriate caption for the corresponding drop menu item.

  Here is the configuration for the `deal` option of the `Caliburn A2 Replacement Pods` product as an example:

  ```yaml
    - id: deal
      type: variant-select
      caption: --Optional-- Caliburn Optimized Flavor Deal
      description: Limited one flavor deal per order.
      items:
        - ...countryCrumbleSalt
        - ...freshMintSalt
        - ...monkeyJuiceSalt
        - ...sonomaGrapeSalt
      itemCaption: |
              ({product,variant}) => {
                    let ptitle = product.title;
                    if (ptitle.endsWith(" Salt")) ptitle = ptitle.substring(0,ptitle.length-5);
                    return `${ptitle} - ${variant.title}`;
              }
  ```

* **product-select**  

  A drop down menu of products (not variants) either set explicitly, or with a filter. A filter can be a combination of *by collection*, *include if tagged* and/or *exclude if tagged*. A `selectVariant` field is expected to be a Javascript function which given the selected `product` object should return which of its variants should be added to the cart item. This function also has access to the selected item for all other options, so it can use these to decide which variant to select. This is done for the `flavors` option of the `Build-Your-Own Pack` product - the correct variant of the selected `Handcrafted Salt` product is selected based on the `nicotine` option just above.

  Has a `multi-select` field which can be set with a `min` and `max` so the user must select at least `min` number of products and at most `max` number of products. If the `show-all` sub-field is set to `true`, `max` drop down menus will be shown, otherwise only `min` drop down menus will be shown and a new one will appear if necessary whenever all the ones visible have been selected.

  Here is the configuration for the `flavors` option of the `Build-Your-Own Pack` product as an example:

  ```yaml
  - id: flavors
    type: product-select
    filters:
      - type: collection
        id: handcrafted-salts
      #- type: include-if-tagged
      #  tags: 
      #    - some-tag
      #- type: exclude-if-tagged
      #  tags: 
      #    - some-tag
    sort: alpha-asc
    selectVariant: |
           ({product,options}) => {
             return product.nicotine[options.nicotine.itemId];
           }
    multiSelect:
      min: 2
      max: 6
      captions:
        - Flavor 1*
        - Flavor 2*
        - Flavor 3
        - Flavor 4
        - Flavor 5
        - Flavor 6
  ```

## Pricing

Since variants do not have any pricing, the pricing is configured explicitly but separate (yet connected) to the options. Here is an example of how it is configured for the `Signature E-Juice` collection:

```yaml
price:
  size:
    ml30: 14.95
    ml60: 19.95
    ml120: 34.95
  custom: 2
  extraCool:
    single: 0.50
    double: 1
    triple: 1.50
  extraFlavoring:
    perc10: 1.25
    perc20: 1.75
    perc30: 2.25
    perc40: 2.75
  addons: 
    sweet: 0.75
    sour: 0.75
    cool: 0.75
```    
Now if you look at the options configured for the `Signature E-Juice` collection, you'll find that the `id` field for the options and their `items` map to the structure above. So if the `ml60` item is selected for the `size` option, *19.95* will count towards the price of the item. If the `sour` and `cool` addons are checked, then 2x`0.75` will count towards the price of the item, and so on. 

The only value above which doesn't directly map to an option is the `custom` value. This is because it can be "activated" by either of two separate option items: the `custom` item of the `nicotine` option and the `custom` item of the `vg` option. This is NOT done automatically because they have the same name, it has to be explicitly set using the `activatePrice` field on the items. So in this case, the two aforementioned items have the `activatePrice` field set to `custom` (the name used in the price structure). Note that, in this way a price can be activated by multiple items, but is only counted once.

The only time a price can be counted multiple times is for a **variant-select** option with the `multiple` field set to `true`.

This is how the price structure is configured for the `Handcrafted Salts` collection:

```yaml
price:
  base: 14.95
  nicotine:
    custom: 2
  agentCool: 7.95
```

The `base` price if configured always counts towards the item price and doesn't have to be activated by any option.

This is how the price structure is configured for the `Build-Your-Own Pack` product:

```yaml
price:
  base: 29.95
  flavors: |
        ({option}) => {
          const table = {
            3: 13.95,
            4: 25.90,
            5: 34.85,
            6: 40.80,
          };
          return table[option.multiSelect.selectedCount] || 0;
        }  
  replacementPods: 9.95
  caliburnA2DeviceKit: 19.95
  agentCool: 5
  sweetener: 5
```

The `flavors` option is a **product-select** option with `min` set to *2* and `max` set to *6*. The `flavors` price value is in this case a Javascript function instead of a constant. Based on the value for `option.multiSelect.selectedCount` it returns the correct price component.

## Linking

Only the **variant-select** and **product-select** option types automatically link to the selected variants - for other option types, where necessary this has to be done explicitly using the `link` field. This can either be a reference to a variant, a list of variant references or a Javascript function that returns a reference to a variant or an array of variant references.

This is how the `agentCool` option is configured for the `Handcrafted Salts` collection.

```yaml
  - id: agentCool
    type: checkbox
    caption: Include Agent Cool? ({{price}})
    description: Easily add the perfect level of cooling to any flavor
    link: agentCool.size.ml15
```

The link here refers to the `agentCool` product (a placeholder configured in the Global Options), its `size` option which has the `track` field set to `true`, and the `ml15` item variant. Note that the id of this option is also `agentCool` which coincides with the product placeholder, but they have nothing to do with each other, the id here could be something else.

If a product has more than one trackable option, such a `Retired Salts` which are tracked by both their `nicotine` and `size` options, the individual variants can be reference in either of the following ways for example:

* productName.nicotine.mg15.size.ml60
* productName.size.ml60.nicotine.mg15

Note that products that have no trackable options simply have a single default variant and can be reference by the product placeholder alone, such as for the `sweetner` option for the `Caliburn A2 Starter Bundle` product:

```yaml
  - id: sweetener
    type: checkbox
    caption: Add Sweetener?
    description: Have you ever wished a flavor was sweeter? Now you can control exactly how sweet you prefer your juice to taste. Our sweetener works with any e-liquid.
    link: sweetener
```

Note also that for options that are optional, such as **checkbox** and **select**, the link is only activated if a value is selected/checked.

For an example of an option which has a Javascript function as its `link` field, here is the `color` option of the `Caliburn A2 Starter Bundle` product:

```yaml
  - id: color
    caption: Color
    items:
      - id: specialEditionWhite
        caption: (SPECIAL EDITION) WHITE
      - id: aquaBlue
        caption: AQUA BLUE
      - id: arcticSilver
        caption: ARCTIC SILVER
      - id: black
        caption: BLACK
      - id: blue
        caption: BLUE
      - id: green
        caption: GREEN
      - id: grey
        caption: GREY
      - id: irisPurple
        caption: IRIS PURPLE
      - id: orange
        caption: ORANGE
    link: |
       ({option}) => {
         return caliburnA2Device.color[option.itemId];
       }
```

So basically here, the proper variant from the `caliburnA2Device` product (from the `color` option which is tracked) is returned based on the selection of the color on the bundle.

And another good example which returns an array of variants is the `flavorPack` option for the `Essential Salts Pack` product:

```yaml
- id: flavorPack
    caption: Flavor Pack
    items:
      - id: agentCool
        caption: AGENT COOL
      - id: mellowMixers
        caption: MELLOW MIXERS
      - id: summerHarvest
        caption: SUMMER HARVEST
      - id: sweetTreats
        caption: SWEET TREATS
    link: |
       ({options,option}) => {
         const packs = {
           agentCool: [
             blueRaspberrySalt,
             cucumberBreezeSalt,
             issaPearadiseSalt,
             vanillaMintSalt,
             theBestDamnMangoSalt
           ],
           mellowMixers: [
             cucumberBreezeSalt,
             countrySummerSalt,
             melonSmoothieSalt,
             monkeyJuiceSalt,
             sassafrasRootBeerSalt
           ],
           summerHarvest: [
             blueRaspberrySalt,
             cucumberBreezeSalt,
             issaPearadiseSalt,
             lycheeCandySalt,
             theBestDamnMangoSalt
           ],
           sweetTreats: [
             blueberryMuffinSalt,
             cerealFiendSalt,
             melonSmoothieSalt,
             monkeyJuiceSalt,
             countryCrumbleSalt           
           ]
         };
         return packs[option.itemId]
           .map((product) => product.nicotine[options.nicotine.itemId]);
       }
```

And finally, a `link` can also be configured directly for the product itself (and not just for its options). For example, for the `White Caliburn A2 Device` the following link is configured:

```yaml
link: caliburnA2Device.color.specialEditionWhite
```