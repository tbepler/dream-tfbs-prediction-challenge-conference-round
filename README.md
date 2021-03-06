# ENCODE-DREAM in vivo transcription factor binding prediction challenge
### Conference round submission
##### Tristan Bepler

#### Python setup
Set up the python environment. This will probably error near the end, but that seems to be ok.
```
conda env create -f environment.yml -n tfbs-prediction
source activate tfbs-prediction
```
Install theano and other python packages not in anaconda. The predictions were generated using theano version 0.9c-dev.

pyBigWig requires libcurl so make sure that's installed first
```
pip install --no-deps git+git://github.com/Theano/Theano.git
pip install pyBigWig synapseclient
```

recommended: add floatX=float32 to your .theanorc file

#### Data setup and preprocessing
Move the raw data into data/raw/\* or download the raw data from synapse using
```
scripts/setup/download_raw
```
Download some auxillary data
```
scripts/setup/download_chrom_sizes
scripts/setup/download_gene_annotations
scripts/setup/download_tf_and_cofactor_list
```

Process the data into a form usable for training models. This may take a while, requires make.
```
scripts/pipeline preprocess
```

#### Train models and generate predictions
Predictions were generated using the model in models/resid\_conv\_crf\_u64\_l3\_6\_6\_3\_max\_pool\_no\_rna\_nbins64\_stride32\_minibatch64.py

This model uses a deep residual convolutional neural network with a conditional random field decoder to produce a predicted probability of each 200bp region being bound. It uses max pooling to downsample the sequences by 2 after 3 layers, 5 after 6 more layers, and another 5 after 6 more layers. The final convolutional outputs then go into a fully connected layer followed by the linear chain CRF. The model is trained on fragmented input regions using RMSprop with Nesterov momentum. Inputs are one-hot encoded genomic sequence concatenated to the DNAse fold coverage signal.

To train the model and generate predictions for all TFs run
```
scripts/fit_production_model -m models/resid_conv_crf_u64_l3_6_6_3_max_pool_no_rna_nbins64_stride32_minibatch64.py
```

Predictions and trained models should now be in the predicts/ directory

