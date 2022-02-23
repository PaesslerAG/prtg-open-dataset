# PRTG Open Dataset
PRTG is a popular IT infrastructure monitoring software. It provides over 200 sensors for monitoring
of network, system, application and cloud infrastructure. Besides, databases, environment,
IoT data sources and virtually everything else can also be monitored.

With over 200000 installations worldwide, all PRTG installations store daily around 1 billion time series with 
over 100 billion data points.

The present open data set consists of a small subset of about 80 million data points (around 20000 time series), 
gathered by 15 PRTG installations in one month.  

We hope this dataset will be useful for researcher working on algorithms for anomaly detection, forecasting, 
and signal processing. 

## Prior art
There are a number of open outlier detection data sets with time series data.

[Outlier Detection DataSets](http://odds.cs.stonybrook.edu/) aims to be a catalogue of various openly available
datasets for outlier detection. It includes three time series datasets:

### TSDL
[TDSL](https://pkg.yangzhuoranyang.com/tsdl/) by Rob Hyndman and Yangzhuoran Yang contains in total 648 
time series, 6 of them on the topic of computing (all of them 6 representing ISP traffic, aggregated daily, 
hourly or every 5 minutes). License: GPL-3

### Yahoo TSAD dataset
[TSAD](https://yahooresearch.tumblr.com/post/114590420346/a-benchmark-dataset-for-time-series-anomaly)
by Nikolay Laptev, Saeed Amizadeh, Youssef Billawala. The dataset is only avilable for educational organizations,
so we're unable to comment on it. License: proprietary.

### Numenta Anomaly Benchmark
[NAB](https://github.com/numenta/NAB) contains 47 real time series from different areas, including 21 time
series related to IT infrastructure, including some of the time series representing not only a statistical
outlier, but also anomalies resulted in system outages. The usual timing is every 5 minutes.

Besides of these, there are several other, more recent public datasets available:

### Exathlon
[Exathlon](https://github.com/exathlonbenchmark/exathlon) by Vincent Jacob, Fei Song, Arnaud Stiegler, Bijan Rad, 
Yanlei Diao, and Nesime Tatbul is using an interesting approach of artificially
generated real-world data: an artificial Spark application performing a typical data pipeline job has been run
and the real monitoring data has been collected. Some of the applications were perturbed to introduce 
anomalies so that the data can be labeled. The dataset provides 93 multivariate timeseries (over 200000 
individual univariate time series) containing 2283 different monitoring aspects, including application 
metrics of Spark, Java and Hive, and system metrics like CPU and disk load and network traffic. Average 
duration of timeseries is 7 hours, the refresh interval is 1 second. License: CC BY-NC-SA 4.0.

### Hexagon ML/UCR Time Series Anomaly Archive 
[UCR-TSAA](https://wu.renjie.im/research/anomaly-benchmarks-are-flawed/) by Renjie Wu and Eamonn J. Keogh
contains 250 supervised anomaly
detection time series. Most time series constitute real-world data, and some anomalies are natural, while
in the rest of time series they have been artificially introduced into the signal. The time series are
mostly coming from health care and biology domains, NASA, economics and environment monitoring.  

### SKAB
[SKAB](https://github.com/waico/skab) by Iurii D. Katser and Vyacheslav O. Kozitsin consists of 35 
multivariate timeseries (corresponds to 280 univariate time series) of real-world data collected from
an experiment. 

### iops.ai
Tsinghua university performs on their website http://iops.ai/ annual competitions related to anomaly detection
in time series. Unfortunately, we weren't able to register and obtain their datasets.

### Machinery Fault Database
[Machinery Fault Database](http://www02.smt.ufrj.br/~offshore/mfs/page_01.html) by Felipe M. L. Ribeiro
provides 1951 multivariate time series (corresponds to 15608 univariate time series) collected from 
experiments conducted on their Machinery Fault Simulator. 

## Our contributuion
Some of the pre-existing datasets provide only a limited number of time series. When researchers are using 
the same time series over and over again, there is an elevated risk of overfitting to this data. By
providing a reasonably big number of time series, we hope to alleviate this risk. Among the reviewed 
datasets, only Exathlon and Machinery Fault Database have more or similar number of time series, that
in this dataset.

Sometimes, the pre-existing datasets provide data ultimately originating from the same system, for example,
SKAB has reused the same physical setup to generate all the time series. This dataset provides data from 15
different companies or other entities, that vary a lot by their nature - from monitoring of a
private smart home infrastructure to the hybrid on-premise/cloud infrastructure of an 
international corporation.

Some of the pre-existing datasets limit either the time span of the time series by several hours, or the
refresh interval by 5 minutes or longer. The typical refresh interval of the time series presented in this
dataset is 1 minute, and the time span is one month - allowing to discover nested seasonality structures,
for example hourly, daily and weekly seasonalities.

Finally, some of the other datasets are only available upon registration or limited to educational 
institutions, while this dataset is open and available for everyone, without limitation or registration.

## Examples of data
Full data set is available only in the Parquet format in the [data/](data/) folder, for performance reasons. 
Nevertheless, we have manually selected 50 of some "intersting" time series as representative examples.
These examples are available as separate uncompressed TSV files in the [examples/](examples/) folder. 
The naming convention for the files is <example_number>_<sensortype>_<channel>.tsv.

You can see a visualization of these examples [here](examples.ipynb)

## Description of sensors and channels
You can find description of all sensor types and corresponding channels either in this comma-separated
file [metadata/sensor_types_and_channels.csv](metadata/sensor_types_and_channels.csv) that can be
opened by a spreadsheet app, or, for a programmatical access, in the JSON format using 
[metadata/sensor_types_and_channels.json](metadata/sensor_types_and_channels.json)

Conceptually, a PRTG sensor is a piece of code retrieving some metrics from the monitored device. All metrics
are retrieved at the same time, with some defined refresh interval. These metrics are called channels
of the sensor. For example, SNMP Traffic sensor has a channel for total incoming traffic, another channel
for total outgoing traffic, another one for the number of errors, etc. We can see the dataset of one
sensor as a multivariate time series.

## Reading data with Python and Pandas
Here is a minimally working example of how to read Parquet files in Python
    
    import pandas as pd

    df = pd.read_parquet('<path_to_dataset>data/snmptraffic')

The second line above will read the whole month of the channel ping_ping_time of all ping sensors in
the dataset, producing a Pandas DataFrame of size 12919964 rows x 19 columns. 

You can now get just one time series for example like this:

    sensor_ids = df.sensor_id.unique() # getting all sensor ids

    # getting channel ping_ping_time of the first sensor
    time_series = df[df.sensor_id==sensor_ids[0]][['ts', 'snmptraffic_traffic_in_speed']]

Loading all channels of all sensors can be time consuming. If you already know the company_id
and the sensor_id of interest, you can load just data of one sensor using

    df = pd.read_parquet('<path_to_dataset>/data/snmptraffic/company_id=1/sensor_id=170')

and if you are interested only in specific channels, you can speed up the loading even
further by passing only specific columns:

    df = pd.read_parquet('<path_to_dataset>/data/snmptraffic/company_id=1/sensor_id=170',
            columns=['ts', 'snmptraffic_traffic_in_speed'])

## Reading data with R
Here is a minimally working example of how to read Parquet files in R

    library(arrow)
    library(dplyr)

    ds <- open_dataset('<path_to_dataset>/data/snmptraffic')

    df <- ds  %>% collect()

Loading all channels of all sensors can be time consuming. If you already know the company_id
and the sensor_id of interest, you can load just data of one sensor using

    df <- ds  %>% 
      filter(company_id==1, sensor_id==170) %>% 
      collect()

and if you are interested only in specific channels, you can speed up the loading even
further by passing only specific columns:

    df <- ds  %>% 
      filter(company_id==1, sensor_id==170) %>% 
      select(ts, snmptraffic_traffic_in_speed) %>% 
      collect()

