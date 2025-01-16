# Overview
This doc explains how view files (files that use the `.view.lkml` extension) and the contents within them should be formatted.

When the doc references "fields", it is meant to reference any view parameter of type `dimension`, `measure`, `dimension_group`, `filter`, or `parameter`.

# View Organization

* All non-field parameters that are not a  should go at the top of the file.
* Below that, all of the field parameters should be organized into sections in the following order. All sections are optional. If there are no fields that match their criteria, they should be removed
  * FILTERS & PARAMETERS — Any `filter` or `parameter` fields
  * DIMENSIONS - Any `dimension` or `dimension_group` fields that either have a `hidden: no` parameter or do not have a `hidden:` parameter
  * MEASURES  - Any `measure` fields that either have a `hidden: no` parameter or do not have a `hidden:` parameter
  * HIDDEN DIMENSIONS - Any `dimension` or `dimension_group` fields that have a `hidden: yes` parameter
  * HIDDEN MEASURES - Any `measure` fields that have a `hidden: yes` parameter
* Each section should be blocked off with some commented section headings so that they are separated visually, as such:
```
view: view_name {
  sql_table_name: sql_table_reference ;;
        
#
# FILTERS & PARAMETERS
#

#
# DIMENSIONS
#

#
# MEASURES
#

#
# HIDDEN DIMENSIONS
#

#
# HIDDEN MEASURES
#

}
```
* Within each section, fields should be sorted alphabetically (Ascending)

# Field Formatting & Guidelines
The below sections define which common parameters are required/optional and the sequence in which they should be listed within each field. If the field has parameters not listed in the examples, they should be added to the end of that field's list of parameters.

## General Guidelines
* Any field that isn't hidden should have a `label` parameter and a `description` parameter.
* Any `hidden: no` parameters should be removed, with 1 exception: If the view has a `fields_hidden_by_default: yes` parameter.
* Fields should be indented once, and their parameters should be indented twice

## Dimension Groups
```
  dimension_group: field_name {
    group_label:  # Optional
    label:        # Required only if field is not hidden.
    description:  # Required only if field is not hidden.
    hidden:       # Required only if value is yes. Default is no.
    type:         # Required. Typically, value will be time.
    convert_tz:   # Required. Typically, this should be no.
    timeframe:    # Required. If this wasn't included, it should be addd with the following default value: [raw, date, day_of_week, day_of_week_index, week, month, month_name, month_num, quarter, year]
    sql:          # Required.
  }
```

## Dimensions
```
  dimension: field_name {
    primary_key:       # Required only if the field is the primary key.
    group_label:       # Optional.
    label:             # Required only if field is not hidden.
    description:       # Required only if field is not hidden.
    type:              # Required.
    hidden:            # Required only if value is yes. Default is no.
    sql:               # Required.
    value_format:      # Optional.
    value_format_name: # Optional.
    html:              # Optional.
```

## Measures
```
  measure: field_name {
    group_label:        # Optional.
    label:              # Required only if field is not hidden.
    description:        # Required only if field is not hidden.
    type:               # Required.
    hidden:             # Required only if value is yes. Default is no.
    sql:                # Required.
    value_format:       # Optional.
    value_format_name:  # Optional.
    html:               # Optional.
```
### Measures should reference dimensions, not columns directly
When creating a measure you should reference a dimension rather than the table column directly.

For example, if we have a column in the table called units that we want to aggregate in our view, it should look like this:
```
# DO THIS
  dimension: order_revenue { # Hidden dimension that references the column that will be aggregated in a separate measure
    type: number
    hidden: yes
    sql: ${TABLE}."ORDER_REVENUE";;
  }

  measure: total_order_revenue { # The measure that aggregates the hidden dimension
    label: "Total Order Revenue"
    description: "Sum of order revenue."
    type: sum
    sql: ${order_revenue};;
  }
```
Here is what you should _not_ do:
```
# DON'T DO THIS
  measure: total_order_revenue {
    label: "Total Order Revenue"
    description: "Sum of order revenue."
    type: sum
    sql: ${TABLE}."ORDER_REVENUE";; # This references the table column directly from within a measure.
  }
```
Why do we do this? Because sometimes you may want to look at the value of a column as a dimension, and sometimes you might want to look at it as a measure. By doing it this way, you'll always preserve that flexibility.
