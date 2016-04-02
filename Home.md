# Catalogue data structure - to be edited (copied form old WMS structure)
There three levels of catalogue for a product 'often used':

### Supplier
Table - **supplier**
* supplier_id
* supplier_name
* supplier_description
* supplier_lead_time_hours
 
### Field Type
Table - **field_type**
* field_type_id
* field_type_name    - this would be ([0:int],[1:string],[2:boolean],[3,decimal])

### Items
This is product without size or colour, i.e. Asics Gel or Iphone 5 - barcode/s lives at this level (parent of Item Line).
Table - **items**
* item_id
* item_name
* item_description
* active    - to permanenly inactivate an item line, all item_variation and supplier_item_variation children are inactive when parent item set inactive
* published    - to temporarily disable an item line, all item_variation and supplier_item_variation children are temporarily disabled when parent item set disabled
* min_days_to_expire_receive    - less than this number of day till expiry (i.e. 180 days) and the system will not allow receiving. Also if populated will require entering the expiry date at receiving and will then track that in WH. Will not allow co-locating same supplier_variation item in WH with different expiry dates (for his to work putaway will need to direct same supplier_variation item with same expiry date to location where there is space and where there is already that item for the same expiry date). This captured date will drive FEFO (first expire is first out) picking as well as tracking stock about to expire or has expired in WH. 
* min_days_to_expire_pick    - Will not allow picking stock within a certain time of expiry (i.e 10 days). Could also be used to trigger picking out stock past this number days to expire
* bulky_level    - 0 is not bulky, can then have levels 1 through ? to indicate how bulky. A weber braai will not go through a dimensioning/weighing scanner at packing but does not need a fork lift for example. A double door fridge will need a special storage area and a forklift. These indications of the size or weight of items will be imporant in directing their flow within the WH i.t.o. storage/packing/equipment types/etc.
* hazardous_level - 0 is not hazardous, 1 slightly hazardous (i.e. batteries may not fly), 2 properly hazardous suh as gas cylinders, pool chemicals, fire lighters etc which require special storage because are dangerous/flamable/cannot be kept near food/etc. Very important for directing storage locations, equipment types and handling for safety, spoilage of other product like food which cannot be stored near chlorine as well as delivery/transport options
* is_parent_assortment    - this is a parent assortment item, not populated for children assortment items. This will drive receiving of these items in the WH
* parent_assortment_item_id    - if is a child of a paren assortment item then populated.This will drive receiving of these items in the WH
* high_value   - 0 is no, 1 is yes and will need toe stored in the high value cage
* serialised    - 0 is no, 1 is yes, i.e. will require serial number capture at receiving so can track which serial nr belongs to which supplier. Will also need to track which serial number ships so know which serial nr customer gets so if they do a return can nesure have returned the item sold.

### Item Barcodes
Table - **barcodes**
* barcode_id
* barcode
* item_id

### Item Variations
This is the product with size and colour but without any pricing, lead time or supplier information, Asics Gel Green& Black size 9 or Iphone 5 Black 6GB (child of Product Line and parent of Item). The variation is driven by the four group value fields group_1_value, group_2_value, group_3_value and group_4_value (see description below)
Table - **variations**
* item_variation_id
* item_variation_name
* item_variation_description
* group_1_value    
* => this is required if there is more than one variation for an item
* => for Iphone 6 this might be the variations of storage capacity, i.e. 8 GB, 16GB, 32 GB.
* => the values stored against the group (i.e. group 1) will be displayed together in a group selection on the site.
* => if there is only one group with values then the selection made will determine which variation is selected.
* => there is no reference to this value in any other table and although this might not be best practises it is a good idea for the following reasons
* ==> if someone changed black to white in the reference table it would affect all products referencing that value and customers expecting black would get white items. One might ask who would do this, Jackie will explain
* ==> it would be possible to add validation that if any variation is referencing a variation attribute in an external attributes table that it cannot be deleted. The converse will not be the case so that if an attribute is no longer used it will not be deleted. As such this table will grow and where you have hundreds of thousands of products, where you have churn on those where some fall out and new ones replace them, it becomes unmanageable.
* ==> with so many values in this external table making selections becomes problematic, especially for weight/volume attributes where there are so many variations (i.e. 1l 1 litre, 1liter, 1litre, 1 liter, 1  liter, 1000ml, 1000 ml, 1 000ml, 1 000 ml, 1000 milliliters, and so it goes on)
* ==> this would put a lot of uneccessary load on the DB where you have the huge varariations table referencing a huge table storing links the attributes table and then of course to the attributes table. It also makes queries more complex and is another point of failure. If someone does something stupid (they will) on one variation then that one breaks. If someone does something stupid on a reference attribute table you suddenly have chaos and it is not always self evident where this problem came from without serious investigation. This works where you have clever business analysts sitting doing this all day but not for the model where you want minimum services interaction with the customers we sell the solution to
* => there is no name for the group on the site (ecomm site), i.e. we do not try store that a group is for GB for the Iphone 6, that is self evident and can be explained further in the descriptionsif there is confusion
* => if this group (1) is populated for any variation for the item then validation will force entering a group_1_value for all variations. This is the only tricky part and will have to be done as a popup which pops up on this validation fail (i.e. one add a second variation and try to save but do not have a group_1_value). On this validaiton fail all variations are displayed and the user can enter all the group_1_value's
* group_2_value
* => this is not required but as soon as is populated for one variation must be populated for all as above. Because this is not required, on a validation failure where the popup comes up (as described for group_1_value) this would allow for adding values for all the variations or clearing the group_2_value which caused the validation fail.
* => will form the second selection group for the item so in the case of the Iphone 6 example above this might hold values of Black, White and Gold. This would mean that there would be 9 variations for each combination of 8/16/32GB and Black/White/Gold with two selection boxes on the site for each of the groups. A selection would be required for both and that selection driving the variation displayed along with its price, availability, picture and description
* group_3_value - see group_2_value
* group_4_value - see group_2_value
* item_id

### Supplier Item Variations
This is the item as per the parent Item Variation but for a specific supplier so will have  pricing, lead time and supplier information (child of Item Variation).
Table - **supplier_variations**
* supplier_variation_id
* supplier_variation_name
* supplier_variation_description
* variation_id
* supplier_id
* default_lead_time_days    - note that this will override 'default_lead_time_days' set on supplier table
* default_lead_time_hours    - note that this will override 'default_lead_time_hours' set on supplier table

### Warehouse
Table - **warehouse**
* warehouse_id
* warehouse_name
* warehouse_description
* warehouse_geo_lat    - latitude
* warehouse_long       - longitude
* warehouse_address_1
* warehouse_address_2
* warehouse_address_3
* warehouse_city
* warehouse_ziporpostal_code
* warehouse_stateprovincecounty
* countryid

## Item Types
Will be populated with Style/Color/Weight/Size initially
Table - **item_types**
* id_item_type
* item_type_name

In item_types will populate with fields which will drive how the item displays on the site. So if 

## Item / Item Types
Table - **item_item_types**
This is the link table between item_types table and items table
* item_id
* item_type_id
* created_at
* updted_at

## Item Attributes

### Attributes
Table - **attribute**
* idAttribute
* attributeName - i.e. Expires

### ItemAttributes (i.e. hazardous, bulky)
Table - **item_attribute**
* item_attribute_id
* item_id
* attribute_id
* valueName - i.e. minDaysToExpiry
* value - i.e. min nr days to expiry date for which receiving is allowed for expiry products
* field_type_id   - see field_typetable above, this specifies if field is of type int, boolean, string or decimal

For examples of Attributes see relevant sections, i.e. in the Inbound section will find a section on receiving items which have the 'Expires' or 'Assortment' attribute. It will be discussed how the receiving process is wizard driven for the children of Items which bare these attributes.

## WH Items
* Items sent to the WH will have the SupplierItemVariationId and the WH will operate at this level (i.e. at the lowest level above).
* In order to distinguish between the same Item Variation from different Suppliers (i.e. different SupplierItemVariationId's with the same itemVariationId) the WH will not allow co-locating different SupplierItemVariationId's with the same itemVariationId and as such the will need to go down as well on the Item Master feed on the Supplier Item Variation feed.
* Where the WH is unable to determine which Supplier Item Variation applies for a barcode scanned the system is to give the user the choice. This might be when finding an item in the WH. 
When there are two different Supplier Item Variations for the same Item Variation in an OBI then these are not allowed into the same Outbound Container in case one shorts at packing (one of the Items fell out) or is damaged and then it is still possible to tell which item is damaged, expired or short. This is especially important for items with serial nrs

## Item Master Files
These are sent ahead of the PO to ensure that all items on the PO will be in the WMS with latest updates.
### Load
* The load here can be high when there is a PO (note non-IBT PO is always to one supplier) with thousands of Items for a Supplier, such as is the case with books ordered during the day which are then ordered at the end of the day or the next morning on lead time from a supplier. 
* In addition it may be necessary to push a bulk load of Item Master data down to the WH such as when first loading the Catalogue into the WH when taking on a new customer, customer WH or new Company for existing customer, i.e. Takealot taking on Superbalist. A different API might be necessary because here you may need to load hundreds of thousands of Items into the WMS.

### Supplier
Supplier file sent 1st ahead of the PO. There is one Supplier per PO.
* supplier_id
* supplier_name
* supplier_description
* supplier_lead_time_days
* supplier_lead_time_hours
* action (i.e. Create/Update/Delete -> in this case Create)

On receiving this message with the 'Create' messageType validate that the idSupplier does not exist already in the suppliers table. If it does update supplierName otherwise create a new row. If the supplierName exists for another idSupplier return an error. 

### Items (Table - supplier_item_variation - see above)
Send 2nd after Supplier message ahead of the PO message, a row per item on the PO.
* idSupplierItemVariation
* idItemVariation - Need this to make sure do not co-locate SupplierItemVariations which have same idItemVariation, i.e. same Item from different Suppliers
* itemName
* idItemAttribute - This will be stored in itemAttributes table
* idSupplier
* expiryDaysReceive - Nr of days or more till expiry date on expiry items, less and will not be able to receive item
* isSerialised - requires scanning the serial nr on receiving as well as on shipping
* action (i.e. Create/Update/Delete -> in this case Create)

On receiving this message with the 'Create' messageType validate that the idSupplierItemVariation does not exist already in the items table. If it does update itemName otherwise create a new row. If the itemName exists for another idItem return an error.

### ItemBarcode
Sent 3rd ahead of the PO, a row per barcode for this item. Note there can be multiple barcodes for an item and are all equivalent in the WH. A barcode cannot be shared between two items and sending a barcode into the system already belonging to another item will fail with appropriate message.
* idItemVariation
* barcode
* action (i.e. Create/Update/Delete -> in this case Create)(i.e. if we want to send message to system to remove barcode for an item so can get added for another item if backend system needed to do that)

On receiving this message with the 'Create' messageType validate that the barcode does not exist for another idItem, if it does return an error ('Barcode yyy already exists of idSupplierItemVariation xxx'). If the barcode already exists in the itemBarcodes table for the idItem then leave as is. If it does not then add for the idItem.
## Item Attributes
The important thing to remember with Item Attributes is that an item can both be a hazardous item which is also an expiry item, i.e. batteries. In addition having an Item Attribute will drive behaviour in the WH, i.e. you will not allow receiving items which will expire in less than the 'minDaysToExpiry'
Note on items table would have idItemAttribute


