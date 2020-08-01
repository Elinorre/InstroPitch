# Technologies Used
## Data Cleaning and Analysis
Sound files will be extracted with a library named librosa. Librosa will be used to standardize every wave file that is used
- sampling rate: 25.05KHz
- normalize bit-depth to +/-1
- flatten audio channels into 1 (mono)
- make files same length (1 second)
- no offset

Pandas will be used to clean the data and perform an exploratory analysis. Further analysis will be completed using Python.

## Database Storage
Audio files are saved to an Amazon Web Service S3 bucket
Postgresql is the database Iintend to use, and Iwill use librosa to display spectrograms.

## Machine Learning
Iwill use a 2D convolutional neural network to process spectrograms. model will use keras Conv2D class.

Iwill use two neural networks, one to identify note and one to identity instrument.
- The note neural network will take in a spectrogram and output 1 of 12 notes.
- The instrument neural network will take in a spectrogram (possibly of longer duration) and output 1 of 14 instruments (possibly 15 outputs, 14 instruments + unknown/other).

## Dashboard
In addition to using a Flask template, Ialso hope to turn script into an executable file that users can drop wav files into and have program determine note and instrument.
- User would input file
- Select start time