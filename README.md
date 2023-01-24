# Data-Camp-with-Python-2nd-project
# The GitHub History of the Scala Language
## Scala's real-world project repository data
#### Importing pandas
import pandas as pd
#### Loading in the data
pulls_one = pd.read_csv('datasets/pulls_2011-2013.csv')
pulls_two = pd.read_csv('datasets/pulls_2014-2018.csv')
pull_files =pd.read_csv('datasets/pull_files.csv')

## 2. Preparing and cleaning the data
#### Append pulls_one to pulls_two
pulls =pulls_one.append(pulls_two)

#### Convert the date for the pulls object
pulls['date'] = pd.to_datetime(pulls["date"], utc=True)

## 3. Merging the DataFrames
#### Merge the two DataFrames
data = pulls.merge(pull_files, on='pid')

## 4. Is the project still actively maintained?
%matplotlib inline
#### Create a column that will store the month
data['month'] = data['date'].dt.month
#### Create a column that will store the year
data['year'] = data['date'].dt.year
#### Group by the month and year and count the pull requests
counts = data.groupby(['year', 'month'])['pid'].count()
#### Plot the results
counts.plot(kind='bar', figsize = (12,4))

#### Output
![image](https://user-images.githubusercontent.com/108233181/214279611-f426211c-e840-449a-8dd0-5e14c12e522f.png)

## 5. Is there camaraderie in the project?
#### Required for matplotlib
%matplotlib inline
#### Group by the submitter
by_user = data.groupby(['user'])['pid'].count()
#### Plot the histogram
by_user.plot(kind='hist', figsize = (12,4))

#### Output
![image](https://user-images.githubusercontent.com/108233181/214279878-fffa34d5-6bb0-4003-bfc9-4fc1249068d9.png)

## 6. What files were changed in the last ten pull requests?
#### Identify the last 10 pull requests
last_10 = pulls.sort_values('date').tail(10)
#### Join the two data sets
joined_pr = pull_files.merge(last_10, on='pid')
#### Identify the unique files
files = set(joined_pr['file'])
#### Print the results
print(files)

#### Output

![image](https://user-images.githubusercontent.com/108233181/214280255-26af3f0e-84a2-4e1f-a92d-d7ad68eda426.png)
![image](https://user-images.githubusercontent.com/108233181/214280362-2bde3926-2b13-411d-aeec-28f4e541d640.png)

## 7. Who made the most pull requests to a given file?

#### This is the file we are interested in:
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

#### Identify the commits that changed the file
file_pr = data[data['file'] == file]
print(file_pr)
####Output
![image](https://user-images.githubusercontent.com/108233181/214280719-3d69244c-3596-44ad-822f-64f4113179a1.png)
![image](https://user-images.githubusercontent.com/108233181/214280806-2b16387f-a8c7-482c-8dae-ffedc3bdd02a.png)

#### Count the number of changes made by each developer
author_counts = file_pr.groupby('user').count()

####Print the top 3 developers
author_counts.nlargest(3, 'file')
#### Output
![image](https://user-images.githubusercontent.com/108233181/214280871-9abebe63-5bec-4fd1-b31a-e56bb1390880.png)

## 8. Who made the last ten pull requests on a given file?
#### Like in the previous task, we will look at the history of
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'
#### Select the pull requests that changed the target file
file_pr = pull_files[pull_files['file'] == file]
#### Merge the obtained results with the pulls DataFrame
joined_pr = pulls.merge(file_pr, on='pid')
#### Find the users of the last 10 most recent pull requests
users_last_10 = set(joined_pr.nlargest(10, 'date')['user'])
#### Printing the results
print(users_last_10)
#### Output
{'bjornregnell', 'retronym', 'soc', 'starblood', 'xeno-by', 'zuvizudar'}

## 9. The pull requests of two special developers
%matplotlib inline

#### The developers we are interested in
authors = ['xeno-by', 'soc']
#### Get all the developers' pull requests
by_author = pulls[pulls['user'].isin(authors)]
#### Count the number of pull requests submitted each year
counts = by_author.groupby([by_author['user'], by_author['date'].dt.year]).agg({'pid': 'count'}).reset_index()
#### Convert the table to a wide format
counts_wide = counts.pivot_table(index='date', columns='user', values='pid', fill_value=0)
print(counts_wide)
#### Plot the results
counts_wide.plot(kind='bar')

#### Output
![image](https://user-images.githubusercontent.com/108233181/214281813-6682cc9f-91ed-4105-a572-090e511acfbb.png)
## 10. Visualizing the contributions of each developer

authors = ['xeno-by', 'soc']
file = 'src/compiler/scala/reflect/reify/phases/Calculate.scala'

#### Merge DataFrames and select the pull requests by the author
by_author = data[data['user'].isin(authors)]
#### Select the pull requests that affect the file
by_file = by_author[by_author['file'] == file]
#### Group and count the number of PRs done by each user each year
grouped = by_file.groupby(['user', by_file['date'].dt.year]).count()['pid'].reset_index()
#### Transform the data into a wide format
by_file_wide = grouped.pivot_table(index='date', columns='user', values='pid', fill_value=0)
#### Plot the results
by_file_wide.plot(kind='bar')
#### Output
![image](https://user-images.githubusercontent.com/108233181/214282045-4ba3d840-5880-476a-8716-2d13e69d9874.png)
