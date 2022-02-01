This page describes in detail the points to be implemented, when curating publications' data for the public instance of [cBioPortal](cbioportal.org) 

- Files should be created following [this documentation](https://docs.cbioportal.org/5.1-data-loading/data-loading/file-formats)
- Files should be named following [this chart](https://github.com/cBioPortal/datahub/blob/master/docs/recommended_staging_filenames.md)

# Data Sanitation
## Clinical Data
### General
- The clinical file should be split to patient and sample level attribute files and should contain attribute meta headers
- `Oncotree Code`, `Cancer Type` and `Cancer Type Detailed` columns should be present. `Cancer Type` and `Cancer Type Detailed` values should be in accordance to the ONCOTREE CODE definition [here](http://oncotree.mskcc.org/#/home)
- Make sure the sample/patient count match the study description
- Always include matched normal status with column `SOMATIC_STATUS` added
- `TMB` column should be added, generated by [this script](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/tmb/calculate_tmb)
### Header
- Proper capitalization(first letters / camel cases) of the attribute name, description and type. (e.g. `Oncotree Code`) 
- No underscore in attribute name and description
- No hyphen in attribute IDs; use underscore to separate words (e.g. `ONCOTREE_CODE`)
- Use [CDD](http://oncotree.mskcc.org/cdd/swagger-ui.html#!/clinical-data-dictionary-controller/getClinicalAttributeMetadataBySearchTermsUsingPOST
) as reference for name, description and type
- Gender related attributes should all be normalized to `Sex`
- For survival data, please use the same prefix and name the attributes to [PREFIX]_STATUS and [PREFIX]_MONTHS and translate into [new format vocabulary](https://github.com/cBioPortal/datahub-study-curation-tools/blob/master/Survival_Data_Migration/survivalStatusVocabularies.txt)

### Values
- Normalize `SAMPLE_CLASS`: Can only contain values `Tumor`, `Cell Line`, `Xenograft`
- Normalize `SAMPLE_TYPE`: Can only contain values `Primary`, `Metastasis`, `Recurrence`
- Normalize `SEX`: Can only contain values `Male`, `Female`
- Normalize `SOMATIC_STATUS`: Can only contain values `Matched`, `Unmatched`

## Mutation Data
- All variants should be annotated with protein change by [Genome Nexus API](https://www.genomenexus.org/swagger-ui.html), using [this wrapper script](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/GN-annotation-wrapper)
- The Entrez Gene Id value should either be 0 or empty if unknown.
- Make sure no ‘NA’ for ref alleles (https://github.com/cBioPortal/datahub/issues/621)
- The Reference Build should be GRCh37. Do a liftover if needed. If the value are 37/hg19/NA replace with GRCh37. 
- `Mutated` Issue: make sure no case as ‘deletion/insertions annotated as missense mutations’ exist (https://github.com/cBioPortal/datahub/issues/255)
- Fix cases with `HGVSp_short` annotated as `MUTATED`
- When pushing data subsetted from MSKIMPACT, germline mutations should be removed
- If there are germline mutations, if yes, double check with PI to see if it should be kept; otherwise, by default we don’t include germline mutations in public portal. 
- Correct gene symbols convereted to Dates (SEPT13 -> 13-Sept) by Excel, using [this scirpt](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/hugo-symbol-corrector)

## Expression Data
- Create z-score profiles for each expression profile, using [this script](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/zscores/zscores_relative_allsamples); *double check whether the data is already log transformed* 

## Gene Panels
- Gene Panel files should be added in reference_data/gene_panels on datahub
- In The gene panel file name should follow the format `data_gene_panel_(panel_name).txt`
- A readme file is added into the study folder, to indicate the download location for all gene panel files
- Calculate the profiled coding base by running [this script](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/tmb/calculate_number_of_profiled_coding_base_pairs), which would add a field in the panel file called `number_of_profiled_coding_base_pairs`

## Case Lists
- Case lists should always be generated using [this script](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/generate-case-lists)
- Check if the logic of sample numbers is correct in different case lists(e.g. rppa has more sample than all)
- Make sure the number of samples in each case list file corresponds to the sample count/sequenced count from the paper.

## Meta Files
### `meta_study.txt`
- The `Name` field should strictly follow the format: Tumor Type (Institute, Journal Year) e.g. `brca_mskcc_2015`
- The `Description` field should be a one-line statement covering the seq type, tumor normals, # of samples or patients in any specific type of tumor/sample classification, tumor type.
- `Citation` and `PubMed ID` should be included if the corresponding paper is published.
- Do not include `add_global_case_lists field` in metafile since case lists for all should be created.
- Add the keyword “pediatric” into study name and description, if any pediatric samples are involved. 
### Other meta files
- Every data type/profile should have a meta file associated with it.
- The `stable_id` field in each meta file should be defined using [this chart](https://github.com/cBioPortal/datahub/blob/master/docs/recommended_staging_filenames.md)
- Include the platform used in the profile `description` field of meta Methylation and meta Expression files.

# Wrapping Up
- Migrate outdated gene symbols in all files using [this script](https://github.com/cBioPortal/datahub-study-curation-tools/tree/master/gene-table-update/data-file-migration)
- Make sure all the files are stored as text file in character set: `Unicode(UTF-8)` and `Unix(LF)` *Helpful [articles](https://sites.psu.edu/symbolcodes/software/textfile/)*
- Make sure licence is added to the study folder.

# Validation
- Create a PR on cBioPortal Datahub and run the Circle CI. 
- Add reviewer(s) to review the PR