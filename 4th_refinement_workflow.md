# Codex Implementation Specification: Workflow Refinement

## Scope

Make the following small visual and labeling refinements to:

`SRS473742_mapping_stats.png`

The changes apply specifically to the **horizontal coverage plot** in the **second row** of the figure.

## 1. Adjust horizontal coverage plot layout

In the right-hand part of the second row, there is currently some unused white space.

* Stretch the horizontal coverage plot slightly to make better use of the available space.
* Increase the horizontal distance between the plot area and its legend.
* Keep the overall two-row layout aligned:

  * The **first row and second row must start at the same horizontal position**.
  * The **first row and second row must end at the same horizontal position**.
  * Do not make either row horizontally longer or shorter than the other.
* Do not otherwise change the overall figure layout unnecessarily.

## 2. Add reference ID information to the x-axis

For the **horizontal coverage plot**, update the x-axis labeling to include the reference ID.

The x-axis description should communicate:

**`reference XYZ position (bp)`**

Where `XYZ` is replaced with the appropriate reference ID for the plotted data.

The reference ID should be taken from the existing data/workflow rather than hard-coded to a specific value.

## 3. Update horizontal coverage plot title

Current title:

`sample ID: coverage and Read ANI by 100bp bin`

Change it to:

`sample ID: Coverage and read ANI by 100bp bin`

Only the capitalization/wording specified above should change.

## 4. Update y-axis label

For the horizontal coverage plot, change the y-axis description to exactly:

`Read depth`

## 5. Update legend label

For the horizontal coverage plot, change the legend description to exactly:

`Read ANI (%)`

Remove the word **`Mean`** from the legend label.

## 6. Remove mean coverage annotation

Remove the rectangle/annotation from the horizontal coverage plot that currently contains:

`Mean coverage per bin`

The rectangle and its associated text should no longer appear.

## Acceptance Criteria

The implementation is complete when:

* [ ] The second-row horizontal coverage plot uses the available white space more effectively.
* [ ] There is visibly more separation between the horizontal coverage plot and its legend.
* [ ] The first and second rows have identical left and right horizontal boundaries.
* [ ] The horizontal coverage x-axis includes the reference ID in the format `reference XYZ position (bp)`.
* [ ] The title reads exactly `sample ID: Coverage and read ANI by 100bp bin`.
* [ ] The y-axis label reads exactly `Read depth`.
* [ ] The legend label reads exactly `Read ANI (%)`.
* [ ] The `Mean coverage per bin` rectangle/annotation has been removed.
* [ ] No unrelated plots, labels, data, or styling are changed.
* [ ] The resulting figure remains visually balanced and aligned with the existing design.
