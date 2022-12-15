# EarthNet2023

## Deliver

## Hacking

### Deliverable
* The error metric is a  **vegetation score (Veg score)**.
    * The Nash-Sutcliffe Efficiency (NSE) [8] rescales the MSE with the variance of the observations σ_0^2 , it is defined as: NSE = 1 − M SE/σ_0^2. 
It is well suited for time series, especially for ours since variance is an important factor influencing the prediction: during some periods (e.g. winter in Europe) vegetation evolves little or not at all, and is therefore very easy to predict. On the other hand, during other periods (e.g. spring in Europe), vegetation will evolve rapidly and respond in a non-linear way to external factors (luminosity, rainfall). Re-scaling by variance is a way to have a more robust metric and to better compare the samples. The NSE is computed on each pixel of vegetaion, using a mask on the other pixel.
    * The NNSE is the Normalized NSE, NNSE = 1/(2-nse). The NSE is defined on ]-inf, 0], The NNSE normalizes the metric on [0, 1]. 
    * The Veg score is defined as  2 - 1/mean(NNSE), where NNSE is a value per pixel, and Veg Score a global metric to evaluate the model on the validation set. The vegetation score is a metric value which follows the behavior of the NSE (We have NSE = 0 => veg score = 0 and NSE = 1 => veg score = 1), it is thus simpler to interpret than the NNSE, in particular for people used to the NSE, while giving an equivalent weight to each sample thanks to the normalization of the NSE.
* The target metric is 0. This would mean that our model is a better predictor than the average of the observations. This is not so easy to obtain since these observations are predicted, and would beat the state of the art on this task.
* The current achieved value is: 
* Work-breakdown:
    * Data understanding and exploration: 20
    * Framework: 20 hours
    * Optimisation: 2 hours (in progress)

### Documentation
The code is implemented using the deep learning framework PyTorch Lightning [9] which is built on top of PyTorch [10]. The scripts/train.py call first utils/parse.py to transform the configs/en23/.../.yaml file in a dictionnary of hyperparameters (and certify that the hyperparameters are correct).

#### Architecture
* scripts/    
    * train.py (already existing, updated)
    * debug.py (already existing, updated)
    * ...
* configs/
    * en23/
        * convlstm_ae/original/base.yaml (implemented from scratch, updated and documented compared to configs of previous models)
    * ...
* earthnet_models_pytorch/
    * dataloader/
        * en23.py (dataloader for the new dataset, implemented from scratch)
        * ...
    * metric/
        * NNSE_metric.py (new metric, implemented from scratch)
        * ... 
    * model/
        * convlstm_ae.py (new model for the new dataset, implemented from scratch)
    * task/
        * loss.py (refactored and updated, improve to include the new dataset)  
        * spatio_temporal.py (already existing)
        * shedule.py (already exsting, not used)
    * utils/
        * logging.py (already existing) 
        * parse.py (refactored, updated and documented)

### My work


#### Changes from the initate.

#### Difficulties

#### what I learned


### pre-processing
#### Normalisation 
The histogram of each satellite bands depends of the landcover type, latitude and period of the year. So an appropriate normalization affect different part of the histogram. can play an important r
https://medium.com/sentinel-hub/how-to-normalize-satellite-images-for-deep-learning-d5b668c885af


## Initiate

### Context
Forecasting the state of vegetation in response to climate and weather events is a major challenge. Requena and al. (2021) [1] define the land surface forecasting task as a strongly guided video prediction task where the objective is to forecast the vegetation developing at very fine resolution using topography and weather variables to guide the prediction and design a first dataset for this task, the EarthNet2021 dataset. Several papers have addressed this issue, namely [2, 3] on the EarthNet2021 dataset, and [4] focusing instead on Africa (freshly accepted!:D). See [4] for more details on the project.

### EarthNet2022, a new brand dataset
EarthNet2022 is a new dataset, with a better cloud masking and new variables (Landsat climatology NDVI and Sentinel - 1 vegetation index), relevant for environnemental science. Additionnally, the data (from different sources) are irregularly sampled time series. This is the main different from the previous dataset, where the data where 10 dayly interpolated, which is very easy to train deep learning models but hides the actual dynamics of the system.

The variables are:
* Sentinel 2 (B02 to B12 bands) - spatio-temporal data
* Sentinel1 (VV, VH, mask) - spatio-temporal data
* NDVI Climatology (12 months Landsat 30m/pix) - spatio-temporal data
* SRTM (DEM) - static data
* ESA World Cover - static data
* ERA5 (t2m, pev, slhf, ssr, sp, sshf, e, tp) - temporal data
* Soil Grids - static data
* Geomorphons - static data
* ALOS, COP 30 (DEMs) - static data

The target is the Normalized Difference Vegetation Index (NDVI), a proxy for vegetation health monitoring calculated from the red and infrared bands.

*Please note:* This new dataset will most likely **not be publicly available** by the end of the project. However, a sample (or few if needed) will be available, as well as the jupyters used to explore the data. 

### Approach - Irregularly sampled time series
**Bring your own method** 

The objective to create a model capable of learning the relationship between vegetation states, local factors and weather conditions, at a very fine resolution in order to characterise and forecast the impact of weather extremes from an ecosystem perspective. I will firstly explore the new dataset (understand the variables, maybe select some of them), and write the dataloader. 

the data have different dimensions (spatio-temporal, only temporal (for weather), or static (terrain topology), we will use the simplest solution which consists in resizing and concatenating everything to the spatio-temporal dimensions.

Then I need to develop a model for the data. The main point is that the data are irregularly sampled time series (from different sources). Although several papers have addressed this issue for RNNs [5-7], we need to extend the methods to include the spatial dimension of our data. this will be our main axis of research. We will compare our results to a ConvLSTM baseline that does not take into account the time-step irregularity. 

A second solution, simpler to implement, consists in directly use the RNN methods developed for time series without spatial component. During the learning phase, the RNN is used only on a random subset of pixels, and the gradient is computed on these pixels. The idea is that in an image, and even more in a landscape, a large number of pixels are (almost) identical, and it is not necessary to compute the gradient for each pixels. In this way, we drastically reduce the learning time required for the use of a RNN on spatio-temporal data. This solution has shown good results with classical RNN, so it is promising for RNN adapted to irregular data. 

Finally, I would like to develop a web application, the tricky part is that I have no knowledge of javascript at the moment, so it could be reduced to a docker, or a simple command line in case of lack of time. But ideally, on the web application, the user will be able to select a sample on the map of Africa, then generate its prediction with some associated analysis. 

### Work-breakdown structure 
 * Data collection: 0 hours
 * Data understanding and exploration: 10 hours   
understanding the distribution 
 * Framework: 10 hours   
PyTorch lighnight framework (already implemented?)
 * Model for irregularly sampled time series: 50 hours
 * Optimisation: 20 hours 
 * Building an application: 40 hours   
 * Final report: 15 hours
 * Presentation: 10 hours
 * Total: 155 hours


### Bibliography
[1] Christian Requena-Mesa, Vitus Benson, Markus Reichstein, Jakob Runge, and Joachim Denzler. "Earthnet2021: A large-scale dataset and challenge for earth surface forecasting as a guided video prediction task." In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 1132–1142, 2021.

[2] Codrut-Andrei Diaconu, Sudipan Saha, Stephan Günnemann, and Xiao Xiang Zhu. "Understanding the role of weather data for earth surface forecasting using a convlstm-based model." In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 1362–1371, 2022.

[3] Klaus-Rudolf William Kladny, Marco Milanta, Oto Mraz, Koen Hufkens, and Benjamin David Stocker. "Deep learning for satellite image forecasting of vegetation greenness." bioRxiv, 2022.

[4] Claire Robin, Christian Requena-Mesa, Vitus Benson, Lazaro Alonso, Jeran Poehls, Nuno Carvalhais, and Markus Reichstein. "Learning to forecast vegetation greenness at fine resolution over Africa with ConvLSTMs". In *Tackling Climate Change with Machine Learning workshop, NeurIPS 2022*.  	https://arxiv.org/pdf/2210.13648.pdf

[5] Shukla, Satya Narayan, and Benjamin M. Marlin. "A survey on principles, models and methods for learning from irregularly sampled time series." arXiv preprint arXiv:2012.00168 (2020).

[6] Schirmer, Mona, et al. "Modeling irregular time series with continuous recurrent units." In *International Conference on Machine Learning. PMLR, 2022*.

[7] Rubanova, Yulia, Ricky TQ Chen, and David K. Duvenaud. "Latent ordinary differential equations for irregularly-sampled time series." Advances in neural information processing systems 32 (2019).

[8] J Eamonn Nash and Jonh V Sutcliffe. River flow forecasting through conceptual models part i—a discussion of principles. Journal of hydrology, 10(3):282–290, 1970.

[9] Falcon, William. "Pytorch lightning." GitHub. Note: https://github. com/PyTorchLightning/pytorch-lightning 3.6 (2019).

[10] Paszke, A., Gross, S., Massa, F., Lerer, A., Bradbury, J., Chanan, G., Killeen, T., Lin, Z., Gimelshein, N., Antiga, L., Desmaison,
A., Kopf, A., Yang, E., DeVito, Z., Raison, M., Tejani, A., Chilamkurthy, S., Steiner, B., Fang, L., Bai, J., and Chintala, S.
(2019). Pytorch: An imperative style, high-performance deep learning library. In Wallach, H., Larochelle, H., Beygelzimer, A.,
d'Alché-Buc, F., Fox, E., and Garnett, R., editors, Advances in Neural Information Processing Systems 32, pages 8024–8035.
Curran Associates, Inc.
