# Column definitions

| | |
|---|---|
| Release | `2026-10-28-statewide.3`, schema version 6 |
| What is counted | Every count is a count of entries as printed on the roll. One person can have more than one entry. |
| Missing values | A count is blank for a part we have not received. A count is zero for a processed page with no entries. Any other value is blank when it was missing, unreadable or invalid. |
| Hashed columns | `epic_hash` and `household_hash` are keyed one-way hashes (HMAC-SHA256, first 24 hex characters) made with key id `c889fbc0`. We keep the key secret. Two hashes can be compared only within releases that share the same key id. |

The column names call an entry a card (`card`) or an occurrence (`occurrence_id`). A part is a polling station section, which the ECI publishes as one PDF.

## Tables

| Table | One row is | Key |
|---|---|---|
| [electors](#electors) | One entry on a processed page | `occurrence_id` |
| [summary](#summary) | One area in one roll edition: the state, a district, a constituency or a part | `snapshot_id`, `level`, `district_code`, `ac`, `part` |
| [population_age](#population_age) | One constituency, age, gender and status | `snapshot_id`, `ac`, `age`, `gender`, `status` |
| [roll_changes](#roll_changes) | One constituency, status, deletion code, gender and age band | `snapshot_id`, `ac`, `status`, `deletion_code`, `gender`, `age_band` |
| [relationship_profile](#relationship_profile) | One constituency, relationship, gender and age band | `snapshot_id`, `ac`, `relationship`, `gender`, `age_band` |
| [data_quality](#data_quality) | One constituency in one roll edition | `snapshot_id`, `ac` |
| [corrections](#corrections) | One entry with an approved review decision | `snapshot_id`, `occurrence_id` |
| [parts](#parts) | One expected part in one roll edition, received or not | `snapshot_id`, `ac`, `part` |
| [constituencies](#constituencies) | One assembly constituency | `snapshot_id`, `ac` |
| [districts](#districts) | One ECI electoral district | `snapshot_id`, `district_code` |

## Columns shared by several tables

These columns mean the same thing wherever they appear.

| Column | Type | Definition |
|---|---|---|
| `snapshot_id` | text | Identifier of the roll edition. |
| `occurrence_id` | text | Identifier of one entry on the roll. It stays the same across releases. |
| `district_code` | text | ECI electoral district code. Blank when we could not match the constituency to a district. |
| `district` | text | ECI electoral district name. |
| `ac` | integer | Assembly constituency number, from 1 to 224. |
| `constituency` | text | Assembly constituency name, as in the ECI's list of constituencies. |
| `part` | integer | Part number within the constituency. |
| `part_name` | text | Name of the polling station, as printed on the roll. |
| `serial` | integer | Serial number printed on the entry. Blank when unreadable. Together with `ac` and `part`, it finds the entry in the ECI's PDF. |
| `age` | integer | Age in years as printed, from 18 to 120. Blank when the age was missing, unreadable or outside that range. |
| `age_band` | text | Age band: 18-24, 25-34, 35-44, 45-54, 55-59, 60-64, 65-74, 75-84, 85+ or unknown. |
| `gender` | text | Gender as printed on the roll: male, female, third_gender, transgender, other or unknown. |
| `relationship` | text | The relative named on the entry: father, husband, mother, wife, guardian, other or unknown. |
| `status` | text | The list in the roll that the entry is printed in: base (the mother roll), addition, deletion or modification. |
| `deletion_code` | text | Deletion code printed on the entry: E (expired), S (shifted), Q (disqualified), R (repeated) or M (missing). Blank when no code is printed. |
| `review_flags` | list of text | Problems our checks found on this entry, from this list: missing_epic, epic_format, missing_serial, missing_name, missing_age, age_out_of_range, missing_gender, low_ocr_confidence, unreadable_status_marker. An empty list means the checks found no problem. |
| `records` | integer | Number of entries in this group. |
| `expected_parts` | integer | Number of parts in the ECI's list for this area. |
| `received_parts` | decimal | Number of expected parts we have downloaded and read. |
| `complete_parts` | decimal | Number of expected parts with every page processed. |
| `matched_parts` | decimal | Number of complete parts whose entry counts match the totals printed in the PDF. |
| `processed_pages` | decimal or integer (varies by table) | Number of pages we have read entries from. |
| `needs_review_pages` | decimal or integer (varies by table) | Number of processed pages marked for review. |
| `printed_net_electors` | decimal or integer (varies by table) | Net number of electors printed on the summary page of each part, added up over the area. This is the ECI's own count. |
| `observed_records` | decimal or integer (varies by table) | Number of entries on processed pages, including deletions and modifications. |
| `non_deleted_records` | decimal | Number of entries with status base, addition or modification. |
| `additions` | decimal | Number of entries printed as additions in this edition. |
| `deletions` | decimal | Number of entries printed as deletions in this edition. |
| `modifications` | decimal | Number of entries printed as modifications (corrections) in this edition. |
| `flagged_records` | decimal | Number of entries with at least one review flag. |
| `flagged_pct` | decimal | 100 × flagged_records / observed_records. |

## electors

Every entry we read from the roll, with where it is printed, age, gender, relationship, status and our review flags. We removed the name, the relative's name, the house number and the voter ID. In their place, `epic_hash` and `household_hash` let you tell when entries share a voter ID or a house number.

Shared columns: `occurrence_id`, `district_code`, `ac`, `part`, `serial`, `age`, `gender`, `relationship`, `status`, `deletion_code`, `review_flags`.

| Column | Type | Definition |
|---|---|---|
| `page` | integer | Page of the source PDF on which the entry is printed. |
| `card` | integer | Position of the entry on its page, in reading order. |
| `section_number` | text | Number of the section, a locality block within the part. Blank when unknown. |
| `epic_hash` | text | Keyed hash of the voter ID. Equal hashes mean equal voter IDs. |
| `epic_series` | text | First three letters of the voter ID, when the ID has the standard form of three letters and seven digits. |
| `household_hash` | text | Keyed hash of the house number within the part. Entries with equal values were printed with the same house number. |
| `confidence` | decimal | OCR confidence for this entry, from 0 to 1. |

In the CSV files, `review_flags` is one text value with the flags separated by `|`.

## summary

Headline counts and rates for the state, each district, each constituency and each part, in one table. The `level` column says which kind of area a row describes.

Shared columns: `snapshot_id`, `district_code`, `district`, `ac`, `constituency`, `part`, `part_name`, `expected_parts`, `received_parts`, `complete_parts`, `matched_parts`, `processed_pages`, `printed_net_electors`, `observed_records`, `non_deleted_records`, `additions`, `deletions`, `modifications`, `flagged_records`, `flagged_pct`.

| Column | Type | Definition |
|---|---|---|
| `level` | text | Kind of area the row describes: state, district, constituency or part. |
| `part_coverage_pct` | decimal | 100 × complete_parts / expected_parts. |
| `parts_with_printed_totals` | integer | Number of parts whose printed totals are included in printed_net_electors. |
| `electors_per_complete_part` | decimal | non_deleted_records / complete_parts. |
| `additions_per_1000` | decimal | 1000 × additions / non_deleted_records. |
| `deletions_per_1000` | decimal | 1000 × deletions / non_deleted_records. |
| `modifications_per_1000` | decimal | 1000 × modifications / non_deleted_records. |
| `male_records` | decimal | Number of non-deleted entries with gender male. |
| `female_records` | decimal | Number of non-deleted entries with gender female. |
| `other_gender_records` | decimal | Number of non-deleted entries with third_gender, transgender or other as gender. |
| `known_gender_records` | decimal | Number of non-deleted entries with a gender other than unknown. |
| `female_pct` | decimal | 100 × female_records / known_gender_records. |
| `sex_ratio` | decimal | Females per 1,000 males among non-deleted entries with a known gender, as the Census of India defines it. |
| `known_age_records` | decimal | Number of non-deleted entries with an age. |
| `mean_age` | decimal | Mean age of the entries counted in known_age_records. |
| `median_age` | decimal | Median age of the entries counted in known_age_records. |
| `age_18_24_pct` | decimal | 100 × non-deleted entries aged 18 to 24 / known_age_records. |
| `age_60_plus_pct` | decimal | 100 × non-deleted entries aged 60 or more / known_age_records. |
| `age_80_plus_pct` | decimal | 100 × non-deleted entries aged 80 or more / known_age_records. |
| `missing_epic_records` | decimal | Number of entries without a well-formed voter ID. |
| `missing_epic_pct` | decimal | 100 × missing_epic_records / observed_records. |

Where `parts_with_printed_totals` is below `expected_parts`, `printed_net_electors` covers only some of the area's parts.

## population_age

Entries in each constituency by single year of age, gender and status. Adding rows gives any age band, district or the state. Entries with no valid age have a blank `age`.

Shared columns: `snapshot_id`, `district_code`, `ac`, `age`, `gender`, `status`, `records`, `flagged_records`.

## roll_changes

Entries printed as additions, deletions or modifications in this edition, by constituency, deletion code, gender and age band. It has no rows for entries in the base roll.

Shared columns: `snapshot_id`, `district_code`, `ac`, `status`, `deletion_code`, `gender`, `age_band`, `records`.

## relationship_profile

Which relative each entry names, by constituency, gender and age band. Only non-deleted entries are counted.

Shared columns: `snapshot_id`, `district_code`, `ac`, `relationship`, `gender`, `age_band`, `records`.

## data_quality

How well we read each constituency: review flags by type, OCR confidence, pages marked for review, and how many parts match the totals printed in their PDFs.

Shared columns: `snapshot_id`, `district_code`, `district`, `ac`, `constituency`, `expected_parts`, `received_parts`, `complete_parts`, `matched_parts`, `processed_pages`, `needs_review_pages`, `observed_records`, `flagged_records`, `flagged_pct`.

| Column | Type | Definition |
|---|---|---|
| `mismatched_parts` | decimal | Number of parts whose entry counts differ from the printed totals. |
| `unavailable_total_parts` | decimal | Number of parts whose printed totals we could not read. |
| `missing_epic` | decimal | Number of entries flagged missing_epic. |
| `epic_format` | decimal | Number of entries flagged epic_format. |
| `missing_serial` | decimal | Number of entries flagged missing_serial. |
| `missing_name` | decimal | Number of entries flagged missing_name. |
| `missing_age` | decimal | Number of entries flagged missing_age. |
| `age_out_of_range` | decimal | Number of entries flagged age_out_of_range. |
| `missing_gender` | decimal | Number of entries flagged missing_gender. |
| `low_ocr_confidence` | decimal | Number of entries flagged low_ocr_confidence. |
| `unreadable_status_marker` | decimal | Number of entries flagged unreadable_status_marker. |
| `mean_confidence` | decimal | Mean OCR confidence across entries, before any correction. |
| `corrections_applied` | integer | Number of entries with an approved review decision, including decisions that kept the original value. Each entry counts once. |

## corrections

Entries that a reviewer has checked against the source PDF, with the fields checked, the fields changed, and the review flags before and after. This release has no rows.

Shared columns: `snapshot_id`, `occurrence_id`, `district_code`, `ac`, `part`, `serial`, `review_flags`.

| Column | Type | Definition |
|---|---|---|
| `original_review_flags` | list of text | Review flags before any correction in this release. |
| `reviewed_fields` | list of text | Fields that a reviewer checked against the source PDF and approved, including fields left unchanged. |
| `corrected_fields` | list of text | Approved fields whose values changed from what we first read. These are a subset of reviewed_fields. |

## parts

Every part the ECI lists, with its source PDF, the totals printed on its summary page, how far we have processed it and the entries we found. Other tables join to it on `snapshot_id`, `ac` and `part`.

Shared columns: `snapshot_id`, `district_code`, `district`, `ac`, `constituency`, `part`, `part_name`, `processed_pages`, `needs_review_pages`, `printed_net_electors`, `observed_records`, `non_deleted_records`, `additions`, `deletions`, `modifications`, `flagged_records`.

| Column | Type | Definition |
|---|---|---|
| `received` | true/false | True when we have downloaded and read the part's PDF. |
| `expected_pages` | integer | Number of pages in the source PDF. Blank until we receive the part. |
| `complete` | true/false | True when we have processed every page of the part. |
| `reconciliation` | text | Result of comparing our entry counts with the totals printed in the PDF: matched, mismatch, unavailable (we could not read the printed totals) or pending. |
| `printed_mother_roll` | integer | Number of mother-roll electors printed on the part's summary page. |
| `printed_additions` | integer | Number of additions printed on the part's summary page. |
| `printed_deletions` | integer | Number of deletions printed on the part's summary page. |
| `printed_modifications` | integer | Number of modifications printed on the part's summary page. |
| `source_url` | text | Address of the ECI's PDF for this part. |
| `source_sha256` | text | SHA-256 checksum of the source PDF. |

## constituencies

The 224 assembly constituencies, each with its ECI electoral district and its count of parts.

Shared columns: `snapshot_id`, `district_code`, `district`, `ac`, `constituency`, `expected_parts`, `received_parts`, `complete_parts`.

## districts

The 34 ECI electoral districts. Their boundaries can differ from Karnataka's administrative districts.

Shared columns: `snapshot_id`, `district_code`, `district`, `expected_parts`, `received_parts`, `complete_parts`.

| Column | Type | Definition |
|---|---|---|
| `constituencies` | integer | Number of constituencies in the district. |
