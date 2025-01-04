# 12_nosql-challenge

Deliverable 1: A Jupyter notebook containing code that imports the data and sets up and updates the uk_food database.<br> 
[NoSQL Setup](https://github.com/wrighang/12_nosql-challenge/blob/main/NoSQL_setup.ipynb)

Deliverable 2: A Jupyter notebook containing code that performs the exploratory analysis queries in the database.<br>
[NoSQL Analysis](https://github.com/wrighang/12_nosql-challenge/blob/main/NoSQL_analysis.ipynb)

# Instructions
The UK Food Standards Agency evaluates various establishments across the United Kingdom, and gives them a food hygiene rating. You've been contracted by the editors of a food magazine, Eat Safe, Love, to evaluate some of the ratings data in order to help their journalists and food critics decide where to focus future articles.

# Requirements

## Part 1: Database and Jupyter Notebook Set Up

- Include the 'mongoimport' command text you used to import establishments.json in a markdown cell at the beginning of your Jupyter notebook file
- The 'mongoimport' command text correctly drops any existing establishments collection before importing establishments.json into MongoDB
- The database is named 'uk_food' and the collection is named 'establishments'
- Correctly imports 'PyMongo' and 'Pretty Print'
- An instance of the 'Mongo Client' is created
- Lists the databases you have in Mongo, which includes 'uk_food'
- Lists the collection(s) in the 'uk_food' database, which includes 'establishments' in the output
- Uses 'find_one()' and 'pprint' to display one document in the 'establishments' collection
- The 'establishments' collection is assigned to a variable

##  Part 2: Update the Database

- The supplied data for the "Penang Flavours" restaurant is correctly inserted into the 'establishments' collection
- A query is performed to find the 'BusinessTypeID' for "Restaurant/Cafe/Canteen" and returns only the 'BusinessTypeID' and 'BusinessType' fields
- The "Penang Flavours" document is updated with the correct value for 'BusinessTypeID'
- A query is correctly performed to delete all the documents in the collection where "Dover Local Authority" is the value for 'LocalAuthorityName'
- A 'count_documents()' check is performed before and after the removal of the Dover documents to ensure the documents were removed
- An 'update_many()' query is performed to convert the latitude and longitude fields from strings to decimal numbers and 'RatingValue' to integers

##  Part 3: Exploratory Analysis

###  Question 1: Which establishments have a hygiene score equal to 20?
> Answer: There are 41 establishments with a hygiene score of 20.
- A query is correctly performed to find the establishments with a hygiene score of 20
- 'count_documents()' is used to list the correct number of documents (answer: 41)
- The first result is printed using 'pprint'
- The results are converted to a Pandas DataFrame and displays the first 10 rows

###  Question 2: Which establishments in London have a RatingValue greater than or equal to 4?
> Answer: There are 33 establishments with London as the Local Authoriy and a Rating Value greater than or equal to 4.
- A query is correctly performed to find the establishments in London with a 'RatingValue' greater than or equal to 4
- The query uses the '$regex' operator to locate the London establishments
- 'count_documents()' is used to list the correct number of documents (answer: 33)
- The first result is printed using 'pprint'
- The results are converted to a Pandas DataFrame and displays the first 10 rows

###  Question 3: What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
> Answer: BusinessName: Howe and Co Fish and Chips - Van 17
>> RatingValue: 5
>> Hygiene: 0

> Answer: BusinessName: Atlantic Fish Bar
>> RatingValue: 5
>> Hygiene: 0

> Answer: BusinessName: Plumstead Manor Nursery
>> RatingValue: 5
>> Hygiene: 0

> Answer: BusinessName: Iceland
>> RatingValue: 5
>> Hygiene: 0

> Answer: BusinessName: Volunteer
>>RatingValue: 5
>> Hygiene: 0

- A query is correctly performed to find the establishments within 0.01 degree of the "Penang Flavours" restaurant
- The query also limits the results to establishments with a 'RatingValue' of 5
- The query uses the 'sort()' method in 'PyMongo' to sort in ascending order on the hygiene score
- The query uses the 'limit()' method in 'PyMongo' to limit the results to 5
- All five results are printed using 'pprint'
- The results are converted to a Pandas DataFrame and displayed

### Question 4: How many establishments in each Local Authority area have a hygiene score of 0? Sort the results from highest to lowest, and print out the top ten local authority areas.
> Answer: 55 establishments
- An aggregation pipeline is built to include a 'match' query, 'group', and 'sort'
- The 'match' query matches documents with a hygiene score of 0
- The 'group' step of the pipeline is grouped on 'LocalAuthorityName' and counts the number of documents
- The 'sort' step of the pipeline sorts the count of the documents in descending order
- The aggregation pipeline is correctly sent to the 'aggregate()' method
- The results from the aggregation query is cast as a list and then saved to a variable
- The first ten results are printed using 'pprint'
- The results are converted to a Pandas DataFrame and displays the first 10 rows

##  Deployment and Submission

- Submit a link to a GitHub repository that’s cloned to your local machine and contains your files
- Use the command line to add your files to the repository
- Include appropriate commit messages in your files

##  Comments

- Be well commented with concise, relevant notes that other developers can understand



# CODING_PROCESS

Overall Approach

- I referenced the class activity files for this module to understand syntax and code structure. These resources provided a foundation for resolving various challenges encountered during the assignment.

## Setup File

- To update two fields (`latitude` and `longitude`) and convert their values to doubles, I looked up the correct setup for the code:

         establishments.update_many( {},    [{
        '$set': {
            'geocode.longitude': {'$toDouble': '$geocode.longitude'},
            'geocode.latitude': {'$toDouble': '$geocode.latitude'}
        }    }])

- To inspect the data types of specific fields within the collection, I retrieved one document and iterated through its fields:

        document = uk_food.establishments.find_one()

        for field, value in document.items():
        print(f"{field}: {type(value).__name__}")

- To look at specific field dataypes in a cllection, I reasearch online what code to use:

        document = uk_food.establishments.find_one()

        for field, value in document.items():
        print(f"{field}: {type(value).__name__}")

## Analysis File

- While constructing `hygiene_count`, I encountered asyntax issue, which I resolved by putting the `hygiene_query` in parenthesis:

        hygiene_count = establishments.count_documents(hygiene_query)

- When building the `long_lat_query`, I had trouble with the code structure and how best to create it:

        long_lat_query = {'geocode.longitude': {'$gte': longitude - degree_search, '$lte': longitude + degree_search},
                'geocode.latitude': {'$gte': latitude - degree_search, '$lte': latitude + degree_search},
                'RatingValue' : 5}

- To group the results by `LocalAuthorityName`, I initially encountered errors due to omitting the underscore in `_id`. After troubleshooting, I used the following corrected code:

        match_hygiene_score = {'$match': {'scores.Hygiene': {'$eq': 0}}}

        group_local_authority = {
            '$group': {
                '_id': '$LocalAuthorityName', 
                'count': { '$sum': 1 }}}
