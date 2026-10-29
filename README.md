# Karnataka electoral roll, 2026 Special Intensive Revision

Digitized and tidy dataset of Karnataka's final electoral roll from the 2026 Special Intensive Revision (SIR) by the Election Commission of India. 

Originally published as individual PDFs, called "parts", we've processed 60,923 parts of the English edition, revision 1. Each row is one entry on the roll and this dataset contains 45,643,915 entries across all 224 assembly constituencies in Karnataka. 

**Note**: This data was processed with OCR software and entries have not been manually verified by hand. Each entry comes with a confidence score and review flags to describe potential issues for that piece of data. The `data_quality` table sums these up for each constituency. Therefore, consider this **a provisional release** and subject to further review.

## Number of deletions 

Additions, deletions and modifications relative to this edition. For example, the headline number in the news is [99.32 lakh names struck off the voter list in Karnataka](https://www.thehindu.com/news/national/karnataka/sir-over-9932-lakh-names-struck-off-voter-list-in-karnataka/article71629941.ece), but the deletions in this dataset amounts to around 1,88,324 people only. This difference occurs because an initial 1.07 crore (107.96 lakh) voters were excluded when the draft roll was first published. Following this, a "Claims and Objections" period allowed voters to dispute changes, leading to 10.08 lakh additions and 1.88 lakh final deletions.

To come to the final headline number through independent analysis, this dataset must be compared to a separate dataset of the pre-revision voter rolls. 

## Files

For counts and rates by area, start with `summary`.

| File | One row is |
|---|---|
| `summary` | One area: the state, a district, a constituency or a part |
| `electors` | One entry, in one file for the state or one file per constituency |
| `parts` | One part, with its source PDF and the totals printed in it |
| `population_age` | One constituency, single year of age, gender and status |
| `roll_changes` | One constituency, status, deletion code, gender and age band |
| `relationship_profile` | One constituency, relationship, gender and age band |
| `data_quality` | One constituency |
| `constituencies` | One assembly constituency |
| `districts` | One ECI electoral district |

[DOWNLOADS.md](DOWNLOADS.md) links every file in CSV and Parquet formats. 

[COLUMNS.md](COLUMNS.md) contains definitions for every column. You can also search this release at [sir.diagramchasing.fun](https://sir.diagramchasing.fun).

## Methodology

The ECI publishes the roll at [voters.eci.gov.in](https://voters.eci.gov.in/download-eroll?stateCode=S10). We downloaded every part, read each page with OCR and split the pages into entries. Apple Vision and RapidOCR were used in parallel to efficiently process the large number of entries. 

To prevent misuse, we have removed the name, relatives, name, address and voter ID from each entry. The voter ID has been hashed to prevent identification, but still serves as a unique ID for each entry. If we publish further datasets, this hash will remain consistent to allow comparison. However, part, page and serial number is kept so that it may be reference with the original ECI's PDFs if needed.

## Licence

This data is published under data under [Open Database License 1.0](LICENSE). You may share and adapt it if you credit Diagram Chasing and release any database you build from it under the same licence.

Cite it as:

> Diagram Chasing (2026). *Karnataka SIR 2026 voter rolls: de-identified electoral roll dataset*, release 2026-10-28-statewide.3 [Data set]. https://sir.diagramchasing.fun

## Contact

Report errors and ask questions in the repository's [issues](https://github.com/diagramchasing/karnataka-sir-voter-rolls-data/issues). To report a wrong row, give its `ac`, `part` and `serial`.

Release `2026-10-28-statewide.3`, published 2026-10-29 under the [Open Database License 1.0](LICENSE). 
