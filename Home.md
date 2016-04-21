Note serial numbers need to go on the ASN

### Supplier
Table - **suppliers**
* supplier_id
* name
* description
* region - for information purposes
* country - for information purposes
* overreceive_percentage - this is the percentage over what was ordered on the PO that the WH is allowed to receive and will round down. E.g. if set to 10% and order 100 then can receive 110 if the supplier over delivers. If order 1 then rounds down to 1 and can only receive 1. This overrides setting made at global level and Organisation level.


### Items
This is product without size or colour, i.e. Asics Gel or Iphone 5 - barcode/s lives at this level (parent of Item Line).
Table - **items**
* item_id
* item_name
* item_description
* active    - to permanently inactivate an item line, all item_variant and supplier_item_variant children are inactive when parent item set inactive
* published    - to temporarily disable an item line, all item_variant and supplier_item_variant children are temporarily disabled when parent item set disabled
* min_days_to_expire_receive    - less than this number of day till expiry (i.e. 180 days) and the system will not allow receiving. Also if populated will require entering the expiry date at receiving and will then track that in WH. Will not allow co-locating same supplier_variant item in WH with different expiry dates (for his to work putaway will need to direct same supplier_variant item with same expiry date to location where there is space and where there is already that item for the same expiry date). This captured date will drive FEFO (first expire is first out) picking as well as tracking stock about to expire or has expired in WH. 
* min_days_to_expire_pick    - Will not allow picking stock within a certain time of expiry (i.e 10 days). Could also be used to trigger picking out stock past this number days to expire

### WH Item Attributes
Will be populated with attributes driving the following WH behaviour. Item Attributes will determine Locations for which variant can be put away. For a variant with the item_attribute_id with the 'name' of 'HAZARDOUS' the item will be directed toward Locations only which have that variant_attribute_id in the location_type_item_attribute table for the Location Type for the Location - more on this in sections below. 

Table - **item_attributes**
* item_attribute_id
* name - for example 'HIGH_VALUE'/'HAZARDOUS'/'FAST_MOVER'/'FIXED'/'GENERAL' (Note 'GENERAL' is for CHAOTIC good stock put away and 'FIXED' is for one Variant/(Supplier Variant CO_LOCATE_SUPPLIER_VARIANTS = 0 (false)) for a location only (or variant level if the CO_LOCATE_SUPPLIER_VARIANTS is set to 1 (i.e. true)(will not store Variant))
 
## Item/ Item Attributes Types
Table - **item_item_attributes**
This is the link table between variant_attributes table and the variants table (which has a variant_id)
* item_id
* item_attibute_id

### Pack Sizes
Initially this would by default hold the following options (SINGLE;INNER;OUTER;PALLET_LAYER;PALLET). On variant_pack_barcodes below, the quantity for each option can be stored with volumetrics (W/D/H/weight/volume) with one row in variant_pack_barcodes being required for a variant_id. As an example for Coke cans:
SINGLE - one can of Coke
INNER - one six pack of Coke
OUTER - one case of Coke (24 cans of Coke)
TI - six cases of coke to a layer on a pallet (6x24 cans of Coke)
PALLET - ten layers (this is known as the HI but not stored) of TI per pallet (6x24x10 cans of coke)

Table - **pack_sizes**
* pack_size_id
* name

### Item Pack Sizes
For an Item it is possible to have a volumetric for each pack size (above) of the Item. These could have the following options (SINGLE;INNER;OUTER;PALLET_LAYER;PALLET). On validation on population of Item Master data there must be at least one item_pack_size row for a variant (for e-commerce generally SINGLE at least). Populating a item_pack_size row for an item_id with a pack_size_id for 'PALLET', for example, would allow scanning the pallet barcode (see Variant Pack Barcodes) on receiving and the received quantity would automatically be that for the pallet. Put away could also direct the pallet to a suitable zone for pallet storage for that Items attributes (i.e. HAZARDOUS). Receiving an OUTER would do something similar but to a location that could store a case of Coke for example which is very different from storing the pallet 120x100x120cm).

Table - **item_pack_sizes**
* item_pack_size_id
* item_id
* pack_size_id
* height_mm - I have added the dimension to the name because we have already had a dimensioning mess in TAL because of confusion on this point so better to include in the name of the variables so super transparent (integer)
* width_mm
* length_mm - validation that this is greater or equal or equal to other dims because will be used to determine if an item can fit into a location. The location will also have a length as the maximum variable
* weight_kg (decimal)

### Variants
Table - **variants**
* variant_id
* name
* min_days_to_expiry - if populated then is an expiry product and WMS will require entering expiry date on receiving and will not allow receiving item if the expiry date is less than this nr of days from receive date. Product will also have all expiry functionality in WH like FEFO (first in first out) as well as generating task for uplifting soon to expire product (days to expire configurable). This attribute sits here because so much in the system relies on this field so cannot be configurable and sit on variant_types table
* is_parent_assortment -  If this flag is true and this will hold the generic supplier parent barcode. NB only one Variant or an Item can hold the 'is_parent_assortment' flag as true.

### Variant Pack Barcodes
For a variant it is possible to have a barcode for each pack size (above) of the variant. These could have the following options (SINGLE;INNER;OUTER;PALLET_LAYER;PALLET). On validation on population of Item Master data there must be at least one variant_pack_barcodes row for each Item Pack Size.
It is configurable as to whether the a barcode_id (i.e. barcode) can exist for more than one variant.(CAN_SHARE_BARCODES) System to validate that a barcode (barcode_id) cannot exist for two different pack_size_id's for the

Table - **variant_pack_barcodes**
* variant_pack_barcode_id
* item_pack_size_id
* variant_id
* barcode_id

### Organisations
Holds the organisations held in the WH. This will include marketplace sellers as well as other companies who's stock is held and shipped through the WH. This means that supplier_variants for MP sellers and other organisations held in the WH will be held under the same variants as those for the 'main' organisation. If he configuration CAN_SHARE_BARCODES is set to 1 (true) then all organisations can share the same set of barcodes. This means that it is not neccessary to re-barcode for other organisations - this is huge because no-one including TAL and Amazon have this right. The only way this can work is that the WMS knows who owns which stock and this is made possible because the WMS will not allow collocating supplier_variants with the same variant_id which do no share the same organisation_id **and** supplier_variant_id.

Table - **organisations**
* organisation_id
* name
* description
* over_receive_percentage - this is the percentage over what was ordered on the PO that the WH is allowed to receive and will round down. E.g. if set to 10% and order 100 then can receive 110 if the supplier over delivers. If order 1 then rounds down to 1 and can only receive 1. This overrides setting made at global level.

### Supplier Variants 
This is the Variant for a specific supplier and organisation so will have the supplier_id, organisation_id as well as the variant_id of parent supplier agnostic variant.

Table - **supplier_variants**
* supplier_variant_id
* name
* description
* variant_id
* supplier_id
* organisation_id

### Purchase Order Types
Note purchase orders are always inbound into the WH. These would be (i.e. name) PO/IBT/CUST_RETURN/SUPPL_RETURN - possible to configure more types.

Table - **po_types**
* po_type_id
* name
* description - description of the above

### Purchase Orders (PO)
A purchase order for inbound supplier_variant items from a supplier into an organisation into a warehouse.

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
* supplier_variant_id - note this is a combination of variant, supplier and organisation
* ordered_quantity
* cost_price

### Advanced Shipping Notifications (ASN)
A supplier will send an ASN against a PO and it indicates an updated ETA as well as what can be expected on the ASN. A PO might be delivered in more than one load by the Supplier and the ASN is what will be on that load with its ETA date/time. There can not be items on an ASN which are not on the original PO and there should not be more items (qty) in total across the ASN's than on the original PO. Whether to allow over receiving, and if so to what percentage, is a configuration in the WMS which can be set as default on Global Configurations or set for an Organisation or at a Supplier level with Supplier configuration overriding Organisation and Organisation overriding Global.

Table - asns
* asn_id
* po_detail_id
* quantity - quantity to expect - sum of qty's on all ASN's for PO for this Supplier Variant should not be more than PO qty - could validate on the ASN API from ERP or Supplier
* updated_eta_date - this is an updated delivery date from supplier, ASN is generated when they have packed the stock in the sending Supplier WH and are about to ship out so the qty and ETA will be more accurate than for the PO
* is_final - this field can be used if the supplier will not deliver everything on a PO. If not used then customer will default to false and receiving to full quantity will close PO or will need to be manually closed on the PO screen. 

### Outbound Order Types
Outbound order types will include EXPRESS/NORMAL/SUPPLIER_RETURN/REPLACEMENT/IBT/COLLECT - additional options can be configured.

Table - **outbound_order_types**
* outbound_order_type_id
* name
* description

### Outbound Orders
An outbound order is for a warehouse and an organisation but could be across suppliers and supplier_variants.

Table - **customer_orders**
* customer_order_id
* warehouse_id - the warehouse from which the outbound order is shipping
* organisation_id
* outbound_order_type_id
* inbound_po_nr - for an IBT this is the related inbound PO nr
* ship_date - the date on which the order must ship
* destination_warehouse_id - in the case of an IBT this is the warehouse to which the IBT (inter branch transfer) is going - NULL if not an IBT order type
* cust_name - these fields are for printing shipping labels and packing slips
* cust_address_1
* cust_address_2
* cust_address_3
* cust_zip_postal_code
* cust_country
* return_ref_nr - this is the customer return reference nr to be populated only for replacement or return of repaired item
* cust_contact_nr
* cust_email
* courier - this is the courier to be printed on the shipping label. Will be determined by the ERP syste based on if the customer has selected an EXPRESS order type option (it should be possible to add multiple shipping options to outbound_order_types (currently EXPRESS and NORMAL only)) 
* pack_instruction - comes up at packing as a popup special instruction (example add an insert pamphlet) if this is populated
* ship_cost
* delivery_instruction - for courier in the event that this needs to be printed

### Outbound Order Details
Table - **outbound_order_details**
* outbound_order_detail_id
* supplier_variant_id
* quantity
* cost_price_incl_vat - for reporting
* sell_price_ex_vat - each for printing invoice or packing slip
* sell_price_incl_vat - each

### Warehouse
Note these are the WH's of the WMS not the supplier. 
Note that supplier_warehouses (holds lead time for supplier to a WMS warehouse) and supplier_variant_warehouses (holds lead time for a specific supplier variant to a WMS warehouse (overrides lead time held at supplier level)) need to move out of the WMS DB and into the Catalogue DB. This is because the lead time (expected time to deliver after order is placed) on PO is calculated in the ERP system and the PO is sent to the WMS with the ETA date already calculated.
Table - **warehouses**
* warehouse_id
* name
* description
* geo_lat    - latitude
* long       - longitude
* address_1
* address_2
* address_3
* city
* ziporpostal_code
* stateprovincecounty
* countryid



