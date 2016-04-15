Note serial numbers need to go on the ASN

### Supplier
Table - **suppliers**
* supplier_id
* name
* description
* region - for information purposes
* country - for information purposes
* overreceive_percentage - this is the percentage over what was ordered on the PO that the WH is allowed to receive and will round down. E.g. if set to 10% and order 100 then can receive 110 if the supplier over delivers. If order 1 then rounds down to 1 and can only receive 1. This overrides setting made at global level and Organisation level.


### Variations
Table - **variations**
* variation_id
* name
* min_days_to_expiry - if populated then is an expiry product and WMS will require entering expiry date on receiving and will not allow receiving item if the expiry date is less than this nr of days from receive date. Product will also have all expiry functionality in WH like FEFO (first in first out) as well as generating task for uplifting soon to expire product (days to expire configurable). This attribute sits here because so much in the system relies on this field so cannot be configurable and sit on variation_types table

### WH Variation Types
Will be populated with attributes driving the following WH behaviour. For example for Variations having the variation_type_id for 'POPUP_RECEIVING' a popup will come up when receiving the item with information for the user receiving, i.e. for books to put the book into a protective plastic sleeve. There are other Variation Types which will determine Locations for which Variation can be put away. For a Variation with the variation_type_id with the 'name' of 'HAZARDOUS' with the 'value' as 1 (true), the item will be directed toward Locations only which have that variation_type_id in the location_type_variation_type table for the Location Type for the Location - more on this in sections below. Variation Types will drive the following types of behaviour.
* determine the zone for put away - i.e. put away to high value and/or hazardous
* displaying a pop up message at receiving or packing 
* ability to create task splits based on these variation types (i.e. put away to a specific zone)
* determine equipment type for receiving and/or picking
* determine which pick wave type the variation (item) will be on

Table - **variation_types**
* variation_type_id
* name - for example 'POPUP_RECEIVING'/'HIGH_VALUE'/'HAZARDOUS'/'FAST_MOVER'/'FIXED'/'GENERAL' (Note 'GENERAL' is for CHAOTIC good stock put away and 'FIXED' is for one Supplier Variation for a location only (or Variation level if the CO_LOCATE_SUPPLIER_VARIATIONS is set to 1 (i.e. true)(will not store Variation))
* value - For 'name' 'POPUP_RECEIVING' could have this value as 'Requires book sleeve' (i.e. message to popup on receiving this Variation for book) - rest examples above are boolean with value of 0 (i.e. false) or 1 (i.e. true)
 
## Variation / Variation Types
Table - **variation_variation_types**
This is the link table between variation_types table and the variations table (which has a variation_id)
* variation_id
* variation_type_id

### Pack Sizes
Initially this would by default hold the following options (SINGLE;INNER;OUTER;TI;PALLET). On variation_pack_barcodes below, the quantity for each option can be stored with volumetrics (W/D/H/weight/volume) with one row in variation_pack_barcodes being required for a variation_id. As an example for Coke cans:
SINGLE - one can of Coke
INNER - one six pack of Coke
OUTER - one case of Coke (24 cans of Coke)
TI - six cases of coke to a layer on a pallet (6x24 cans of Coke)
PALLET - ten layers (this is known as the HI but not stored) of TI per pallet (6x24x10 cans of coke)

Table - **pack_sizes**
* pack_size_id
* name

### Variation Pack Barcodes
For a Variation (i.e. product) it is possible to have a barcode and volumetrics for each pack size (above) of the Variation. These could have the following options (SINGLE;INNER;OUTER;TI;PALLET). On validation on population of Item Master data there must be at least one variation_pack_barcodes row for a Variation (for e-commerce generally SINGLE at least). Populating a variation_pack_barcodes row for a variation_id with a pack_size_id for 'PALLET', for example, would allow scanning the pallet barcode on receiving and the received quantity would automatically be that for the pallet. Put away could also direct the pallet to a suitable zone for pallet storage for that Variations attributes (i.e. is_hazardous). Receiving an OUTER would do something similar but to a location that could store a case of Coke for example which is very different from storing the pallet 120x100x120cm).
It is configurable as to whether the WMS can have more than one barcode_id with the same barcode and variation_id.

Table - **variation_pack_barcodes**
* variation_pack_barcode_id
* variation_id
* pack_size_id
* barcode
* height_mm - I have added the dimension to the name because we have already had a dimensioning mess in TAL because of confusion on this point so better to include in the name of the variables so super transparent (integer)
* width_mm
* depth_mm
* weight_kg (decimal)

### Organisations
Holds the organisations held in the WH. This will include marketplace sellers as well as other companies who's stock is held and shipped through the WH. This means that supplier_variations for MP sellers and other organisations held in the WH will be held under the same Variation as those for the 'main' organisation. If he configuration CAN_SHARE_BARCODES is set to 1 (true) then all organisations can share the same set of barcodes. This means that it is not neccessary to re-barcode for other organisations - this is huge because no-one including TAL and Amazon have this right. The only way this can work is that the WMS knows who owns which stock and this is made possible because the WMS will not allow collocating supplier_variations with the same variation_d which do no share the same organisation_id **and** supplier_variation_id.

Table - **organisations**
* organisation_id
* name
* description
* over_receive_percentage - this is the percentage over what was ordered on the PO that the WH is allowed to receive and will round down. E.g. if set to 10% and order 100 then can receive 110 if the supplier over delivers. If order 1 then rounds down to 1 and can only receive 1. This overrides setting made at global level.

### Supplier Variations
This is the Variation for a specific supplier and organisation so will have the supplier_id, organisation_id as well as the variation_id of parent supplier agnostic variation.

Table - **supplier_variations**
* supplier_variation_id
* name
* description
* variation_id
* supplier_id
* organisation_id

### Purchase Order Types
Note purchase orders are always inbound into the WH. These would be (i.e. name) PO/IBT/CUST_RETURN/SUPPL_RETURN - possible to configure more types.

Table - **po_types**
* po_type_id
* name
* description - description of the above

### Purchase Orders (PO)
A purchase order for inbound supplier_variation items from a supplier into an organisation into a warehouse.

Table - **purchase_orders**
* po_id
* warehouse_id - i.e. the warehouse to receive the PO
* supplier_id
* organisation_id
* po_type_id
* ship_date - when supplier will ship to the WH  
* receive date - date when the WH will receive PO
* return_ref_nr - used in conjunction with po_type_id to hold either the customer return nr or the supplier return nr for an item returned back to the WH 
* sending_warehouse_id - populated for IBT po_type
* outbound_order_nr - this is populated only for the IBT po_type and is the outbound order type id of the related outbound IBT
* receive_instruction - a popup will come up displaying this message on receiving PO if this is populated - i.e. special instructions for receiving

### Purchase Order Details
Table - **purchase_order_details**
* po_detail_id
* po_id
* supplier_variation_id - note this is a combination of variation, supplier and organisation
* ordered_quantity
* cost_price

### Advanced Shipping Notifications (ASN)
A supplier will send an ASN against a PO and it indicates an updated ETA as well as what can be expected on the ASN. A PO might be delivered in more than one load by the Supplier and the ASN is what will be on that load with its ETA date/time. There can not be items on an ASN which are not on the original PO and there should not be more items (qty) in total across the ASN's than on the original PO. Whether to allow over receiving, and if so to what percentage, is a configuration in the WMS which can be set as default on Global Configurations or set for an Organisation or at a Supplier level with Supplier configuration overriding Organisation and Organisation overriding Global.

Table - asns
* asn_id
* po_detail_id
* quantity - quantity to expect - sum of qty's on all ASN's for PO for this Supplier Variation should not be more than PO qty - could validate on the ASN API from ERP or Supplier
* updated_eta_date - this is an updated delivery date from supplier, ASN is generated when they have packed the stock in the sending Supplier WH and are about to ship out so the qty and ETA will be more accurate than for the PO
* is_final - this field can be used if the supplier will not deliver everything on a PO. If not used then customer will default to false and receiving to full quantity will close PO or will need to be manually closed on the PO screen. 







# OLD Catalogue data structure - to be edited (copied form old WMS structure)
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

