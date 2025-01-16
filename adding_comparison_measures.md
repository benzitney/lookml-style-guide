# Overview
When you are asked to "Add the comparison measures to this view file", you are being asked to do all of the steps in this document.

We are updating our LookML code to make it easier for users to do time-based comparisons for different measures. This involves updating all of our existing measures and creating derivate measures of those existing measures.

These are the high-level steps.
1. Find all `type: sum` measures that aren't hidden, then (1) add a filter to them and (2) update/add their `group_label` parameters. Create derivative measures off of those existing non-hidden `sum` measures that will give the Comparison period value, Change between the Current period and comparison period, and % Change between the current period and the comparison period
2. Do the same for non-hidden `type: number` measures. We do this as a distinct step because it will require usage of measures created in step 2
3. Apply LookML style guidelines to the entire view file. These should already be stored in your knowledge base. If you do not already have them , they are here: https://github.com/benzitney/lookml-style-guide/blob/main/view_files.md

The following will go into detail of how to complete these steps. 
Each step should be mutually exclusive. You should complete step 1 in full. Then complete step 2 in full. Then complete step 3 in full. Then complete step 4 in full.

## Note on LookML references
In these instructions, when we say "measure name", we're referring to the identifier of a measure in a lookml view. In the following example, the measure name is `total_amazon_valid_orders`:
```
  measure: total_amazon_valid_orders {
    type: sum
    group_label: "Amazon"
    label: "Amazon Valid Orders"
    description: "Total valid orders from Amazon"
    sql: ${amazon_valid_orders} ;;
    value_format_name: decimal_0
  }
```

# Step 1 — Sum measures
## 1.1 Find qualfying measures
Find all of the measures that:
1. Do not have a `hidden: yes` parameter
2. Have `type: sum`

## 1.2 Update/add filters
For each of the qualifying measures, you should:
* If the measure does not have a `filters` parameter, add one at the bottom of the list of parameters. It should look like this: `filters: [calendar_v2.period: "current"]`
* If the measure already has a `filters` parameter, you need to add the following to the contents: `, calendar_v2.period: "current"`.
  * For example, if the measure already has a `filters` parameter like `filters: [date.something: "yesterday"]`, the contents should be updated to be: `filters: [date.something: "yesterday", calendar_v2.period: "current"]`.
  * Another example: If the measure already has a `filters` parameter like `filters: [date.something: "yesterday", date.everything: "tomorrow"]`, it should be updated to be: `filters: [date.something: "yesterday", date.everything: "tomorrow", calendar_v2.period: "current"]`

## 1.3 Update/add group_label
For each of the qualifying measures, you should also:
* If the measure does not have a `label` parameter, create one. The value should be the titlecase version of the measure name. Underscores should be replaced by spaces.
* If the measure has a `group_label` parameter, replace the value with the same value as the `label` parameter
* If the measure does not have a `group_label` parameter, add one and give it the same value as the `label` parameter. 

## 1.4 Create derivative measures
For each qualfiying measure (we'll call these "seed" measures), you will create 3 additional measure that will be derivatives of the seed measure. The 3 derivatives will be: `Comp Period`, `Chg vs Comp`, and `% Chg vs Comp`,

All of the derivatives should inherit any existing parameters in the seed measure, unless instructions are given below to update/add/remove those parameters.

### Comp Period
* measure name should be the same as the seed measure, but should have `_comp` appended
* measure `type` should be `sum`
* give it the same `group_label` as the seed measure
* measure `label` should be the same as the current measure, but with the text ` (Comp Period)` appended
* measure `description` should be `"seed_label during the comparison period."` (with `seed_label` replaced by the label value from the seed measure
* measure `filters` parameter should be the same as the seed measure, except the filter criteria for `calendar_v2.period` should be `"comp"` instead of `"current"`
* measure `value_format_name` should be whatever the `value_format_name` of the seed measure was

### Chg vs Comp
* measure name should be the same as the seed measure, but should have `_chg_vs_comp` appended
* measure `type` should be `number`
* give it the same `group_label` as the seed measure
* measure `label` should be the same as the current measure, but with the text ` (Chg vs Comp)` appended
* measure `description` should be `"The difference between the current period and the comparison period."`
* measure `sql` parameter should be `${seed_measure} - ${comp_period} ;;` but replace `seed_measure` with the name of the seed measure and `comp_period` with the name of the seed measure's comp_period measure
* measure `value_format_name` should be whatever the `value_format_name` of the seed measure was

### % Chg vs Comp
* measure name should be the same as the seed measure, but should have `_pct_chg_vs_comp` appended
* measure `type` should be `number`
* give it the same `group_label` as the seed measure
* measure `label` should be the same as the current measure, but with the text ` (% Chg vs Comp)` appended
* measure `description` should be `"The percent change between the current period and the comparison period."`
* measure `sql` parameter should be `1.00 * ${seed_measure} / NULLIF(${comp_period},0) - 1 ;;` but replace `seed_measure` with the name of the seed measure and `comp_period` with the name of the seed measure's comp_period measure
* measure value_format_name should be `percent_change_0`

Here is an example ...
Seed measure:
```
  measure: total_amazon_valid_orders {
    type: sum
    group_label: "Amazon"
    label: "Amazon Valid Orders"
    description: "Total valid orders from Amazon"
    sql: ${amazon_valid_orders} ;;
    value_format_name: decimal_0
  }
```

This should be replaced by the following:
```
  measure: total_amazon_valid_orders {
    type: sum
    group_label: "Amazon Valid Orders"
    label: "Amazon Valid Orders"
    description: "Total valid orders from Amazon"
    sql: ${amazon_valid_orders} ;;
    value_format_name: decimal_0
    filters: [calendar_v2.period: "current"]
  }
  measure: total_amazon_valid_orders_chg_vs_comp {
    type: number
    group_label: "Amazon Valid Orders"
    label: "Amazon Valid Orders (Chg vs Comp)"
    description: "The difference between the current period and the comparison period."
    sql: ${total_amazon_valid_orders} - ${total_amazon_valid_orders_comp};;
    value_format_name: decimal_0
  }
  measure: total_amazon_valid_orders_comp {
    type: sum
    group_label: "Amazon Valid Orders"
    label: "Amazon Valid Orders (Comp Period)"
    description: "Total valid orders from Amazon during the comparison period."
    sql: ${total_amazon_valid_orders} ;;
    value_format_name: decimal_0
    filters: [calendar_v2.period: "comp"]
  }
  measure: total_amazon_valid_orders_pct_chg_vs_comp {
    type: number
    group_label: "Amazon Valid Orders"
    label: "Amazon Valid Orders (% Chg vs Comp)"
    description: "The percent change between the current period and the comparison period."
    sql: 1.00 * ${total_amazon_valid_orders} / NULLIF(${total_amazon_valid_orders_comp},0) - 1 ;;
    value_format_name: percent_change_0
  }

```
# Step 2 — Number Measures
## 2.1 Find qualifying measures
Find all of the measures that:
1. Are `type: number`
2. Are not any of the `_chg_vs_comp` or `_pct_chg_vs_comp` measures you created in the last step
3. Do not have a `hidden: yes` parameter

## 2.2 Update/Add group label
For each of the qualifying measures, you should:
* If the measure does not have a `label` parameter, create one. The value should be the titlecase version of the measure name. Underscores should be replaced by spaces.
* If the measure has a `group_label` parameter, replace the value with the same value as the `label` parameter
* If the measure does not have a `group_label` parameter, add one and give it the same value as the `label` parameter. 

## 2.3 Create derivative measures
As in step 1.4, you'll create 3 derivative measures off of each of the qualifying "seed" measures, but some details will be slightly different

### Comp Period
* measure name should be the same as the seed measure, but should have `_comp` appended
* measure `type` should be `number`
* give it the same `group_label` as the seed measure
* measure `label` should be the same as the current measure, but with the text ` (Comp Period)` appended
* measure `description` should be `"seed_label during the comparison period."` (with `seed_label` replaced by the label value from the seed measure
* measure `value_format_name` should be whatever the `value_format_name` of the seed measure was
* measure `sql` parameter should be the same as the seed measure EXCEPT the fields that are being referenced should be to the `_comp` variations.
  * For example, if the `sql` parameter for the seed measure is `sql: ${total_marketing_spend} / NULLIF(${total_new_customers},0) ;;`, the `sql` parameter for the `_comp` measure should be: `sql: ${total_marketing_spend_comp} / NULLIF(${total_new_customers_comp},0) ;;`

### Chg vs Comp
* measure name should be the same as the seed measure, but should have `_chg_vs_comp` appended
* measure `type` should be `number`
* give it the same `group_label` as the seed measure
* measure `label` should be the same as the current measure, but with the text ` (Chg vs Comp)` appended
* measure `description` should be `"The difference between the current period and the comparison period."`
* measure `sql` parameter should be `${seed_measure} - ${comp_period} ;;` but replace `seed_measure` with the name of the seed measure and `comp_period` with the name of the seed measure's comp_period measure
* measure `value_format_name` should be whatever the `value_format_name` of the seed measure was

### % Chg vs Comp
* measure name should be the same as the seed measure, but should have `_pct_chg_vs_comp` appended
* measure `type` should be `number`
* give it the same `group_label` as the seed measure
* measure `label` should be the same as the current measure, but with the text ` (% Chg vs Comp)` appended
* measure `description` should be `"The percent change between the current period and the comparison period."`
* measure `sql` parameter should be `1.00 * ${seed_measure} / NULLIF(${comp_period},0) - 1 ;;` but replace `seed_measure` with the name of the seed measure and `comp_period` with the name of the seed measure's comp_period measure
* measure value_format_name should be `percent_change_0`

## Step 3 — Apply LookML style guidelines
Once the content in the view file is updated, it should be formatted to accord with the lookml style guidelines in this file: https://github.com/benzitney/lookml-style-guide/blob/main/view_files.md
