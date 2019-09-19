# SPRO-CUSTOMIZING-PATH
SPRO Node Path.


![Image1](https://github.com/vidyadharg/SPRO-CUSTOMIZING-PATH/blob/master/images/image1.png)
![Image2](https://github.com/vidyadharg/SPRO-CUSTOMIZING-PATH/blob/master/images/image1.png)

## CODE :
### 1.Include LSHI01F1I ( Form HANDLE_MENU_REQUEST, End)
```
ENHANCEMENT 1  ZFIIMP_CUSTOMIZING_PATH.    "active version
  IF sy-tcode EQ 'SPRO'.
    call method g_tree_data->menu->add_function
    exporting
      fcode = 'ZZSHOWPATH'
      text  = 'Show activity path'.
  ENDIF.
ENDENHANCEMENT.
```

### 2.Include LSHI01F1J ( Form HANDLE_MENU_SELECT, Start )
```
ENHANCEMENT 2  ZFIIMP_CUSTOMIZING_PATH.    "active version

TYPES :
  ty_string TYPE c LENGTH 5000.

DATA :
  lv_rc        TYPE i,
  lt_string    TYPE STANDARD TABLE OF ty_string,
  lw_string    TYPE ty_string.

 IF sy-tcode EQ 'SPRO'.

 CASE g_fcode.
    WHEN 'ZZSHOWPATH'.

      DATA: BEGIN OF zz,
            lv_nodekey TYPE tv_nodekey ,
            le_item TYPE shi_item,
            le_node TYPE treev_node,
            lv_relatkey TYPE tv_nodekey,
            lv_relatship_ant TYPE int4,
            html TYPE string,
            END OF zz.

      CHECK g_tree_data->tree IS NOT INITIAL.

      READ TABLE g_tree_data->nodes INTO zz-le_node WITH KEY node_key = g_tree_data->node_key  .
      CHECK sy-subrc = 0.

      READ TABLE g_tree_data->items INTO zz-le_item
        WITH KEY
          node_key = g_tree_data->node_key
          item_name = 'TEXT'.
      CHECK sy-subrc = 0.

      zz-html =  zz-le_item-text.

      zz-lv_relatkey  = zz-le_node-relatkey.
      zz-lv_relatship_ant = zz-le_node-relatship.

      WHILE zz-lv_relatkey IS NOT INITIAL.

        READ TABLE g_tree_data->nodes INTO zz-le_node WITH KEY node_key = zz-lv_relatkey .
        CHECK sy-subrc = 0.

        READ TABLE g_tree_data->items INTO zz-le_item 
          WITH KEY node_key = zz-le_node-node_key item_name = 'TEXT' .
        CHECK sy-subrc = 0.

        zz-lv_relatkey = zz-le_node-relatkey .

        IF zz-le_item-text IS NOT INITIAL AND zz-lv_relatship_ant = '4'.
          zz-html = zz-le_item-text && '->' && zz-html.
        ENDIF.
        zz-lv_relatship_ant = zz-le_node-relatship.
      ENDWHILE.


    IF lt_string IS INITIAL.
      lw_string = zz-html. APPEND lw_string TO lt_string.
    ENDIF.

    CALL METHOD cl_gui_frontend_services=>clipboard_export
      IMPORTING
        data = lt_string
      CHANGING
        rc   = lv_rc.
    IF lv_rc = 0.
      MESSAGE 'Activity Path copied to clipboard' TYPE 'S'.
    ENDIF.

     IF zz-html IS NOT INITIAL.
       zz-html = escape( val    = zz-html
                         format = cl_abap_format=>e_html_text ).

       cl_demo_output=>new(
         )->begin_section( 'Activity path'
         )->write_html( zz-html
         )->display( ).
    ENDIF.
    RETURN.
  ENDCASE.
ENDIF.

ENDENHANCEMENT.
```
