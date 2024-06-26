## A Phenology-Based Method of Locating Nonnative Tree Species in NYC
 
 I explored using phenology metrics derived from satellite data to parse native and nonnative tree species in New York City. In the temperate climate of the northeastern US, most of our tree species are deciduous. Nonnative species tend to follow a phenological cycle that is distinct from native species, leafing out earlier in the spring and losing their leaves later in the fall. These phenological signals could enable a remote sensing method to supplement field surveys for monitoring the presence of invasive species. 

### Data

* [Natural Areas Conservancy](https://naturalareasnyc.org/) tree species data: Collected through field survey methods, this dataset identifies the dominant tree species (based on proportion of basal area in each plot) in 1,052 plots in forested areas throughout New York City. 
* Of these plots, 927 are dominated by a native species and 125 by a nonnative species (see charts below).
  * The three most common native species are Oak, Sweetgum, and Black Cherry. The three most common nonnative species are Black Locust, Mulberry, and Tree of Heaven.
* Satellite data: The 30m resolution [Harmonzed Landsat/Sentinel-2 data product](https://lpdaac.usgs.gov/data/get-started-data/collection-overview/missions/harmonized-landsat-sentinel-2-hls-overview/) was accessed using NASA's Earth Data API. For code used to access and process satellite data, see [this notebook](./R/phenology_metrics.Rmd)
* Vector data: shapefiles of New York City's parks and boroughs were used to crop the raster data.

Sample plots in Pelham Bay Park, one of the forested areas represented in the dataset             |  
:-------------------------:|
![](./images/plots.png)

Number of Plots for Each Native Species             |  
:-------------------------:|
![](./images/native_species2.png)  


Number of Plots for Each Nonnative Species          |
:-------------------------:|
![](./images/nonnative_species.png)

### Methods

Sentinel-2 imagery is captured every five days, but in practice, usable imagery is available less frequently due to cloud cover. In order to create rich time series of growing season TDVI, I extracted TDVI time series for three consecutive years and then merged them into a single time series. I then smoothed the time series for each pixel using a cubic spline function to produce daily time series of TDVI like the ones shown below.

Examples of the smoothed, interpolated TDVI time series for two pixels containing predominantly Oak or Black Locust trees. The colored dots represent the start and end of the growing season.          |
:-------------------------:|
![](./images/pheno_plot.png)


I [calculated](./R/phenology_metrics.Rmd) 22 different phenology metrics, such as Length of Growing Season, rate of spring greenup and fall senescence, peak TDVI and point of peak TDVI. The plots below visualize some of these metrics. 

Three definitions of the growing season          | 
:-------------------------:|
![](./images/los_plot.png)

Periods of spring Greenup and Fall Senescence          |
:-------------------------:|
![](./images/greenup_plot.png)


These metrics, along with spectral band information from spring and fall, were used as input to a series of random forest models. I first trained the models to classify forested pixels as either native or nonnative, using individual native species as the native category. Next, I trained a model that used all 43 different native species in the dataset as the native category. My code for this process can be found [here](./R/train_random_forest_models.Rmd)

To train each model, data were split into training (70%) and testing (30%) subsets. Each model was trained and tested using 10-fold cross-validation. Resampling was conducted within cross-validation using either up-sampling or down-sampling to mitigate class imbalance in the training data. 

Each model was trained and tested five times and the mean accuracy metrics were recorded. Precision, recall, and F1 were the metrics I used to assess the models' performance. 

Finally, I [used the all-species model](./R/classify_nonnative_species_nyc.Rmd) to produce maps predicting the locations of nonnative species throughout the city's forested areas. 

### Results 

*Single Species Models*

The highest performing individual species models were the Sweetgum and Red Maple models, which achieved high F1 scores for both native and nonnative categories.

The severe class imbalance in the Black Cherry and Hickory models may have impeded training. However, the Red Maple model and Tulip Tree model show that, even with highly imbalanced data, phenological information can successfully differentiate these species. 



F1 for individual species: Back Cherry, Hickory, Oak, Red Maple, Sweetgum and Tulip Tree          |
:-------------------------:|
![](./images/species_accuracy_plot.png)

Number of plots available for each species          |
:-------------------------:|
![](./images/species_population_plot.png)


*All-species Model*

The all-species model had highly imbalanced training data (927 native vs. 125 nonnative samples). Using resampling techniques, the model can be optimized to maximize either precision or recall, depending on the priorities of the end user.

Recall shows the percentage of pixels in each category correctly identified by the model. The down-sampled model performs better in this task, correctly identifying about 75% of total pixels in each category. The up-sampled model was able to identify less than 25% of nonnative pixels.

Recall for all-species model using down-sampling (left) or up-sampling (right)      |
:-------------------------:|
![](./images/recall_allspecies_plot.png)



Precision shows how many predictions in each category were correct.  In this case, the up-sampled model performs better, with 60% of nonnative predictions and 90% of native predictions being correct. 

Precision for all-species model using down-sampling (left) or up-sampling (right)      |
:-------------------------:|
![](./images/precision_allspecies_plot.png)

*Maps*

This map of Manhattan was produced using the up-sampled all-species model predictions for all pixels with greater than 60% tree canopy. This model is known to predict nonnative pixels with ~60% accuracy, but to identify ~25% of total nonnative pixels present. This map therefore presents a conservative estimate of the presence of nonnatives, but we can be fairly confident that those identified are accurate.

![](./images/mn_map.png)

These inset maps of Prospect Park in Brooklyn and Van Cortland Park in the Bronx show the probability that each pixel belongs to the nonnative class. For reference, the binary native/nonnative maps of the borough is included on the left. Both maps are produced using the up-sampled model.

<b>Prospect Park</b>

![](./images/prospect_park.png)

<b>Van Cortland Park</b>

![](./images/vancortland_park.png)

### Conclusion

While phenological information can be sufficient to parse native from nonnative species, the severely imbalanced nature of the available data impedes the training process. Acquiring a higher number of nonnative samples would allow for a fuller assessment of this method’s potential.  However, the success of the high performing models shows that a combination of clear phenological separation between classes, adequate sample size, and resampling strategy can produce useful guidance for forest managers seeking to estimate the presence of invasive species. 
