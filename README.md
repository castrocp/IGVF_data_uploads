## Example steps for submitting an SEM prediction file to the IGVF sandbox

### 1) Install and configure igvf_utils

`https://sandbox.igvf.org/help/data-submission/submission-by-igvf-utils/`  

#### Installation
`https://github.com/IGVF-DACC/igvf_utils/wiki/Installation`

#### Configuration
Set up environment variables  
`https://github.com/IGVF-DACC/igvf_utils/wiki/Configuration`  

Conda environment is set up on the foxglove server for me as "igvf_utils_env"

### 2) Submit human_donor object  
Objects need to be submitted in a specific order. For example, the prediction_set object requires that the human_donor object already exist.  

The order for submissions: `https://github.com/IGVF-DACC/igvfd/blob/dev/src/igvfd/loadxl.py#L15`  

From Idan: "for predictions that are not sample specific, they should be linked to a HumanDonor via the property "donors"  
"for predictions that are sample specific, you link to Samples via "samples"  
"since in both cases, these are not specific Donor or Samples, there is a boolean property on these objects called "virtual" and it should be set as true"  
"for sample agnostic, Samples property is empty or not used. Instead "donors" is used linking to a HumanDonor that is virtual=true  

a HumanDonor object file for SEM predictions might look something like this:  
`sem_metadata_humandonors.txt`  
```
taxa	virtual    aliases
Homo sapiens	true	alan-boyle:SEM_human_donor
```  
Note: Required objects "IGVF_AWARD" and "IGVF_LAB" are not included here because they are stored globally in .bashrc from the configuration step

Aliases for all objects will have the format `alan-boyle:[some_identifier]`  

Post human_donor object:
`iu_register.py -m sandbox -p human_donor -i /home/castrocp/igvf_uploads/sem_metadata_humandonors.txt -d`  

Running the command with `-d` will run it in "dry run" mode for testing. Remove that to actually submit the file.


### 3) Submit prediction_set object
Example of schema at: `https://data.igvf.org/profiles/prediction_set/`  
Required fields: "lab", "award", "file_set_type", "samples" or "donor"  
"file_set_type" can be one of "pathogenicity", "functional effect", "protein stability", "activity level"  
"samples" should refer to the sample(s) associated with the file set.  
In this case, "sample" is asking if the prediction for a variant is specific to a cell or organ type. For SEM scores, the sample is agnostic.  
For an organ-specific prediction, we would list the UBERON id for the given organ under the "sample" column  

The prediction_set metadata file references the human_donor object submitted in the previous step.  

`sem_metadata_predictionSet.txt`  
```
file_set_type   donors  aliases
functional effect       alan-boyle:SEM_human_donor      alan-boyle:SEM_preds
```

Post prediction_set file:  
`iu_register.py -m sandbox -p prediction_set -i /home/castrocp/igvf_uploads/sem_metadata_predictionSet.txt`  


### 4) Submit tabularfile object
After human_donor and prediction_set objects have been submitted, tabular_file will be similar.
As of (Feb. 7, 2024) the most appropriate "content_type" available was "peaks", so that's what was used here.  

For the required "file_set" in the tabular_file object, use the alias for the prediction_set object it is referencing. Prediction_set is a subgroup of file_set.  

The schemas didn't seem to indicate it, but "assembly" and "description" seemed to be required.  

`sem_metadata_tabularfile_AHR.txt`  
`content_type    file_format     assembly        file_set        md5sum  submitted_file_name     aliases description
peaks   tsv     GRCh38  alan-boyle:SEM_preds    881c4ecdc3248400f50b717dad79721b        /home/castrocp/igvf_uploads/sem_data/AHR_annotations.tsv.gz     alan-boyle:ahr_sem_tabfile      tabular_file object for AHR SEM`  

Post tabular_file metadata file:  
`iu_register.py -m sandbox -p tabular_file -i /home/castrocp/igvf_uploads/sem_metadata_tabularfile_AHR.txt`  








