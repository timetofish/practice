# SAI API and attribute implementation

|Revision| Author| Date |
|----|----|----|
|Initial version|Shaoyu|26/02/2021|
 
As a chip vendor, we need to provide SAI debian package to community. This article aims to provide how to implement SAI API and SAI attribute from [OCP SAI standard](https://github.com/opencomputeproject/SAI).

For proprietary term, we also introduce how to Add a new API definition and new SAI attribute in a header file.

# Table of Contents
- [SAI API and attribute implementation](#sai-api-and-attribute-implementation)
- [Table of Contents](#table-of-contents)
- [SAI Implementation](#sai-implementation)
  - [SAI API](#sai-api)
  - [SAI Attribute](#sai-attribute)
- [SAI Extension](#sai-extension)
  - [New SAI API checking list](#new-sai-api-checking-list)
  - [New SAI Attribute checking list](#new-sai-attribute-checking-list)
***

# SAI Implementation 
## SAI API

Basically, SAI support four operations to manipulate SAI object. I will explain flow of these operation type. You can also reference [Mellonax SAI](https://github.com/Mellanox/SAI-Implementation) implementation for detail.

* Create
   
   The most important thing of this operation type is to create a SAI object ID and return to upper layer. 
   
   1. Create new object for initialization. 

   2. Traverse all attributes to get attribute id and value. Transfer SAI parameter to SDK parameter.

   3. Invoke sdk API and get SDK object id return from ASIC.

   4. Compose SDK object id and module type of SAI and store into software database.

   5. Return SAI Object and Log the procedure state.

* Get
   
   Get action is to get specific attributes of SAI object. This API operation type is not used in frequency. Usually, The using scenario is used to get switch capabilities, default state and counters.

   1. Get SAI object from software database by object ID.

   2. Traverse all attributes to get SAI attribute id and value. 

   3. Get attribute value from software database or sdk.

   4. Log the procedure state.

* Set
   
   In SONiC, we can set object attribute value individually. Then, SAI provides a action to set SAI attributes. 

   1. Get object from software database by object id.

   2. Traverse all attributes to get SAI attribute id and value. Transfer SAI parameter to SDK parameter. 

   3. Update the attribute value of this SAI object to software database.

   4. Log the procedure state.

* Remove


   1. Get SAI object from software database by SAI object ID and parse SDK object ID from SAI object.

   2. Remove SDK object via SDK API.
   3. Reset or free software database.
   4. Log the procedure state.


## SAI Attribute
 ```
    /**
     * @brief SAI ECMP default hash offset
     *
     * @type sai_uint32_t
     * @flags CREATE_AND_SET
     * @default 0
     */
    SAI_SWITCH_ATTR_ECMP_DEFAULT_HASH_OFFSET,
```
When a new SAI attribute is introduced, We need to handle it in switch case while parsing attribute list. From SAI header file, each attribute provides descriptions in detail. While SAI API is invoked, SAI meta.cpp helps to validate the format of sai object. 

* brief: Provide the brief descriptions of attribute.

* type: Shows the data type of attribute structure.

* flags: Shows the operation type. Could be 
  * MANDATORY_ON_CREATE, 
  * KEY, 
  * DYNAMIC, 
  * CREATE_AND_SET, 
  * CREATE_ONLY
  * READ_ONLY 

* defult: Default value of SAI.

***

# SAI Extension

For SAI extension(experimental) attributes or APIs, OCP provides a process to adding [SAI extensions](https://github.com/opencomputeproject/SAI/blob/master/doc/SAI-Extensions.md). We may follow the rule to extend SAI.  

If Adding a whole new SAI module, you may check the [PR1010](https://github.com/opencomputeproject/SAI/pull/1010)

## New SAI API checking list

Adding API definition into 
SAI/inc/saiXXX.h

and implement sairedis body in

```
sonic-sairedis/lib/src/sai_redis_XXX.cpp 

sonic-sairedis/lib/src/sai_vs_XXX.cpp 
```
***

## New SAI Attribute checking list

Adding API definition into 
SAI/inc/saiXXX.h

and 

SAI/inc/saitypes.h if define a new type.

Also add key words into SAI validation tool.

```
SAI/meta/acronyms.txt
SAI/meta/parse.pl
SAI/meta/aspell.en.pws
SAI/meta/saisanitycheck.c
SAI/meta/saimetadatatypes.h
```





