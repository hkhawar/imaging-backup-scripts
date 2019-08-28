
1: Clone the repository into the software folder on EFS from the following link

https://github.com/broadinstitute/imaging-backup-scripts.git


2:cd imaging-backup-scripts


3: Define Variables
Example:
PROJECT_NAME= 2015_10_05_DrugRepurposing_AravindSubramanian_GolubLab_Broad
BATCH_ID = 2016_04_01_a549_48hr_batch1


4: This step will first unlock backend files to later available for download from Glacier

parallel \
  --results restore \
  -a list_of_plates.txt \
  ./glacier_restore.sh \
  --project_name ${PROJECT_NAME} \
  --batch_id ${BATCH_ID} \
  --plate_id {1}\
  --get_backend \
  --check_status


  Output of this step are logfiles "stdout" which are stored in the following path restore/1/stdout


5: Download backend files from Glacier

1st Step (To collect URLs)

1) cd "imaging-backup-scripts"
2) ./get_archive_urls.sh --logpath "Userdefined Path to logfiles" --plates "Userdefined list_of_plates.txt"

Note: All URLs will be downloaded in an output file name "url_list.txt"

2nd Step (Downloading Files from URLs)

parallel -a url_list.txt aws s3 cp .


Alternatively, following strategy which bypass 1st step also works

parallel -a list_of_plates.txt aws s3 cp s3://imaging-platform-cold/imaging_analysis/${PROJECT_NAME}/plates/${PROJECT_NAME}_${BATCH_ID}_{1}_backend.tar.gz .


6: UZip tar.gz files

parallel -a list_of_plates.txt tar -xvzf ${PROJECT_NAME}_${BATCH_ID}_{1}_backend.tar.gz


7: Sync to S3 bucket

parallel -a list_of_plates.txt \
  aws s3 sync \
    ${PROJECT_NAME}_${BATCH_ID}_{1}/${PROJECT_NAME}/workspace/backend/${BATCH_ID}/ \
    s3://${BUCKET}/projects/${PROJECT_NAME}/workspace/backend/${BATCH_ID}/

8: Sync from S3 to local

aws s3 sync \
    --exclude "*.sqlite" \
    s3://${BUCKET}/projects/${PROJECT_NAME}/workspace/backend/${BATCH_ID}/ /Users/habbasi/Desktop/2016_03_14_TargetID_Wagner_Schenone_BWH/2015_10_05_DrugRepurposing_AravindSubramanian_GolubLab_Broad/2016_04_01_a549_48hr_batch1
 /Users/habbasi/Desktop/2016_03_14_TargetID_Wagner_Schenone_BWH/2015_10_05_DrugRepurposing_AravindSubramanian_GolubLab_Broad/2016_04_01_a549_48hr_batch1






