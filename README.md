# NoSQL Challenge
> How can we do data management with unstructured data that cannot necessarily be contained in strict pre-defined schemas?
## Folder Contents
- A `Resources` folder containing just one `establishments.json` file containing almost 40,000 JSON documents.
- A `EatSafeLove_Setup.ipynb` Notebook file that imports the JSON in the aforementioned folder into your computer's local Mongo Server and does some minor data processing.
  - *NOTE: Run this file first. The code in the other Notebook requires you to have done this setup.*
- A `EatSafeLove_Analysis.ipynb` Notebook file that checks out the cleaned up data and answers questions about the data through PyMongo queries.
- A `.gitignore` file that ignores common things like PyCache, Jupyter Notebook checkpoints, and other common gitignorable Python entities. 

### Installation/Prerequisites
- Make sure you can run Python. The development environment I used was set-up with:
```
conda create -n dev python=3.10 anaconda -y
```
#### Imported Modules
- Installing via the conda command given should give you access to most of the script's modules locally. Be sure to grab yourself the following libraries:
  - [Pandas](https://pandas.pydata.org/docs/getting_started/install.html) for basic DataFrame production needs (It's not the main focus of the project).
  - [PyMongo](https://pymongo.readthedocs.io/en/stable/installation.html) to be able to interact with MongoDB.
  - Other modules are built into Python itself.

#### Get MongoDB and Friends
- We'll be using MongoDB as our NoSQL database of choice, so you'll need to have [MongoDB set up](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-windows/) as well as some of its other related tools. Follow official documentation and instructions for the best results.
  - You'll also need to download the [Mongo Shell](https://www.mongodb.com/docs/mongodb-shell/install/) and [MongoDB Database Tools](https://www.mongodb.com/docs/database-tools/installation/installation-windows/).
  - Please follow their documentation and add the `bin` binary files to your computer's `PATH` environment variable.

## Code Breakdown
As usual, most of the reasoning and coding techniques are discussed at length in the comments of the notebook files, so this part is more about the workflow of our NoSQL project. As mentioned in the Folder Contents, please open `EatSafeLove_Setup.ipynb`, run its instructions and code, and then do `EatSafeLove_Analysis.ipynb`'s content. 

### Data Importing

Just to reiterate the instructions in the setup file, please run the following line of code in your terminal to import the data into your local Mongo server:
> This line assumes you're at the *root* of this project's directory.
```
mongoimport --type json -d uk_food -c establishments --drop --jsonArray Resources/establishments.json
```
- Running this line again will also delete the database and reimport the JSON file, which is good for starting over if you accidentally mess up and over-edit the contents of the database.
- If you downloaded [Mongo Compass](https://www.mongodb.com/try/download/compass), you'll be able to find your imported documents by navigating to the `establishments` collection of the `uk_food` database like so:

![Where you can find your imports](https://cdn.discordapp.com/attachments/939673945240637450/1131439489760370729/image.png)

### Querying and Aggregating

Generally, your PyMongo code would follow this pattern:

```python
# Set up Mongo
mongo = MongoClient(port=27017)

# Get the db you want to access
db = mongo["some_db"]

# Get the collection you want to access
col = db["some_collection"]

# General format
col.some_command(query_here, extra_parameters_here)

# Example: Finding what you need
col.find({"some_field":"some_value"})
```
Sometimes, you might need to [aggregate things](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/) and [export the results to Pandas](https://stackoverflow.com/a/27617290):
```python
# Filters for documents that meet some requirement
match = {
  "$match":{
    "some_field":"some_condition"
  }
}

# Pandas Groupby, but for Mongo
group = {
  "$group":{
    "_id":"what_you're_grouping_by",
    "Result Name":{"$some_aggregation_accumulator":"some_parameter"}
  }
}

# Sorting: x=1 for ascending order, -1 for descending order
sort = {
  "$sort":{
    "Result Name":x
  }
}

# Saving the results as a list since the query itself is a "cursor", which does nothing for you
results = list(col.aggregate([match, group, sort]))

# Export results to Pandas DataFrame
df = pd.DataFrame(results)
```
It works so smoothly because, by definition, you can turn a list of dictionaries (what JSONs basically are) into a Pandas DataFrame.

## Resources that helped a lot
- Official documentation always helps, but do be aware that [Mongo Shell commands](https://www.mongodb.com/docs/manual/) are slightly different from [PyMongo commands](https://pymongo.readthedocs.io/en/stable/index.html). For example, `findOne()` in Mongo is `find_one()` in Python. `$operators` in Mongo are not strings while `"$operators"` in Python are.
  - However, as they are *similar* enough, knowledge here is fairly interexchangeable. I would get started by studying a MongoDB tutorial such as this one by [Web Dev Simplified](https://www.youtube.com/watch?v=ofme2o29ngU) first, then by referencing PyMongo documentation or Google what you want to do as you convert your Mongo knowledge over to PyMongo code.
- Also do note that Mongo has fairly specific notation. Consider the `$count` operator: It can be an "[aggregation accumulator](https://www.mongodb.com/docs/manual/reference/operator/aggregation/count-accumulator/#mongodb-group-grp.-count)" or an "[aggregation pipeline stage](https://www.mongodb.com/docs/manual/reference/operator/aggregation/count/)". Those are not the same things and cannot be applied the same way, so additional study and reference is required.

## FINAL NOTES
> Project completed on July 13, 2023
