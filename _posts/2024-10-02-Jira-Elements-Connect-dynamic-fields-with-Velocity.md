---
title: "Dynamic Jira Fields with Elements Connect plugin: dynamic field via real-time data integration from other system"
date: 2023-05-01
tags: jira
---

When managing complex projects, you often need to integrate Jira with external systems to provide real-time data directly in your tickets. This becomes essential when you want to centralize information from multiple sources into a single place where all stakeholders can access up-to-date information.

## The Challenge

We needed Jira fields that could:
- Fetch real-time data from external systems (in our case IPAM/DCIM Nautobot system)
- Display dynamic dropdown options with current up-to-date values
- Query sets of objects, and filter out subset efficiently via API
- Show live data for network prefixes, VLANs, IP addresses, and team ownership information

## The Solution: Elements Connect plugin for Jira

After evaluating several plugins, we chose Elements Connect as it provided all the functionality we needed. While learning the Velocity template syntax initially posed a little challenge, it ultimately gave us the flexibility to implement exactly what our users required.

![Elements Connect Application Owner Field](/assets/images/2024-10-02-application-owner-edit-view.png)



## Key Benefits

**Real-time Integration**: Fields automatically pull current data from source systems, ensuring information is always up-to-date.

**User-friendly Interface**: Complex backend queries are presented as simple dropdown menus for end users.

**Cross-system Visibility**: Teams can access relevant data from multiple systems without leaving Jira.

## Implementation Notes

The plugin uses Velocity templates to define how data is fetched and displayed. While this requires some initial learning, it provides powerful customization capabilities for complex integration scenarios.

### Example Velocity Template

Here's the actual Velocity template code from our application owner field configuration:

**Query and custom field value storage:**
```velocity
#if($currentCustomfieldValue.length() > 0 && $userInput.length() == 0)

#set($ipam_query = "")
#set($string_of_values = "$currentCustomfieldValue.serialize(',,,').encodeURIComponent()") ## rather use triple commas (,,,) for better detection to be sure to detect
#foreach($arg in $string_of_values.split("%2C%2C%2C"))
    #set($arg = "$test")
    #set($ipam_query = "$ipam_query&value_ic=$arg")
#end

extras/custom-field-choices/?field_id=a723564f-a1c4-4e33-1203-4fe656618522$ipam_query

#elseif ($userInput.length() > 0)
extras/custom-field-choices/?field_id=a723564f-a1c4-4e33-1203-4fe656618522&value_ic=$userInput.encodeURIComponent()
#end
```

**Root element configuration:**
```
$.results
```

**Column mapping:**
- **Name:** `name`  
- **JSON path:** `$.value`

This template handles both scenarios: displaying previously selected values when editing a ticket, and providing filtered search results as users type in the field.

*This approach significantly improved our cross-team collaboration by bringing all stakeholders and relevant data into a centralized location within our existing Jira workflow.*




