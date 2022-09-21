# PhD thesis downloader - Deutsche Nationalbibliothek

This is a Jupyter notebook that downloads PhD disertations from the [Deutsche Nationalbibliothek](https://www.dnb.de/DE/Home/home_node.html). 

The only input required is a search term. The notebook will then download all the PDFs that match the search term. This can be useful for downloading a large number of PDFs for a specific topic to perform a literature review or meta analysis.

Example input (="security"):
```
https://portal.dnb.de/opac/showPreviousResultSite?currentResultId=tit+all+%22security%22%26any%26dnb.hss%26onlinefree
````

## requirements.txt

- `jupyterlab` This is the Jupyter notebook library. Alternatively, the python content can be run in a python script.
- `tqdm` This is a progress bar library
- `p_tqdm` This handles multiprocessing with tqdm
- `beautifulsoup4` This is a library for parsing HTML
- `requests` This is a library for making HTTP requests

Install all of them using the following command:
```bash
pip install -r requirements.txt
```

## Scraping the PDF links

The code is here only for reference. The notebook contains the same code.


```python
from bs4 import BeautifulSoup
from p_tqdm import p_map

html = resp.text
soup = BeautifulSoup(html, "html.parser")
import tqdm

overall_number = int(soup.find_all("span", {"class": "amount"})[0].text.split(' ')[-1])

def poolFunction(i):
    details = (base_url.replace('showPreviousResultSite','showFullRecord')+'&currentPosition='+str(i))
    item = requests.get(details).text
    soup = BeautifulSoup(item, "html.parser")
    js = {}
    for tr in soup.find_all("tr"):
        try:
            href = tr.find_all('td')[1].find('a')
            if href:
                js[tr.find('strong').text.strip()] = href['href']
            else:
                js[tr.find('strong').text.strip()] = tr.find_all('td')[1].text.strip()
        except:
            pass
    return js

#print(poolFunction(1))
r = p_map(poolFunction, list(range(0, overall_number)))
````

## Extracting the PDF links
```python
def getPDF(url):
    if '.pdf' in url:
        print(url)
    else:
        tuprints = requests.get(url).text
        soup = BeautifulSoup(tuprints, "html.parser")
        baseurl = '/'.join(url.split('/')[:3])
        hrefs = soup.find_all('a')
        pdfs = {}
        for href in hrefs:
            if href.has_attr('href') and 'pdf' in href['href']:
                pdfs[href['href']]=True
            if href.has_attr('class') and href['class'] == 'downloadTextLink':
                pdfs[href['href']]=True
        if not len(pdfs):
            print('WARNING',url)
        for pdf in pdfs.keys():
            if pdf.startswith('http'):
                print(pdf)
            else:
                print(baseurl+pdf)

for element in r:
    if 'URL' not in element or 'http' not in element['URL']:
        link = (element['Online-Zugriff'])
    else:
        link = (element['URL'])
    if 'd-nb.info' not in link:
        print(link)
        try:
            getPDF(link)
        except:
            pass
````

## Downloading the PDFs
The previous code will print all the PDF links. You can then copy them into a file called `pdfs.txt` and download them using `wget`:
```bash
wget -i pdfs.txt
```