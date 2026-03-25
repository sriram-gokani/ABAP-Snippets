# ABAP Code Snippets
A collection of reusable SAP ABAP code snippets for all levels — Beginner to Advanced.
Maintained by Sriram Gokani | SAP ABAP Consultant at Accenture

---

## Who Is This For?

- Beginners learning ABAP for the first time
- Intermediate developers looking for quick references
- Advanced consultants needing ready-to-use templates
- Anyone preparing for SAP ABAP interviews

---

## Table of Contents

1. [Beginner Snippets](#1-beginner-snippets)
2. [Intermediate Snippets](#2-intermediate-snippets)
3. [Advanced Snippets](#3-advanced-snippets)
4. [S/4 HANA and Modern ABAP](#4-s4-hana-and-modern-abap)
5. [Performance Tips](#5-performance-tips)

---

## 1. Beginner Snippets

### 1.1 Hello World
```abap
REPORT z_hello_world.
WRITE: / 'Hello, SAP World!'.
```

### 1.2 Internal Table with Work Area
```abap
REPORT z_internal_table.

TYPES: BEGIN OF ty_employee,
         emp_id   TYPE numc10,
         name     TYPE char50,
         salary   TYPE p DECIMALS 2,
       END OF ty_employee.

DATA: lt_employees TYPE TABLE OF ty_employee,
      ls_employee  TYPE ty_employee.

" Adding data
ls_employee-emp_id = '0000000001'.
ls_employee-name   = 'Sriram Gokani'.
ls_employee-salary = '75000.00'.
APPEND ls_employee TO lt_employees.

" Looping through table
LOOP AT lt_employees INTO ls_employee.
  WRITE: / ls_employee-emp_id, ls_employee-name, ls_employee-salary.
ENDLOOP.
```

### 1.3 SELECT from Database Table
```abap
REPORT z_select_example.

DATA: lt_mara TYPE TABLE OF mara,
      ls_mara TYPE mara.

SELECT matnr mtart maktx
  INTO TABLE @lt_mara
  FROM mara
  UP TO 10 ROWS.

LOOP AT lt_mara INTO ls_mara.
  WRITE: / ls_mara-matnr.
ENDLOOP.
```

### 1.4 IF / ELSE / ENDIF
```abap
DATA: lv_score TYPE i VALUE 85.

IF lv_score >= 90.
  WRITE: / 'Grade: A'.
ELSEIF lv_score >= 75.
  WRITE: / 'Grade: B'.
ELSE.
  WRITE: / 'Grade: C'.
ENDIF.
```

### 1.5 Simple ALV Report
```abap
REPORT z_simple_alv.

DATA: lt_mara    TYPE TABLE OF mara,
      lo_alv     TYPE REF TO cl_salv_table,
      lx_error   TYPE REF TO cx_salv_msg.

SELECT * INTO TABLE @lt_mara FROM mara UP TO 20 ROWS.

TRY.
  cl_salv_table=>factory(
    IMPORTING r_salv_table = lo_alv
    CHANGING  t_table      = lt_mara ).
  lo_alv->display( ).
CATCH cx_salv_msg INTO lx_error.
  MESSAGE lx_error->get_text( ) TYPE 'E'.
ENDTRY.
```

---

## 2. Intermediate Snippets

### 2.1 BAPI Call with Error Handling
```abap
REPORT z_bapi_example.

DATA: ls_headdata   TYPE bapiorders1,
      lt_return     TYPE TABLE OF bapiret2,
      ls_return     TYPE bapiret2,
      lv_order_no   TYPE bapivbeln-vbeln.

ls_headdata-doc_type   = 'TA'.
ls_headdata-sales_org  = '1000'.
ls_headdata-distr_chan  = '10'.
ls_headdata-division   = '00'.

CALL FUNCTION 'BAPI_SALESORDER_CREATEFROMDAT2'
  EXPORTING
    order_header_in = ls_headdata
  IMPORTING
    salesdocument   = lv_order_no
  TABLES
    return          = lt_return.

LOOP AT lt_return INTO ls_return WHERE type = 'E'.
  MESSAGE ls_return-message TYPE 'E'.
ENDLOOP.

IF lv_order_no IS NOT INITIAL.
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING wait = 'X'.
  WRITE: / 'Order Created:', lv_order_no.
ENDIF.
```

### 2.2 BDC Recording Template
```abap
REPORT z_bdc_example.

DATA: lt_bdcdata TYPE TABLE OF bdcdata,
      ls_bdcdata TYPE bdcdata,
      lt_message TYPE TABLE OF bdcmsgcoll,
      ls_message TYPE bdcmsgcoll.

" Fill screen data
DEFINE bdc_field.
  CLEAR ls_bdcdata.
  ls_bdcdata-fnam = &1.
  ls_bdcdata-fval = &2.
  APPEND ls_bdcdata TO lt_bdcdata.
END-OF-DEFINITION.

DEFINE bdc_dynpro.
  CLEAR ls_bdcdata.
  ls_bdcdata-program  = &1.
  ls_bdcdata-dynpro   = &2.
  ls_bdcdata-dynbegin = 'X'.
  APPEND ls_bdcdata TO lt_bdcdata.
END-OF-DEFINITION.

bdc_dynpro 'SAPLMGMM' '0060'.
bdc_field  'RMMG1-MATNR' '000000000000100001'.
bdc_field  'BDC_OKCODE'  '/00'.

CALL TRANSACTION 'MM02'
  USING lt_bdcdata
  MODE  'N'
  MESSAGES INTO lt_message.

LOOP AT lt_message INTO ls_message WHERE msgtyp = 'E'.
  WRITE: / ls_message-msgtx.
ENDLOOP.
```

### 2.3 IDoc Basic Trigger
```abap
REPORT z_idoc_trigger.

DATA: lv_docnum  TYPE edidc-docnum,
      ls_control TYPE edidc,
      lt_edidd   TYPE TABLE OF edidd,
      lt_return  TYPE TABLE OF bdwfreturn.

ls_control-mestyp  = 'ORDERS'.
ls_control-rcvprt  = 'LS'.
ls_control-rcvprn  = 'RECEIVER'.
ls_control-sndprt  = 'LS'.
ls_control-sndprn  = 'SENDER'.

CALL FUNCTION 'MASTER_IDOC_DISTRIBUTE'
  EXPORTING
    master_idoc_control            = ls_control
  TABLES
    communication_idoc_rec_list    = lt_return
    master_idoc_data               = lt_edidd.

WRITE: / 'IDoc triggered successfully'.
```

### 2.4 Enhancement via BAdI
```abap
" BAdI Implementation Example
" BAdI: ME_PROCESS_PO_CUST (Purchase Order)

METHOD if_ex_me_process_po_cust~process_item.

  DATA: ls_mepoitem TYPE mepoitem.

  ls_mepoitem = im_item->get_data( ).

  " Custom validation: block items with no material group
  IF ls_mepoitem-matkl IS INITIAL.
    RAISE EXCEPTION TYPE cx_me_my_exception
      EXPORTING
        textid = cx_me_my_exception=>no_material_group.
  ENDIF.

ENDMETHOD.
```

---

## 3. Advanced Snippets

### 3.1 OOP ABAP - Class and Interface
```abap
REPORT z_oop_example.

" Interface definition
INTERFACE lif_vehicle.
  METHODS: get_speed RETURNING VALUE(rv_speed) TYPE i,
           accelerate IMPORTING iv_amount TYPE i.
ENDINTERFACE.

" Class definition
CLASS lcl_car DEFINITION.
  PUBLIC SECTION.
    INTERFACES lif_vehicle.
    METHODS constructor IMPORTING iv_model TYPE string.
  PRIVATE SECTION.
    DATA: mv_model TYPE string,
          mv_speed TYPE i.
ENDCLASS.

CLASS lcl_car IMPLEMENTATION.
  METHOD constructor.
    mv_model = iv_model.
    mv_speed = 0.
  ENDMETHOD.

  METHOD lif_vehicle~get_speed.
    rv_speed = mv_speed.
  ENDMETHOD.

  METHOD lif_vehicle~accelerate.
    mv_speed = mv_speed + iv_amount.
  ENDMETHOD.
ENDCLASS.

" Main program
START-OF-SELECTION.
  DATA: lo_car TYPE REF TO lcl_car.
  CREATE OBJECT lo_car EXPORTING iv_model = 'Kia Seltos'.
  lo_car->lif_vehicle~accelerate( 60 ).
  WRITE: / 'Speed:', lo_car->lif_vehicle~get_speed( ).
```

### 3.2 RFC Function Module Template
```abap
FUNCTION z_rfc_get_employee_data.
*"----------------------------------------------------------
*"  IMPORTING
*"     VALUE(IV_EMPID) TYPE PERNR_D
*"  EXPORTING
*"     VALUE(ES_EMPLOYEE) TYPE ZST_EMPLOYEE
*"  EXCEPTIONS
*"     NOT_FOUND = 1
*"----------------------------------------------------------

  SELECT SINGLE pernr ename
    INTO @DATA(ls_emp)
    FROM pa0001
    WHERE pernr = @iv_empid
      AND endda >= @sy-datum.

  IF sy-subrc <> 0.
    RAISE not_found.
  ENDIF.

  es_employee-pernr = ls_emp-pernr.
  es_employee-ename = ls_emp-ename.

ENDFUNCTION.
```

### 3.3 Dynamic SELECT with Field List
```abap
REPORT z_dynamic_select.

DATA: lv_table  TYPE string VALUE 'MARA',
      lv_fields TYPE string VALUE 'MATNR,MTART,MBRSH',
      lt_result TYPE REF TO data,
      lo_table  TYPE REF TO cl_salv_table.

FIELD-SYMBOLS: <lt_result> TYPE STANDARD TABLE.

" Create dynamic table
cl_alv_table_create=>create_dynamic_table(
  EXPORTING it_fieldcatalog = VALUE lvc_t_fcat( )
  IMPORTING ep_table        = lt_result ).

ASSIGN lt_result->* TO <lt_result>.

SELECT (lv_fields)
  INTO TABLE @<lt_result>
  FROM (lv_table)
  UP TO 50 ROWS.

TRY.
  cl_salv_table=>factory(
    IMPORTING r_salv_table = lo_table
    CHANGING  t_table      = <lt_result> ).
  lo_table->display( ).
CATCH cx_salv_msg.
ENDTRY.
```

---

## 4. S/4 HANA and Modern ABAP

### 4.1 CDS View - Basic
```abap
@AbapCatalog.sqlViewName: 'ZVCUSTOMER'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Customer Basic Info'

define view Z_CDS_CUSTOMER
  as select from kna1
{
  key kunnr as CustomerNumber,
      name1 as CustomerName,
      land1 as Country,
      ort01 as City,
      pstlz as PostalCode
}
```

### 4.2 CDS View - With Association
```abap
@AbapCatalog.sqlViewName: 'ZVSALESORDER'
@EndUserText.label: 'Sales Order with Customer'

define view Z_CDS_SALES_ORDER
  as select from vbak
  association [0..1] to kna1 as _Customer
    on $projection.SoldToParty = _Customer.kunnr
{
  key vbeln  as SalesOrder,
      erdat  as CreatedOn,
      kunnr  as SoldToParty,
      netwr  as NetValue,
      waerk  as Currency,
      _Customer
}
```

### 4.3 ABAP 7.40 Inline Declarations
```abap
REPORT z_inline_declarations.

" Inline DATA declaration
SELECT * FROM mara INTO TABLE @DATA(lt_mara) UP TO 10 ROWS.

" Inline loop with filter
LOOP AT lt_mara INTO DATA(ls_mara) WHERE mtart = 'FERT'.
  WRITE: / ls_mara-matnr, ls_mara-mtart.
ENDLOOP.

" String templates
DATA(lv_message) = |Material { ls_mara-matnr } is of type { ls_mara-mtart }|.
WRITE: / lv_message.

" Conditional operator
DATA(lv_stock_status) = COND string(
  WHEN ls_mara-mstae = 'A' THEN 'Active'
  WHEN ls_mara-mstae = 'Z' THEN 'Blocked'
  ELSE 'Unknown' ).
WRITE: / lv_stock_status.
```

### 4.4 AMDP Method Template
```abap
CLASS zcl_amdp_example DEFINITION PUBLIC.
  PUBLIC SECTION.
    INTERFACES if_amdp_marker_hdb.

    TYPES: BEGIN OF ty_result,
             matnr TYPE mara-matnr,
             mtart TYPE mara-mtart,
             meins TYPE mara-meins,
           END OF ty_result,
           tt_result TYPE TABLE OF ty_result.

    CLASS-METHODS get_materials
      IMPORTING VALUE(iv_mtart) TYPE mara-mtart
      EXPORTING VALUE(et_result) TYPE tt_result
      RAISING cx_amdp_error.
ENDCLASS.

CLASS zcl_amdp_example IMPLEMENTATION.
  METHOD get_materials
    BY DATABASE PROCEDURE FOR HDB LANGUAGE SQLSCRIPT
    USING mara.

    et_result = SELECT matnr, mtart, meins
                FROM mara
                WHERE mtart = :iv_mtart;
  ENDMETHOD.
ENDCLASS.
```

---

## 5. Performance Tips

### 5.1 Use Field List Instead of SELECT *
```abap
" BAD - Never do this
SELECT * FROM mara INTO TABLE lt_mara.

" GOOD - Select only needed fields
SELECT matnr mtart meins
  INTO TABLE @lt_mara
  FROM mara.
```

### 5.2 Avoid SELECT Inside Loop
```abap
" BAD - SELECT inside LOOP hits database every iteration
LOOP AT lt_orders INTO ls_order.
  SELECT SINGLE * FROM kna1 INTO ls_customer
    WHERE kunnr = ls_order-kunnr.
ENDLOOP.

" GOOD - Use JOIN or FOR ALL ENTRIES
SELECT vbak~vbeln, kna1~name1
  INTO TABLE @DATA(lt_result)
  FROM vbak
  INNER JOIN kna1 ON vbak~kunnr = kna1~kunnr
  FOR ALL ENTRIES IN @lt_orders
  WHERE vbak~vbeln = @lt_orders-vbeln.
```

### 5.3 Use HASHED Table for Key Lookups
```abap
" For frequent READ TABLE with key - use HASHED table
DATA lt_mara TYPE HASHED TABLE OF mara
             WITH UNIQUE KEY matnr.

SELECT matnr mtart INTO TABLE @lt_mara FROM mara.

" Direct O(1) lookup
READ TABLE lt_mara INTO DATA(ls_mara)
  WITH TABLE KEY matnr = '000000000000100001'.
```

---

## How to Use This Repo

1. Browse the sections above
2. Copy the snippet you need
3. Adapt it to your requirement
4. Test in your sandbox before using in production

---

## Contributing

Found a bug or want to add your own snippet? Feel free to raise a Pull Request!

---

## Author

**Sriram Gokani**
SAP ABAP Consultant | 3+ years at Accenture
sriramgokarni.sap@gmail.com

---

## License

This project is open source and available under the MIT License.
