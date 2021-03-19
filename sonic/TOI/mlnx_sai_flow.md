# SAI Implementation example(Mlnx)

|Revision| Author| Date |
|----|----|----|
|Initial version|Shaoyu|19/03/2021|

[Mellonax SAI](https://github.com/Mellanox/SAI-Implementation) shows how to implement SAI API and attribute. We choose one module to demonstrate
the mapping of the implementation steps which were mentioned in [SAI implementation](https://github.com/timetofish/practice/blob/main/sonic/TOI/README.SAI_implementation.md)

we choose hostif_trap object to trace the flow. the api list could be found in 

```
const sai_hostif_api_t mlnx_host_interface_api = {
...
    mlnx_create_hostif_trap,
    mlnx_remove_hostif_trap,
    mlnx_set_hostif_trap_attribute,
    mlnx_get_hostif_trap_attribute,
```
at the end of mlnx_sai_host_interface.c

* Create
   
   The most important thing of this operation type is to create a SAI object ID and return to upper layer. 
   
   1. Parameter check.
 ```
if (SAI_STATUS_SUCCESS !=
        (status =
             check_attribs_metadata(attr_count, attr_list, SAI_OBJECT_TYPE_HOSTIF_TRAP, trap_vendor_attribs,
                                    SAI_COMMON_API_CREATE))) {
        SX_LOG_ERR("Failed attribs check\n");
        return status;
    }
 ```  
   2. Create new object for initialization. (In Mlnx, all object database are allocate at beginning stage.)
   
```
static void sai_db_values_init()
```

   3. Traverse all attributes to get attribute id and value. Transfer SAI parameter to SDK parameter.
```
    status = find_attrib_in_list(attr_count, attr_list, SAI_HOSTIF_TRAP_ATTR_TRAP_TYPE, &trap_id, &trap_id_index);
    assert(SAI_STATUS_SUCCESS == status);
```

   4. Invoke sdk API and get SDK object id return from ASIC.

```
        sx_status = sx_api_host_ifc_trap_id_ext_set(gh_sdk, cmd, &trap_key, &trap_attr);
        if (SX_ERR(sx_status)) {
            SX_LOG_ERR("Failed to %s for index %u trap %u/%u=%u, error is %s\n", SX_ACCESS_CMD_STR(cmd),
                       index, trap_index + 1, mlnx_traps_info[index].sdk_traps_num,
                       mlnx_traps_info[index].sdk_trap_ids[trap_index], SX_STATUS_MSG(sx_status));
            return sdk_to_sai(sx_status);
        }
```



   5. Compose SDK object id/idx of database and module type of SAI and store into software database.

```
    g_sai_db_ptr->traps_db[index].action     = action->s32;
    g_sai_db_ptr->traps_db[index].trap_group = (group) ? group->oid : g_sai_db_ptr->default_trap_group;

if (SAI_STATUS_SUCCESS !=
        (status = mlnx_create_object(SAI_OBJECT_TYPE_HOSTIF_TRAP, trap_id->s32, NULL, hostif_trap_id))) {
        SX_LOG_EXIT();
        return status;
    }
```

   6. Return SAI Object and Log the procedure state.

```
    trap_key_to_str(*hostif_trap_id, key_str);

    SX_LOG_NTC("Created trap %s\n", key_str);
```

* Get
   
   Get action is to get specific attributes of SAI object. This API operation type is not used in frequency. Usually, The using scenario is used to get switch capabilities, default state and counters.

   1. Parameter check.
```
if (SAI_STATUS_SUCCESS !=
   (status =
         check_attribs_metadata(attr_count, attr_list, object_type, functionality_vendor_attr,
                              SAI_COMMON_API_GET))) {
   SX_LOG((((SAI_STATUS_IS_ATTR_NOT_IMPLEMENTED(status)) || (SAI_STATUS_IS_ATTR_NOT_SUPPORTED(status))) ?
            SX_LOG_WARNING : SX_LOG_ERROR),
         "Failed attribs check, key:%s\n", key_str);
   SX_LOG_EXIT();
   return status;
}
```

   2. Get SAI object from software database by object ID.
```
    status = sai_vendor_attr_index_find(attr->id, functionality_vendor_attr, &index);
    if (SAI_ERR(status)) {
        SX_LOG_EXIT();
        return status;
    }
```
   3. Traverse all attributes to get SAI attribute id and value.


```
for (ii = 0; ii < attr_count; ii++) {
        attr_id = attr_list[ii].id;
```
   There is a get_dispatch_attribs_handler to mapping to each get function. 

   4. Get attribute value from software database or sdk.

```
status = functionality_vendor_attr[index].getter(key, &(attr_list[ii].value), ii, &cache,
                                                         vendor_getter_arg);
```
After calling the callback function, attribute will be obtained.
```
static sai_status_t mlnx_trap_type_get(_In_ const sai_object_key_t   *key,
_Inout_ sai_attribute_value_t *value,
_In_ uint32_t                  attr_index,
_Inout_ vendor_cache_t        *cache,
void                          *arg)
{
   ...
}

```
   5. Log the procedure state.

```
    SX_LOG_EXIT();
```

* Set
   
   In SONiC, we can set object attribute value individually. Then, SAI provides a action to set SAI attributes. 

   1. Parameter check.
```
if (SAI_STATUS_SUCCESS !=
   (status =
         check_attribs_metadata(1, attr, object_type, functionality_vendor_attr, SAI_COMMON_API_SET))) {
   SX_LOG((((SAI_STATUS_ATTR_NOT_IMPLEMENTED_0 == status) || (SAI_STATUS_ATTR_NOT_SUPPORTED_0 == status)) ?
            SX_LOG_WARNING : SX_LOG_ERROR),
         "Failed attribs check, key:%s\n", key_str);
   SX_LOG_EXIT();
   return status;
}
```
   2. Get object from software database by object id.
```
    status = sai_vendor_attr_index_find(attr->id, functionality_vendor_attr, &index);
    if (SAI_ERR(status)) {
        SX_LOG_EXIT();
        return status;
    }
```


   3. Get SAI attribute id and value. Transfer SAI parameter to SDK parameter. 

    The set function is not implemented by mlnx, In general case, set_dispatch_attrib_handler will be invoked to mapping the implementation function.
```
N/A
```

   1. Update the attribute value of this SAI object to software database.
```
N/A
```
   5. Log the procedure state.
```
N/A
```

* Remove


   1. Parameter check.
```
if (SAI_STATUS_SUCCESS !=
   (status = mlnx_object_to_type(hostif_trap_id, SAI_OBJECT_TYPE_HOSTIF_TRAP, &trap_id, NULL))) {
   SX_LOG_EXIT();
   return status;
}
```
   2. Get SAI object from software database by SAI object ID and parse SDK object ID from SAI object.
```
if (SAI_STATUS_SUCCESS != (status = find_sai_trap_index(trap_id, MLNX_TRAP_TYPE_REGULAR, &index))) {
   SX_LOG_ERR("Invalid trap %x\n", trap_id);
   return SAI_STATUS_INVALID_PARAMETER;
}
```
   3. Remove SDK object via SDK API.
```
status = mlnx_trap_unset(index);
if (SAI_ERR(status)) {
   goto out;
}
```
   4. Reset or free software database.
```
    g_sai_db_ptr->traps_db[index].action     = mlnx_traps_info[index].action;
    g_sai_db_ptr->traps_db[index].trap_group = g_sai_db_ptr->default_trap_group;
```
   5. Log the procedure state.
```
   SX_LOG_EXIT();
```