---
layout: post
title: Time series prediction with LSTM neural nets
subtitle: Wrapping the Keras model
image: /img/old_clock_thumb.jpg
gh-repo: eskilan/LSTMwrapper
gh-badge: [star, fork, follow]
---

In this post I'd like to talk about my recent experience with a type of recurrent neural network called "Long-short term memory" or LSTM neural net. It is used to predict a sequence or the next step of a sequence, typically for time-series prediction problems. We could use LSTMs, for example, to predict the number of customers that will come into a store at a given time or to compute the future price of a commodity given historical data. LSTMs have also been used for text-to-speech synthesis, for translation, and for handwriting recognition.

You can find more information about LSTMs in this amazing blog post: http://colah.github.io/posts/2015-08-Understanding-LSTMs/
More detailed information can be found here as well: http://www.cs.toronto.edu/~graves/phd.pdf

In this post I'd like to share my experience creating a Python class that wraps an LSTM model made with Keras (with a tensorflow backend). Afterwards, I use it to make polluting predictions. Ok, here we go:

First, we load our libraries:
````python
# numpy, pandas, plot
import numpy as np
np.random.seed(1337) # for reproducibility
import pandas as pd
import matplotlib.pyplot as plt
# keras
from keras.models import Sequential
from keras.layers import Dense
from keras.layers import LSTM
# sklearn scaler
from sklearn.preprocessing import MinMaxScaler
````
We define the model:
````python
class Model():
    def __init__(self, csvName='', dataFrame=[], nIn=1):
        self.csvName = csvName
        if csvName != '':
            self.loadDataFromFile()
        else:
            self.dataset = dataFrame
        
        # convert dataframe to float numpy array, and removing index
        self.data = self.dataset.values.astype('float32')
        
        self.scalerX = MinMaxScaler()
        self.scalerY = MinMaxScaler()
        self.nIn = nIn
````

Then I wrote the following accesory member functions to load the data, scale the independent X data, and the dependent Y data, and to setup the outputs.
````python
    def loadData(self):
        self.dataset = pd.read_csv(self.csvName, header=0, index_col=0)
        print('Data Loaded:')
        print(self.dataset.head())
    
    # scaling functions create scaler object and return data in 0 to 1 range
    def scaleXData(self, unscaledX):
        return self.scalerX.fit_transform(unscaledX)
        
    def scaleYData(self, unscaledY):
        return self.scalerY.fit_transform(unscaledY)
        
    def setupOutputs(self,outputVarNamesList):
        self.outIndices = []
        for outputName in outputVarNamesList:
            for columnInd, columnName in enumerate(self.dataset.columns):
                if outputName == columnName:
                    self.outIndices.append(columnInd)
````

The next function takes the internally stored pandas DataFrame values stored in self.data and creates two 3D matrices called self.X3D and self.Y3D that are shaped as (numSamples, numTimeSteps, numFeatures).

````Python
# Scales and shapes the data into 3D matrices               
    def rearrangeData(self):
        nIn = self.nIn
        data = self.data
        
        arrayLength = data.shape[0]
        self.nSamples = arrayLength - nIn
        self.nFeatures = data.shape[1] 
        # initialize some 3d matrices. X for inputs, Y for outputs
        self.X3d = np.zeros((self.nSamples,nIn,self.nFeatures))
        self.Y3d = np.zeros((self.nSamples,1,len(self.outIndices)))
        
        # scale data
        xData = data
        yData = data[:,self.outIndices]
        
        scaledX = self.scaleXData(xData)
        scaledY = self.scaleYData(yData)
        
        # fill up 3d matrix with samples for autoregression
        for ind in range(0,self.nSamples):
            self.X3d[ind,:,:] = scaledX[ind:ind+nIn,:]
            self.Y3d[ind,0,:] = scaledY[ind+nIn,:]
        
        # set data shape
        self.inShape = (nIn,self.nFeatures)
````
Once the sequential data has been converted into 3D matrices, we split the data into training and validation, and the output Y is reshaped (because Keras demands it when "output_sequences is set to False):
````Python
# splits the 3D matrices into training and validation
# also reshapes the output as a 2d matrix since only outputing at time t
	def splitData(self,lastTimeFrame):
		self.X3dTrain = self.X3d[0:lastTimeFrame,:,:]
		self.X3dVal = self.X3d[lastTimeFrame:,:,:]
		self.Y3dTrain = self.Y3d[0:lastTimeFrame,:,:]
		self.Y3dVal = self.Y3d[lastTimeFrame:,:,:]
		
		# number of samples for training and validation
		nTrain = self.X3dTrain.shape[0]
		nVal = self.X3dVal.shape[0]
		
		# reshaping target to avoid mismatch at dense layer
		self.Y3dTrain = np.reshape(self.Y3dTrain,(nTrain,len(self.outIndices)))
		self.Y3dVal = np.reshape(self.Y3dVal,(nVal,len(self.outIndices)))
````
Finally, we can train the model by calling trainModel(). 
````Python
    def trainModel(self,numUnits,lossFunc,optimizerType,nEpochs,batchSize):
        self.model = Sequential()
        self.model.add(LSTM(numUnits, input_shape=self.inShape))
        self.model.add(Dense(units=1*len(self.outIndices)))
        self.model.compile(loss=lossFunc, optimizer=optimizerType)
        # run
        self.history = self.model.fit(self.X3dTrain, self.Y3dTrain, epochs=nEpochs, batch_size=batchSize, validation_data=(self.X3dVal, self.Y3dVal), verbose=2, shuffle=False)
````
The rest of the code can be found in my Github repository (https://github.com/eskilan/LSTMwrapper). Now, I'd like to give it a try. I got a set of pollution data gathered at the US embassy in Beijing. The dataset includes pollution levels and atmospheric data recorded each hour for about five years since 2010-01-02 00:00:00 to 2014-12-31 23:00:00, and includes pollution, dew point, temperature, atmospheric pressure, wind direction, wind speed, snow, and rain levels. After removing snow, and wind direction from the inputs (for simplicity) I ran the data through my wrapper object.

````Python
# create LSTM wrapping model
myNN = Model(dataFrame=pollutionReduced, nIn=12)
#outputVarNamesList = ['pollution','dew']
outputVarNamesList = pollutionReduced.columns.get_values()
# set outputs
myNN.setupOutputs(outputVarNamesList)
# create 3d dataset
myNN.rearrangeData()
# create split data sets
lastTimeFrame= 3*365*24
myNN.splitData(lastTimeFrame)
# set up hyperparammeters
numUnits = 20
lossFunc = 'mse'
optimizerType = 'adam'
nEpochs = 25
batchSize = 70
# train the LSTM NN
myNN.trainModel(numUnits,lossFunc,optimizerType,nEpochs,batchSize)
#myNN.plotHistory()

# number of steps to predict
numSteps = 12
#Y = myNN.predictNextTimeFrame(lastTimeFrame)
Yseq = myNN.predictSequence(lastTimeFrame,numSteps)
Yseq2 = myNN.predictMultipleNextSteps(lastTimeFrame,numSteps)
# comparing
plotSequences(pollutionReduced, lastTimeFrame, Yseq)
plotSequences(pollutionReduced, lastTimeFrame, Yseq2)
````
The first plot is the following, and it's what is output after training the model for 25 epochs. The model is set to predict the next 12 hours of ALL predictors. It does it by predicting the pollution, dew point, temperature, etc. for time t. Then it uses the predictions from time t, to predict time t+1. The result is that while the prediction for the first hour are OK, the error builds up thoughout time.

![alt text](/img/LSTM_fig1.png "Predicting everything for 12 hours. Not great.")

The next figure was generated by predicting the next hour, then assuming we have new sensor information for all predictors, and again predicting the next hour. The correction from having the proper information becomes apparent.

![alt text](/img/LSTM_fig2.png "Predicting every hour. Not bad.")

Next, I decided to only fit pollution as a function of all predictors, and build each future prediction on current sensor information for all predictors except for pollution. Think of it as predicting the future after weather conditions are known but the pollution sensor is broken.

![alt text](/img/LSTM_fig3.png "Predicting the future with known weather, unkown pollution. Not great.")

I trained the model with three years worth of data, and the results are actually quite dissapointing. The good thing is I have a simple object to organize and reshape the data in case I want to try LSTMs for a different and perhaps more interesting application.
