# Create a AWS Stack including DataBrew project and supporting resources. 

Using this template allows a user to deploy a DataBrew project and supporting DataBrew resources.  Additionally, it supplies a few work-arounds to some DataBrew shortcomings.  One is a Lambda function, that makes some corrections to the incoming data that DataBrew cannot correct, such as embedded Escape characters or double quotes.  Another is a Lambda function that moves processed DataBrew input files to an archive location, after a job run.

Various S3 storage locations are used to storing the incoming data at each stage.  The user can specify existing bucket names and paths, or create new buckets by setting the "Create Buckets?" parameter to true.

The Input Bucket/Path location is the landing place for new data. When the new data arrives, a File Preprocessing Lambda will be automatically triggered, performing a set of selected transformations to prepare the data for DataBrew. The preprocessed output is stored in the Post Processing location for input into DataBrew, where the template-specified resources will complete the data cleansing.  The same bucket can be used provides the paths differ for each step. 

# Installation

To install the template, log into AWS CloudFormation and click on _Create stack_ to load the DataBrewProcessingTemplate.yaml template.

One parameter of note is the "Create Buckets?". This parameter is determines if the buckets will be created during deploy.  If not, existing bucket names must be used.  The other parameter "Archive Processed Files?" will move files from the Process Files location to the Archive location.

Note: To complete the deployment of the DataBrew project a file must exist in the dataset location.  Therefore, a lambda creates a file dummy_data_brew_file.csv at this location.  Be sure to remove this file when valid data is ready to process.

Optionally, use the AWS CLI tool to deploy:
```
aws cloudformation create-stack --stack-name DataBrewSampleStack --template-body file://DataBrewProcessingTemplate.yaml --capabilities CAPABILITY_NAMED_IAM
```

## Overall File Preprocessing

For the pre-DataBrew preprocessing, the script provides an option to remove embedded backslash/escape character. This character can significantly disrupt DataBrew from interpreting the data file. The file type can also be selected.


## CSV File Preprocessing

For CSV files there are some settings as well as other optional transformations to improve the data file.

# Resources

Some of the resources this template creates are:

- An automated script to fix up data issues DataBrew cannot handle, included:
  - Embedded Escape Chars
  - Embedded Double Quotes
  - Hashtags in CSV headers
  - Spaces in CSV headers
  - Leading and trailing spaces in CSV headers
- An automated script to move files to an archive location once DataBrew has successfully processed the files
- An eventbridge rule to trigger the above mentioned script when a DataBrew process has completed
- A DataBrew Dataset referencing an S3 location.
- A DataBrew Recipe that:
  - Removes Double Quotes from:
    - Address
    - Email
    - Account\_Number
  - Remove Dollar Sign and Comma from Sales\_Amt
  - Changes Zipcode data type to String
- A DataBrew Project
- A DataBrew Ruleset which checks that Zipcode is 5 digits or 9 digits with hyphen
- A DataBrew Job that generates output of the data using the DataSet and Recipe
- A DataBrew Schedule that triggers the Job. It is loaded with a date set in the past to be modified later by the developer.

The Recipe steps are hardcoded in the template but can easily be modified by the user before deployment. The initial template removes some characters for specified fields and changes the data type of another. These were the commonly requested resolution. Modify was you see hit. If other modifications are requires, see the DataBrew documentation or use the DrawBrew console to modify the recipe and then export the YAML file to include in the template.

