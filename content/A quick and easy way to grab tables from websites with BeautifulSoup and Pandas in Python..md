---
title: A quick and easy way to grab tables from websites with BeautifulSoup and Pandas in Python.
---

For those of you who are somewhat familiar with `BeautifulSoup` and are comfortable enough to read throgh the script and comments, the script I used is as follows. The hints in the comments should guide you through the script. If you want a step by step breakdown, I have put it below the whole script.

Here is the code without comments. (The code with comments is right below)

```

import requests
from lxml import html
from bs4 import BeautifulSoup
import pandas as pd


headers_1 = {'Accept-Encoding': 'identity', 'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1)'} 
connection_one = requests.get('http://www.onekp.com/public_data.html', headers = headers_1) 
soup_object = BeautifulSoup(connection_one.content,features="lxml") 

table = soup_object.find('table') 
rows = table.findAll(lambda tag: tag.name=='tr') 

l =[] 

for tr in rows: 
    td = tr.find_all('td') 
    row = [] 
    for tr in td: 
        
        try: 
            ref = tr.find('a',href=True) 
            i = ref['href'] 
            row.append(i) 
        except: 
            i = tr.text
            row.append(i)
    l.append(row)
df = pd.DataFrame(l) 

df.to_csv('Onekp_data.tsv',sep='\t') 


```

Here is the code with comments.

```
# the following libraries are need for this script to work
import requests
from lxml import html
from bs4 import BeautifulSoup
import pandas as pd

# The following three lines will make the code able to access the file on the internet. The following three lines currently work but I am not sure if the setup is sustainable. It is a patchwork of patchworks that may or may not keep working.

headers_1 = {'Accept-Encoding': 'identity', 'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1)'} # this is need to save us from being hit with captcha
connection_one = requests.get('http://www.onekp.com/public_data.html', headers = headers_1) # this is the website
soup_object = BeautifulSoup(connection_one.content,features="lxml") # this create a soup object that we will parse

table = soup_object.find('table') # The table that I am working with has only one so I am using find but you can use findAll to find tables.

rows = table.findAll(lambda tag: tag.name=='tr') # This converts each row in my table into a row that can be parsed. I understand 'tr' to mean table row in html but not sure. 

l =[] # this is the list that we will make a dataframe out of.

for tr in rows: # here we are going through the loopable 'rows' object that we made a second ago
    td = tr.find_all('td') # This goes through the individual rows in the tr objects and makes the members of the rows loopable.
    row = [] # this is to hold each new row items and emptied out each time for a new row to come in.
    for tr in td: # we made an object td that has all the items in each row, but those are html lines and not the information we need or at least it has more information than we really need.
        #ref= tr.find_all('a',href=True)
        try: # using the try method in python makes it much easier to handle error. Errors such as empty cells, which ironaically I have not handled here but plan to do in the future.
            ref = tr.find('a',href=True) # this takes the individual item in thw row and makes it possible to go through the elements of the item, parsing HTML is wild stuff!
            i = ref['href'] # href stand for the hyperlink refernce which is what I need. Otherwise you will only get the text with 'tr.text'. Some of my columns have hypterlinks, some of my columns have just text and I need to work around that. 
            row.append(i) # append it to the 'row' object that we made above
        except: # some of the rows are text only so the ones that are rejected are sent below to have their text extracted
            i = tr.text # extracts text
            row.append(i) # appends item
    l.append(row) # append the row
df = pd.DataFrame(l) # makes a dataframe

df.to_csv('Onekp_data.tsv',sep='\t') # this one makes a dataframe in the current directory.

```

I was recently looking for a way to grab a table from an html site from the web as a pandas dataframe and what I found on google was not satisfactory. The fact of the matter is `BeautifulSoup` is an amazing package that makes a gargantuan task easy but it is not quite a one-liner package like `Pandas` and needs a little bit of preamble of code to work. So, if what you want to do deviates from tutorials AND if you do not know the aforementioned preamble you will have a not so nice time trying to do something as simple as making a dataframe out of a table on a website.

So, I have documented a method that might make it easy for you to get this specific task done.

Let's go through the steps then.

#### Import the packages that we will need.

We will need the following packages for this to work. Use `pip` to install the ones you are missing with `pip install {insert package name here}`.

```
# the following libraries are need for this script to work
import requests
from lxml import html
from bs4 import BeautifulSoup
import pandas as pd
```

#### Now give the script the ability to servers think it is a browser.

If I understood this correctly, servers can tell if they are being accessed by automated tools and want to discourage this (or something along the lines of that) and as such we have to make our script pretend it is a browser when it contacts the server or url of interest. The code below helps us do this.

Here, try to leave the `headers_1 = {'Accept-Encoding': 'identity', 'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1)'}` line as is. This gives your script a mask that it can use on the server.

As for the `url`, you can supply your own url as the first parameter to the `connection_one` object. You will then supply the `connection_one` object to BeautifulSoup itself.
```
# The following three lines will make the code able to access the file on the internet. The following three lines currently work but I am not sure if the setup is sustainable. It is a patchwork of patchworks that may or may not keep working.

headers_1 = {'Accept-Encoding': 'identity', 'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1)'} # this is need to save us from being hit with captcha
connection_one = requests.get('http://www.onekp.com/public_data.html', headers = headers_1) # this is the website
soup_object = BeautifulSoup(connection_one.content,features="lxml") # this create a soup object that we will parse
```

#### Get the tables on the website.

You can/should get tables from a webpage ,among others, two of the following ways:

Using `find`

```
table = soup_object.find('table') # The table that I am working with has only one so I am using find but you can use findAll to find tables.

```

OR

The website in this demo only has one table so find is fine but if your website has multiple tables use `findAll` (The `A` is capital).

```
table = soup_object.findAll('table')
```

If you had multiple tables, your tables would be indexed into a list and the first table would be accessed as `table[0]`, the second one as `table[1]` and so on.

HTML element are strctured/named with a start tag, a end tag and some content in-between as

```
<table>

{some data here}

</table>
```
And what we are doing with the `soup_object.find('{insert tag here}')` is to search and locate all the elements named table in our soup object.

#### Now to go through a table

```
rows = table.findAll(lambda tag: tag.name=='tr') # This converts each row in my table into a row that can be parsed. I understand 'tr' to mean table row in html but not sure. 

```
Your table in HTML is just like any other table, it is made up of columns and rows. And rows in a HTML table are defined with a `<tr>` start and a `</tr>` end tag and just like a table tag, we can search for a row tag INSIDE of a `BeautifulSoup` table object.

So, this line of code goes through our designated table and find all the row elements and assigns each row to the list called `rows`, a loopable list.

#### Finally going through the cells of the rows.

Now the remainder of the code is somewhat of a monolith that does a few things at the same time. Let's go through it.

```
l =[] 

for tr in rows: 
    td = tr.find_all('td') 
    row = [] 
    for tr in td: 
        
        try: 
            ref = tr.find('a',href=True) 
            i = ref['href'] 
            row.append(i) 
        except: 
            i = tr.text
            row.append(i)
    l.append(row)
```

The function of `l = []` becomes clear later.

Using the `for tr in rows` line we are going through the `rows` object we created earlier, i.e. each cell in the row. BUT that is not enough becuase each cell in each row is another HTML element, specifically the `td` element. 

So, for each `td` element we need to find a way to get into the `td` element and extract meanigful information out of the `HTML` markup. To do this we return to our good friend `find()` and supply it with the `td` parameter as `td = tr.find_all('td')`. This will give us all the elements in the `td` tag and put them in the list `td` for us to loop through.

Now we create an empty list called `row`, to keep our `td` cell elements from each row. Which then sets us up to enter the last loop in our script.

```
for tr in td: 
        
        try: 
            ref = tr.find('a',href=True) 
            i = ref['href'] 
            row.append(i) 
        except: 
            i = tr.text
            row.append(i)
    l.append(row)
```

Here we loop through each cell of each row of each table. We are using the `try/except` method to filter out both `hrefs` (hyperlinks) and text. We try to find hyperlinks in the current cell element with `ref = tr.find('a',href=True)` (making a lsit of all the elements in each cell) and then using `i = ref['href']` to grab the hyperlink. If the hyperlinke is there we append it to the `row` we created in the preceeding line. If not we move on to extract just the text with `i = tr.text` and append the text to the `row`.
We append each `row` to the list `l` we created above. 

#### Making a dataframe out of the list of lists

Now as we append each `row` to the `l` list, we create a massive list of lists, where each element is the row of a table/dataframe. This is something we can feed to `Pandas` to create a dataframe, which is exactly what `df = pd.DataFrame(l)`.

And there you have your website table in pandas dataframe.

#### Save the dataframe to file

You can save your dataframe to file with `df.to_csv('Onekp_data.tsv',sep='\t')`.