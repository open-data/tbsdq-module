# TBS Data Quality
This module adds reusable Data Quality components for Python based projects based on the Data Quality metrics defined by the Treasury Board Secretariat for the Goverment of Canada's Open Data portal.

* [Metadata Quality Metrics](#metadata-quality)
* [Resource Quality Metrics](#resource-quality)
* [Installation](#installation)
* [Usage](#usage)
* [Sample Project](#sample-project)

## Metrics
This module calculates which achievements a particular dataset should be awarded.  The metrics are broken up into two categories: Metadata Quality and Resource Quality.

### Metadata Quality

> #### Timely
> The Timely achievement is awarded if an update to the data has occured within the frequency specified during initial publishing.  > There is a 25% grace period applied to each interval within which a point will still be awarded even in the absence of an update.  > For example, if the specified update frequency is hourly, then the dataset must be updated within 75 minutes of the last update in > order to maintain this achievement.
> 
> There are a handful of update frequencies that cannot be evaluated effectively that will automatically be awarded this achievement > (e.g. 'as needed', 'irregular', 'unknown').

> #### Literate
> The Literate achievement is awarded if any valid supporting documentation is supplied alongside the data in the package.  Supporting >documentation is any file tagged as a 'guide', 'terminology', 'faq', 'publication', 'plan', or 'specification'.  The achievement is >awarded once per package and multiple points will not be awarded if multiple supporting documents are supplied.

> #### Connected
> The Connected achievement is awarded if a valid maintainer email address is supplied with the dataset.  Datasets belonging to > departments outside of TBS using the open@tbs-sct.gc.ca email address will not be awarded this achievement.  Publishers are > encouraged to create and maintain their own central inbox for their department's open data.

> #### Communicator
> The Communicator achievement is awarded if the main package description is at or below an appropriate readability level.  There is a separate achievement for French and for English so two Communicator achievements can be earned for each package.  Readability is calculated using the Flesch-Kincaid readability scale and descriptions at or below grade 8 will be awarded this achievement.


### Resource Quality

> #### Participation
> The Participation achievement is awarded if the provided link to the dataset is valid (i.e. returns a valid HTTP status and not 404, 500, 503, etc.)

> #### Tidy
> The Tidy achievement is awarded if the dataset is successfully validated via a linting tool and returns no errors.

> #### Honest
> The Honest achievement is awarded if the supplied data file matches the indicated file type.  For example, if a dataset is specified as being a CSV file when published, the link should return a CSV file and not a ZIP.

> #### International
> The International achievement is awarded if the dataset is supplied with UTF-8 character encoding.

## Installation
During initial development, you will need to install this module from this github repository.

### Pip Install
To add this to a project, execute the following command in your virtual environment:
    
    pip install git+https://github.com/msisktbs/tbsdq-module

### Adding to requirements.txt
To add this to your requirements.txt, add the following entry:
    
    git+git://github.com/msisktbs/tbsdq-module

## Usage
Once installed, add the following import to your python file:
`from tbsdq import data_quality as dq`

The primary entry point is the `dq_validate` method which accepts a source (i.e. the data to validate) and an execution mode (i.e. single or csv).

    dq.dq_validate(<source>, <execution_mode>)

### Input
Regardless of which `execution_mode` you're using, the `source` must include the following keys/columns:
Key | Description
---------------|-----------------------------
description_en | The package description in English that would be displayed on the main package page when published on the open data portal
description_fr | The package description in French that would be displayed on the main package page when published on the open data portal
owner_org | The full English name of the owning department (e.g. "Treasury Board Secretariat of Canada")
maintainer_email | The contact email address for this dataset (e.g. "some.user@canada.ca")
url | The url that points directly to the data file you would like to validate
source_format | The format of the dataset (e.g. "csv", "xml", "xls", etc.) that you would select when publishing to the open data portal

### Output
Once validation is complete, you will receive a [pandas](https://pandas.pydata.org/) dataframe as a return object.  The dataframe will have the original data you passed in to the `dq_validate` method with the following columns added:

Key | Description
-------------------|-------------------
valid_readability_en | Whether or not your English description is at or below the specified value for readability on the Flesch-Kincaid readability scale returned as a bit
valid_readability_fr | Whether or not your French description is at or below the specified value for readability on the Flesch-Kincaid readability scale returned as a bit
valid_url | Whether or not the provided URL is valid returned as a bit
valid_file_type | Whether or not your file matches the specified file type returned as a bit
valid_encoding | Whether or not your dataset is encoded to the UTF-8 or UTF-8 with BOM standard returned as a bit
valid_format | Whether or not your dataset has successfully passed through a linter returned as a bit
goodtables_report | The full JSON report returned from the [goodtables](http://goodtables.io/) validation

### Single Mode
If you've got a single dataset that you want to validate, you can execute the `dq_validate` method in "single" mode.  The `source` should be a python dictionary that matches the expected input format above.

    from tbsdq import data_quality as dq
    import pandas as pd

    python_dictionary = {
        'description_en': "English Description",
        'description_fr': "French Description",
        'owner_org': "Treasury Board Secretariat of Canada",
        'maintainer_email': "some.user@canada.ca",
        'url': "https://link.to/some.csv",
        'source_format': "csv"
    }

    df_quality = dq.dq_validate(python_dictionary, "single")

    print(df_quality.head(10))


### CSV Mode
If you've got multiple datasets that you want to validate, you can execute the `dq_validate` method in "csv" mode.  The `source` should be a CSV file whose columns match the expected input format above.

    from tbsdq import data_quality as dq
    import pandas as pd

    my_input_csv = '/path/to/your/file.csv'
    my_output_csv = '/path/to/your/output_file.csv'

    df_quality = dq.dq_validate(my_input_csv, "csv")
    df_quality.to_csv(my_output_csv, index=None, header=True, encoding='utf-8')


## Sample Project
The Data Quality project uses this module for both a command-line interface and a simple web form and API to deliver data quality evaluations to users.  You can see that project here: https://github.com/open-data/data