---
title: Migrating AGC Sermons to CGCSM Website
cdate: 2022-06-07T13:49:28-04:00
tags:
  - how-to
mdate: 2022-06-08T13:49:28-04:00
publish: true
---
## The Task

This article will document the task of migrating the table of sermons from [AGC's website](http://www.agcweb.org/sermons) over to [CGCSM's website](https://chinesegospelchurch.net/cgcsm/) due to the amalgamation of AGC and CGCS in June of 2022. This will be done using a custom web scraper. The web scraper will be built using Python and using the Python libraries, Requests, and Beautiful Soup.

The web scraper would have to be able to scrap the following data from the [AGC site](http://www.agcweb.org/sermons):
- Date of sermon
- Title of sermon
- Video link to sermon
- Scripture reference
- Bible Gateway link to scripture
- Name of speaker

The data would then be exported into CSV format and then processed in Google Sheet to create a HTML table and put onto the CGCSM website.

## Web Scraping
### Acquiring the HTML Source of the AGC Site
Using the Python libraries Requests, and Beautiful Soup the HTML of the website can be acquired and parsed so that data is easier to manipulate and collect from the AGC website. Each row in the table is then extracted and stored into a list called `rows`.

```python
import requests, bs4
# Using Requests to get HTML of website.
res = requests.get('http://www.agcweb.org/sermons')
res.raise_for_status()

# Parses the HTML using Beautiful Soup.
webpage = bs4.BeautifulSoup(res.text, 'html.parser')

# Select the table and store each row into a list.
table = webpage.find('table', attrs={'class':'goog-ws-list-table'})
table_body = table.find('tbody')
rows = table_body.find_all('tr')
```

### Extracting Data from the First Row

Since each row will contain the same set of data. The algorithm needed to extract the data of interest can be determined by testing it on the first row. The rest of the table can then be processed the same way using a for loop.

```python
# Find all columns in row 1 and put them in a list
cols = rows[0].find_all('td')
```

The data contained in the first row are shown below.

```python
[<td width="20%">2022年5月29日 </td>,
 <td class="goog-ws-list-url" width="20%"><a href="sermons/2022/20220529" target="_new">举目仰望天堂，在地上走正义路</a> </td>,
 <td class="goog-ws-list-url" width="20%"> </td>,
 <td class="goog-ws-list-url" width="20%"><a href="https://www.biblegateway.com/passage/?search=Psalm25&amp;version=CUVS;NKJV" target="_new">诗篇 25</a> </td>,
 <td dir="ltr" width="20%">Mike Whittle 牧师 </td>]
 ```

### Date, Title of Sermon, Scripture Reference, and Speaker
The date, title of sermon, scripture reference, and speaker can be extracted using `text.strip()` on the following `cols` elements.

```python
# Date of sermon
date = cols[0].text.strip()
# Title of sermon
title = cols[1].text.strip()
# Scripture
scripture = cols[3].text.strip()
# Speaker
speaker = cols[4].text.strip()
```

### Bible Gateway Link
The Bible Gateway link to the scripture reference can be collected using `select('a', href=True)[0]['href']` on the scripture column. Since some of the rows have no scripture data, it is placed inside a try except block to handle `IndexError`.

```python
try:
    scripture_link = cols[3].select('a', href=True)[0]['href']
except IndexError:
    scripture_link = ""
```

### YouTube Video Link
The YouTube link to the sermon video is more involved since the hyperlink on the table links to another page on the AGC site that have the [YouTube player embeded](http://www.agcweb.org/sermons/2022/20220529) instead of directly linking to the [YouTube video](https://www.youtube.com/watch?v=j_zW1p6LcvY). Requests and Beautiful Soup will be used to download and parse the HTML to get the YouTube link. Some rows does not contain a YouTube link, so the following code would be put inside a try except block to handle `IndexError`.

```python
# The path to the sermon page on AGC site
link_to_sermon = cols[1].select('a', href=True)[0]['href']
# Requests the HTML of that page
yt_page = requests.get(f'http://www.agcweb.org/{link_to_sermon}')
# Parse the HTML with Beautiful Soup
soup = bs4.BeautifulSoup(yt_page.text, 'html.parser')
# Select the YouTube link
vid_link = soup.select('iframe')[0]['src']
```

The YouTube link to the sermon video is extracted. However, the link is an embed link (https://www.youtube.com/embed/j_zW1p6LcvY?rel=0&wmode=opaque). It does not link to the actual YouTube site. It is sufficient at this point to use the embed link, but since people might be more used to watching YouTube videos on the YouTube site (and having access to the descriptions, comments, channel are helpful as well), the actual YouTube link will be generated.

Fortunately, the embed link contains the video id (`j_zW1p6LcvY`). The YouTube link can be generated by substituting the video id inside of this url string `https://www.youtube.com/watch?v={video_id}`, giving us the video link on the YouTube site (https://www.youtube.com/watch?v=j_zW1p6LcvY). In Python this can be done using RegEx and some string manipulation.

```python
import re
# RegEx pattern to extract the video id.
pattern = 'd\/.+\?'
vid_id = re.search(pattern, vid_link).group(0)[2:-1]
# The YouTube link.
yt_link = 'https://www.youtube.com/watch?v=' + vid_id
```

The data is then appended to the `data` list.

```python
result = [date, title, yt_link, scripture, scripture_link, speaker]
data.append(result)
```

This process can be done over all the rows in the table using a for loop. The data can then be exported in CSV via pandas.

```python
for row in rows:
    ...

import pandas as pd
df = pd.DataFrame(data)
df.to_csv('data.csv')
```

For more details on the code visit the [GitHub repo](https://github.com/leiyu3/agape-scrapper).

## Data into HTML Table on WordPress
The CGCSM website uses WordPress, where a table can be edited using HTML. Therefore, by creating a HTML table with all the rows and columns properly formatted and hyperlinked, it can be directly inserted into WordPress.

### Creating the Table in Google Sheet
Import the CSV file into Google Sheet. Create a new sheet and fill in the header of the table.

Then fill in the first row with the formulas as shown in the figure below. Change `data` to the name of the sheet containing your CSV data. This will take the data from the CSV file and fill it into the new table. The formula  creates a hyperlinks given the label and link. 


Select the first row and drag the blue square on the bottom right down until all of the rows are filled in.

The Google Sheet should look like this: [Agape Sermon Details](https://docs.google.com/spreadsheets/d/1T0oUsJMKK8tODWRyD2moDE7kqK7ND49Wuen-9hhO1CM/edit?usp=sharing).

### Exporting to HTML
Export the table in HTML. Go to File &rarr; Download &rarr; Webpage (.html). Unzip the file and then open the HTML containing the final table. Copy the HTML code for the table by selecting the start of the table tag up to the end of the table tag (use Ctrl-F). It should look something like this.

```html
<table class="waffle" cellspacing="0" cellpadding="0">
    Table content...
</table>
```

### Copying into WordPress
In WordPress paste in the HTML by selecting the *edit as HTML* option on a table. A code block should appear where the HTML can be pasted. 

Switch back to by selecting the *edit visually* option and the table should have all the rows and columns hyperlinked properly. 

## Final Result
Finally, edit the header, and style the table as needed. The final result can be viewed below.

Before: [AGC Site](http://www.agcweb.org/sermons)

After: [CGCSM Site](https://chinesegospelchurch.net/cgcsm/agape/)


