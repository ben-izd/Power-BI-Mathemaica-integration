# Mathematica integration with Microsoft Power BI
Since 2018 Microsoft Power BI has the ability to connect with R and python. Now you can use Mathematica to create visualization inside Power BI or send data from Mathematica to Power BI. This repository consists of two code:
- [`viz_server.nb`](https://github.com/beny74483/Power-BI-Mathemaica-integeration/blob/main/viz_server.nb) to use Mathematica visualization inside Power BI
- [`data_server.nb`](https://github.com/beny74483/Power-BI-Mathemaica-integeration/blob/main/data_server.nb) to send Mathematica data to Power BI


## How to send Mathematica Plots/Visualization to Microsoft Power BI
Power BI supports python visualization made by `matplotlib`, this solution will send Mathematica code from Power BI, receive the result as an image, and store it inside a `matplotlib` object.

## How to send Mathematica data to Microsoft Power BI
Power BI supports many data sources, we could either use python or a web interface. This solution will deploy a local server that redirects to your data as `JSON`.

## Requirements:
- Python
- Python `matplotlib` module
- Mathematica

## Setup
Before starting, make sure the python Power BI uses, is the correct one by going to `File` > `Options and Settings` > `Options` > `Python Scripting`

Either download [`viz_server.nb`](https://github.com/beny74483/Power-BI-Mathemaica-integeration/blob/main/viz_server.nb) or copy the code and evaluate them. Now you'll have a local server running on port 38000 (you can change it to any number you like as long as it's accessible)

Inside Power BI insert a new `Python Visual`.

![](https://i.imgur.com/k688zFk.png)

If asks you to enable `Enable script visuals`, click `Enable`.

![](https://i.imgur.com/8pDQpKU.png)

You should also provide at least one Field to be able to write python code so here I'll import a sample data file and drag `Price` Field to `Python Visual`'s `Values` section. If you need any field in your calculation, drag them into `Values` section.

![](https://i.imgur.com/L6YbMd7.png)

In the `Python Script Editor`, paste these codes:
```python
import io
import base64
import matplotlib.pyplot as plt
import json
from urllib import request

# Change this section
PORT = 38000

mathematica_code =  'ListLinePlot[Flatten[arg1]]'

pdata = json.dumps({'script':mathematica_code,'data':{'arg1':dataset.values.tolist()}})
# end section

req = request.Request(f'http://localhost:{PORT}/evaluate',data= pdata.encode('utf-8'),method='POST')
res = request.urlopen(req).read()
result = io.BytesIO(base64.b64decode(res))

img = plt.imread(result)
img_dim = img.shape
img_min = min(img_dim)/5

imgplot = plt.imshow(img,extent=[0,img_dim[1]/img_min,0,img_dim[0]/img_min])
plt.axis('scaled')
plt.axis('off')
plt.tight_layout()
plt.show()
```
Mathematica will receive the request and will evaluate `script` with the data provided in `data` (if is available). You can use any variable name, here we used `arg1` therefore I should use `arg1` in the Mathematica too.

The result of our sample code is:

![](https://i.imgur.com/Pf4oIrI.png)

You could use only the script part without providing any `data`:

![](https://i.imgur.com/Q4S9J0P.png)

### Example 2 - Clustering
Use mathematica `FindClusters` to find clusters for `price` and `quantity` fields:

![](https://i.imgur.com/IEdd2hq.png)

> Warning: Since Mathematica code uses the `ToExpression` function which can easily be manipulated to harm your computer, only run and open script files that you trust.


## Possible Issues
If your script failed to evaluate, error messages will be sent as images to Power BI. Here you see the result of plotting `Sin[x]/0`:

![](https://i.imgur.com/5A84u5J.png)


# Send Mathematica data to Microsoft Power BI


## Requirements:
- Python
- Python `matplotlib` module
- Mathematica
- Enabling `Json Table Inference` in Power BI preview features

## Setup
We are using the web interface to load a JSON table, you need to enable a preview feature to load JSON data as a table by going to `File` > `Options and Settings` > `Options` > `Preview features`, enable `Json Table Inference`.

Either download or copy the [`data_server.nb`](https://github.com/beny74483/Power-BI-Mathemaica-integeration/blob/main/data_server.nb) code and evaluate them. Use the following function to create a JSON data source:
```mathematica
sendToPowerBI[data, headers]
```
Now you have a server redirecting any URL to your `data` as JSON with column names in `headers` on the default port (39000)

In Power BI, add new data source `Web`, type `http://localhost:39000/file.json` or `http://127.0.0.1:39000/file.json` and press `OK`:

![](https://i.imgur.com/vFGIAfI.png)

In the `Power Query Editor` you can preview your data, here data consists of 5 columns with different types (`Date`, `Float`, `Boolean`, `Integer`, `String`):

![](https://i.imgur.com/3RlyOb2.png)

Click `Close & Apply`

> Keep in mind that your data should be a 2-dimensional list in Mathematica.
