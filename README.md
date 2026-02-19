# RSB_-Flexible-work-flow
Flexible work flow

Data Definitions

@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Plant Value Help For Flexible workflow'
@Metadata.ignorePropagatedAnnotations: true
@ObjectModel.usageType:{
    serviceQuality: #X,
    sizeCategory: #S,
    dataClass: #MIXED
}
@OData.publish: true
define view entity ZI_PlantVh_workflow
  as select from I_PlantVH
{
  key Plant,
      PlantName
}
***********************************************************************************************************************

Classes 

class ZCL_MM_PR_WF_CONDEVAL definition
  public
  final
  create public .

public section.

  interfaces IF_BADI_INTERFACE .
  interfaces IF_SWF_FLEX_IFS_CONDITION_DEF .
  interfaces IF_SWF_FLEX_IFS_CONDITION_EVAL .
ENDCLASS.



CLASS ZCL_MM_PR_WF_CONDEVAL IMPLEMENTATION.


  METHOD if_swf_flex_ifs_condition_def~get_conditions.

* condition id - value to be changed
    CONSTANTS: co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    ct_condition = VALUE #(
      ( id      = co_id
        subject = 'Is amount higher than'(001)
        type    = if_swf_flex_ifs_condition_def=>cs_condtype-start_step
      )
    ).

    ct_parameter = VALUE #(
      ( id        = co_id
        name      = 'Amount' ##NO_TEXT " Do not translate component 'name'!
        shorttext = 'Amount'(002)
        xsd_type  = if_swf_flex_ifs_condition_def=>cs_xstype-integer
        mandatory = abap_true
      )
    ).

  ENDMETHOD.


  METHOD if_swf_flex_ifs_condition_eval~evaluate_condition.

**** condition id - value to be changed
***    CONSTANTS co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.
***
***    cv_is_true = abap_false.
***
***    IF is_condition-condition_id <> co_id.
***      RAISE EXCEPTION TYPE cx_ble_runtime_error.
***    ENDIF.
***
**** If the condition evaluation is triggered for a draft business object (only in simulation), the cloud customer is not able to select the data for this draft business object.
**** In this case the evaluation should be stopped, so that it doesn't show one of the next workflow definitions inside workflow preview section, which is potentially not the
**** right one. In order to stop the evaluation, an exception must be raised.
***    IF iv_is_draft = abap_true.
***      RAISE EXCEPTION TYPE cx_ble_runtime_error.
***    ENDIF.
***
***
***    READ TABLE it_parameter_value INTO DATA(ls_param_value)
***      INDEX 1.
***    IF sy-subrc <> 0 OR ls_param_value-value IS INITIAL.
**** paramter is defined as mandatory
***      RAISE EXCEPTION TYPE cx_ble_runtime_error.
***    ENDIF.
***
***    TRY.
***        DATA(lv_amount) = CONV i( ls_param_value-value ).
***      CATCH cx_root INTO DATA(lx_exc) ##CATCH_ALL.
***        RAISE EXCEPTION TYPE cx_ble_runtime_error
***          EXPORTING
***            previous = lx_exc.
***    ENDTRY.
***
**** evaluate condition
***    IF lv_amount = 0.
**** check on business object instance not needed, condition is always TRUE
***      cv_is_true = abap_true.
***    ELSE.
*******   evaluation for your business object instance  ****
****   1: get business object instance by application key IS_SAP_OBJECT_NODE_TYPE
****     select single amount from <CDS_VIEWNAME> into @DATA(lv_inst_amount)
****        where ID = @is_sap_object_node_type-sont_key_part_1.
***
****   2: If object instance amount is greater than condition value return true else false
****    IF lv_inst_amount > lv_amount.
****     cv_is_true = abap_true.
****    ELSE.
****     cv_is_true = abap_false.
****    ENDIF.
***    ENDIF.

    CONSTANTS co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    cv_is_true = abap_false.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
*        EXPORTING
*          previous = NEW cx_swf_flex_lra( textid = cx_swf_flex_lra=>object_instance_not_found ).
    ENDIF.

    IF is_condition-condition_id <> co_id.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.

* If the condition evaluation is triggered for a draft business object (only in simulation), the cloud customer is not able to select the data for this draft business object.
* In this case the evaluation should be stopped, so that it doesn't show one of the next workflow definitions inside workflow preview section, which is potentially not the
* right one. In order to stop the evaluation, an exception must be raised.
    IF iv_is_draft = abap_true.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.


    READ TABLE it_parameter_value INTO DATA(ls_param_value)
      INDEX 1.
    IF sy-subrc <> 0 OR ls_param_value-value IS INITIAL.
* paramter is defined as mandatory
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.
    .

    TRY.
        DATA(lv_plant) = CONV string( ls_param_value-value  ).
      CATCH cx_root INTO DATA(lx_exc) ##CATCH_ALL.
        RAISE EXCEPTION TYPE cx_ble_runtime_error
          EXPORTING
            previous = lx_exc.
    ENDTRY.

    SELECT SINGLE purchaseorder, plant
     FROM i_purchaseorderitemapi01
     INTO  @DATA(ls_purchaseorder)
     WHERE purchaseorder = @is_sap_object_node_type-sont_key_part_1
     AND plant = @lv_plant.

    IF sy-subrc = 0.
*    evaluate condition
      cv_is_true = COND #( WHEN lv_plant IS INITIAL OR lv_plant <> ls_purchaseorder-plant
                           THEN abap_false
                           WHEN lv_plant IS NOT INITIAL AND lv_plant = ls_purchaseorder-plant
                           THEN abap_true ).
*      lv_plant = ls_purchaseorder-plant.
    ELSE.
      cv_is_true = abap_true.
*      lv_plant = lv_plant.
    ENDIF.





  ENDMETHOD.
ENDCLASS.
*********************************************************************************************************************************

class ZCL_MM_PR_WF_CONDITION definition
  public
  final
  create public .

public section.

  interfaces IF_BADI_INTERFACE .
  interfaces IF_SWF_FLEX_IFS_CONDITION_DEF .
protected section.
private section.
ENDCLASS.



CLASS ZCL_MM_PR_WF_CONDITION IMPLEMENTATION.


  METHOD if_swf_flex_ifs_condition_def~get_conditions.

    CONSTANTS: co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    ct_condition = VALUE #(
      ( id      = co_id
        subject = 'Plant of Purchase requisition is'(001)
        type    = if_swf_flex_ifs_condition_def=>cs_condtype-start_step
      )
    ).

    ct_parameter = VALUE #(
         ( id        = co_id
           name      = 'Plant' ##NO_TEXT " Do not translate component 'name'!
           shorttext = 'Plant'(002)
           xsd_type  = if_swf_flex_ifs_condition_def=>cs_xstype-string
           service_path = '/sap/opu/odata/sap/S_MMPURWORKFLOWVH_CDS'
           entity = 'S_MMPURWorkflowVH'
           property = 'Plant'
           mandatory = abap_true
         )
       ).



  ENDMETHOD.
ENDCLASS.
*********************************************************************************************************************************

class ZCL_MM_RFQ_WF_CONDITION definition
  public
  final
  create public .

public section.

  interfaces IF_BADI_INTERFACE .
  interfaces IF_SWF_FLEX_IFS_CONDITION_DEF .
protected section.
private section.
ENDCLASS.



CLASS ZCL_MM_RFQ_WF_CONDITION IMPLEMENTATION.


  METHOD if_swf_flex_ifs_condition_def~get_conditions.

* condition id - value to be changed
    CONSTANTS: co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    ct_condition = VALUE #(
      ( id      = co_id
        subject = 'RFQ Plant is'"(001)
        type    = if_swf_flex_ifs_condition_def=>cs_condtype-start_step
      )
    ).

    ct_parameter = VALUE #(
      ( id           = co_id
        name         = 'Plant' ##NO_TEXT " Do not translate component 'name'!
        shorttext    = 'Plant'"(002)
        xsd_type     = if_swf_flex_ifs_condition_def=>cs_xstype-string
        service_path = '/sap/opu/odata/sap/S_MMPURWORKFLOWVH_CDS'
        entity       = 'S_MMPURWorkflowVH'
        property     = 'Plant'
        mandatory    = abap_true
      )
    ).
  ENDMETHOD.
ENDCLASS.
*******************************************************************************************************************************************************

class ZCL_MM_RFQ_WF_CONDITION_EVAL definition
  public
  final
  create public .

public section.

  interfaces IF_BADI_INTERFACE .
  interfaces IF_SWF_FLEX_IFS_CONDITION_DEF .
  interfaces IF_SWF_FLEX_IFS_CONDITION_EVAL .
ENDCLASS.



CLASS ZCL_MM_RFQ_WF_CONDITION_EVAL IMPLEMENTATION.


  METHOD if_swf_flex_ifs_condition_def~get_conditions.

* condition id - value to be changed
    CONSTANTS: co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    ct_condition = VALUE #(
      ( id      = co_id
        subject = 'RFQ Plant is'"(001)
        type    = if_swf_flex_ifs_condition_def=>cs_condtype-start_step
      )
    ).

    ct_parameter = VALUE #(
      ( id        = co_id
        name      = 'Plant' ##NO_TEXT " Do not translate component 'name'!
        shorttext = 'Plant'"(002)
        xsd_type  = if_swf_flex_ifs_condition_def=>cs_xstype-string
        service_path = '/sap/opu/odata/sap/S_MMPURWORKFLOWVH_CDS'
        entity = 'S_MMPURWorkflowVH'
        property = 'Plant'
        mandatory = abap_true
      )
    ).

  ENDMETHOD.


  METHOD if_swf_flex_ifs_condition_eval~evaluate_condition.

**** condition id - value to be changed
***    CONSTANTS co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.
***
***    cv_is_true = abap_false.
***
***    IF is_condition-condition_id <> co_id.
***      RAISE EXCEPTION TYPE cx_ble_runtime_error.
***    ENDIF.
***
**** If the condition evaluation is triggered for a draft business object (only in simulation), the cloud customer is not able to select the data for this draft business object.
**** In this case the evaluation should be stopped, so that it doesn't show one of the next workflow definitions inside workflow preview section, which is potentially not the
**** right one. In order to stop the evaluation, an exception must be raised.
***    IF iv_is_draft = abap_true.
***      RAISE EXCEPTION TYPE cx_ble_runtime_error.
***    ENDIF.
***
***
***    READ TABLE it_parameter_value INTO DATA(ls_param_value)
***      INDEX 1.
***    IF sy-subrc <> 0 OR ls_param_value-value IS INITIAL.
**** paramter is defined as mandatory
***      RAISE EXCEPTION TYPE cx_ble_runtime_error.
***    ENDIF.
***
***    TRY.
***        DATA(lv_amount) = CONV i( ls_param_value-value ).
***      CATCH cx_root INTO DATA(lx_exc) ##CATCH_ALL.
***        RAISE EXCEPTION TYPE cx_ble_runtime_error
***          EXPORTING
***            previous = lx_exc.
***    ENDTRY.
***
**** evaluate condition
***    IF lv_amount = 0.
**** check on business object instance not needed, condition is always TRUE
***      cv_is_true = abap_true.
***    ELSE.
*******   evaluation for your business object instance  ****
****   1: get business object instance by application key IS_SAP_OBJECT_NODE_TYPE
****     select single amount from <CDS_VIEWNAME> into @DATA(lv_inst_amount)
****        where ID = @is_sap_object_node_type-sont_key_part_1.
***
****   2: If object instance amount is greater than condition value return true else false
****    IF lv_inst_amount > lv_amount.
****     cv_is_true = abap_true.
****    ELSE.
****     cv_is_true = abap_false.
****    ENDIF.
***    ENDIF.



    CONSTANTS co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.
*    BREAK ntt_abap7.
    cv_is_true = abap_false.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
*        EXPORTING
*          previous = NEW cx_swf_flex_lra( textid = cx_swf_flex_lra=>object_instance_not_found ).
    ENDIF.

    IF is_condition-condition_id <> co_id.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.

* If the condition evaluation is triggered for a draft business object (only in simulation), the cloud customer is not able to select the data for this draft business object.
* In this case the evaluation should be stopped, so that it doesn't show one of the next workflow definitions inside workflow preview section, which is potentially not the
* right one. In order to stop the evaluation, an exception must be raised.
    IF iv_is_draft = abap_true.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.


    READ TABLE it_parameter_value INTO DATA(ls_param_value)
      INDEX 1.
    IF sy-subrc <> 0 OR ls_param_value-value IS INITIAL.
* paramter is defined as mandatory
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.
    .

    TRY.
        DATA(lv_plant) = CONV string( ls_param_value-value ).
      CATCH cx_root INTO DATA(lx_exc) ##CATCH_ALL.
        RAISE EXCEPTION TYPE cx_ble_runtime_error
          EXPORTING
            previous = lx_exc.
    ENDTRY.

*BREAK ntt_abap7.
    SELECT SINGLE purchasingdocument, plant
*     FROM i_purchaseorderitemapi01
     FROM i_purchasingdocumentitemstdvh
     INTO  @DATA(ls_purchaseorder)
     WHERE purchasingdocument = @is_sap_object_node_type-sont_key_part_1
     AND plant = @lv_plant.

    IF sy-subrc = 0.
*    evaluate condition
      cv_is_true = COND #( WHEN lv_plant IS INITIAL OR lv_plant <> ls_purchaseorder-plant
                           THEN abap_false
                           WHEN lv_plant IS NOT INITIAL AND lv_plant = ls_purchaseorder-plant
                           THEN abap_true ).

    ELSE.
      cv_is_true = abap_false.

    ENDIF.
  ENDMETHOD.
ENDCLASS.
***********************************************************************************************************************************

class ZZ1_MM_POFLEX_CONDITION definition
  public
  final
  create public .

public section.

  interfaces IF_BLE_BADI_MARKER_INTERFACE .
  interfaces IF_BADI_INTERFACE .
  interfaces IF_SWF_FLEX_IFS_CONDITION_DEF .
protected section.
private section.
ENDCLASS.



CLASS ZZ1_MM_POFLEX_CONDITION IMPLEMENTATION.


  method IF_SWF_FLEX_IFS_CONDITION_DEF~GET_CONDITIONS.
    "PARAMETER TRACING 2202
    DATA(_sap_internal_parameter_values) = VALUE if_ble_badi_runtime=>tt_parameter_value(
      ( parameter_name = 'CT_CONDITION' parameter_value = REF #( CT_CONDITION ) )
      ( parameter_name = 'CT_PARAMETER' parameter_value = REF #( CT_PARAMETER ) )
    ).
    cl_ble_badi_runtime=>if_ble_badi_runtime~open( iv_implementation = 'ZZ1_MM_POFLEX_CONDITION' it_parameter_values = _sap_internal_parameter_values ).
    TRY.
        " ------------------------------
        " payload goes after this comment
        " ------------------------------

* condition id - value to be changed
    CONSTANTS: co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    ct_condition = VALUE #(
      ( id      = co_id
        subject = 'Plant of Purchase order is'(001)
        type    = if_swf_flex_ifs_condition_def=>cs_condtype-start_step
      )
    ).

 ct_parameter = VALUE #(
      ( id        = co_id
        name      = 'Plant' ##NO_TEXT " Do not translate component 'name'!
        shorttext = 'Plant'(002)
        xsd_type  = if_swf_flex_ifs_condition_def=>cs_xstype-string
        SERVICE_PATH = '/sap/opu/odata/sap/S_MMPURWORKFLOWVH_CDS'
        ENTITY = 'S_MMPURWorkflowVH'
        PROPERTY = 'Plant'
        mandatory = abap_true
      )
    ).
        " ------------------------------
        " end of payload
        " ------------------------------
      CATCH BEFORE UNWIND cx_no_check cx_static_check cx_dynamic_check INTO DATA(lx_no_handler).
        cl_ble_badi_runtime=>if_ble_badi_runtime~handle_exception( exception = lx_no_handler caller = me ).
    ENDTRY.
    cl_ble_badi_runtime=>if_ble_badi_runtime~close( _sap_internal_parameter_values ).

  endmethod.
ENDCLASS.
*******************************************************************************************************************************************

class ZZ1_MM_POFLEX_CONDITION_EVAL definition
  public
  final
  create public .

public section.

  interfaces IF_BLE_BADI_MARKER_INTERFACE .
  interfaces IF_BADI_INTERFACE .
  interfaces IF_SWF_FLEX_IFS_CONDITION_EVAL .
protected section.
private section.
ENDCLASS.



CLASS ZZ1_MM_POFLEX_CONDITION_EVAL IMPLEMENTATION.


  method IF_SWF_FLEX_IFS_CONDITION_EVAL~EVALUATE_CONDITION.
    "PARAMETER TRACING 2202
    DATA(_sap_internal_parameter_values) = VALUE if_ble_badi_runtime=>tt_parameter_value(
      ( parameter_name = 'IS_SAP_OBJECT_NODE_TYPE' parameter_value = REF #( IS_SAP_OBJECT_NODE_TYPE ) )
      ( parameter_name = 'IS_CONDITION' parameter_value = REF #( IS_CONDITION ) )
      ( parameter_name = 'IT_PARAMETER_VALUE' parameter_value = REF #( IT_PARAMETER_VALUE ) )
      ( parameter_name = 'IV_IS_DRAFT' parameter_value = REF #( IV_IS_DRAFT ) )
      ( parameter_name = 'CV_IS_TRUE' parameter_value = REF #( CV_IS_TRUE ) )
    ).
    cl_ble_badi_runtime=>if_ble_badi_runtime~open( iv_implementation = 'ZZ1_MM_POFLEX_CONDITION_EVAL' it_parameter_values = _sap_internal_parameter_values ).
    TRY.
        " ------------------------------
        " payload goes after this comment
        " ------------------------------

* condition id - value to be changed
    CONSTANTS co_id TYPE if_swf_flex_ifs_condition_def=>ty_condition_id VALUE 'CL_SWF_FLEX_IFS_BADI_COND_SAMP' ##NO_TEXT.

    cv_is_true = abap_false.

    IF sy-subrc <> 0.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
*        EXPORTING
*          previous = NEW cx_swf_flex_lra( textid = cx_swf_flex_lra=>object_instance_not_found ).
    ENDIF.

    IF is_condition-condition_id <> co_id.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.

* If the condition evaluation is triggered for a draft business object (only in simulation), the cloud customer is not able to select the data for this draft business object.
* In this case the evaluation should be stopped, so that it doesn't show one of the next workflow definitions inside workflow preview section, which is potentially not the
* right one. In order to stop the evaluation, an exception must be raised.
    IF iv_is_draft = abap_true.
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.


    READ TABLE it_parameter_value INTO DATA(ls_param_value)
      INDEX 1.
    IF sy-subrc <> 0 OR ls_param_value-value IS INITIAL.
* paramter is defined as mandatory
      RAISE EXCEPTION TYPE cx_ble_runtime_error.
    ENDIF.
    .

    TRY.
        DATA(lv_plant) = CONV string( ls_param_value-value  ).
      CATCH cx_root INTO DATA(lx_exc) ##CATCH_ALL.
        RAISE EXCEPTION TYPE cx_ble_runtime_error
          EXPORTING
            previous = lx_exc.
    ENDTRY.

    SELECT SINGLE purchaseorder, plant
     FROM i_purchaseorderitemapi01
     INTO  @DATA(ls_purchaseorder)
      WHERE purchaseorder = @is_sap_object_node_type-sont_key_part_1.
*      AND plant = @lv_plant.

    IF sy-subrc = 0.
*    evaluate condition
      cv_is_true = COND #( WHEN lv_plant IS INITIAL OR lv_plant <> ls_purchaseorder-plant
                           THEN abap_false
                           WHEN lv_plant IS NOT INITIAL AND lv_plant = ls_purchaseorder-plant
                           THEN abap_true ).
*      lv_plant = ls_purchaseorder-plant.
    ELSE.
      cv_is_true = abap_true.
*      lv_plant = lv_plant.
    ENDIF.


*



* check on business object instance not needed, condition is always TRUE
*    cv_is_true = COND #( WHEN lv_plant = 0 OR lv_plant <> '1105'
*                         THEN abap_false
*                         WHEN lv_plant = '1105'
*                         THEN abap_true ).
        " ------------------------------
        " end of payload
        " ------------------------------
      CATCH BEFORE UNWIND cx_no_check cx_static_check cx_dynamic_check INTO DATA(lx_no_handler).
        cl_ble_badi_runtime=>if_ble_badi_runtime~handle_exception( exception = lx_no_handler caller = me ).
    ENDTRY.
    cl_ble_badi_runtime=>if_ble_badi_runtime~close( _sap_internal_parameter_values ).

  endmethod.

ENDCLASS.
**************************************************************************************************************************************

