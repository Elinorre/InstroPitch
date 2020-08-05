# InstroPitch

> Import necessary libraries using ```pip install -r ".\requirements.txt"```

> To run the application type ```python app.py```

A neural network to predict an instrument and the note.

**Source**: https://zenodo.org/record/3685367#.Xo1NVi2ZOuU (these sounds were originally recorded at Ircam in Paris (France) between 1996 and 1999, as part of a larger project named Studio On Line (SOL))

1. Bass Tuba
2. French Horn
3. Trombone
4. Trumpet in C
5. Accordion
6. Contrabass
7. Violin
8. Viola
9. Violoncello
10. Bassoon
11. Clarinet in B-flat
12. Flute
13. Oboe
14. Alto Saxophone


### Data

All sound files are converted into spectrograms, which are then saved to *notesdf*. In order to alter the pitch data in this dataframe, I used the split function on the pitch column to generate two new columns: notes and octaves, which would subsequently be utilized as inputs in the model.


### Model

Model used in this project is a CNN, which can be easily adapted for other types of sounds

The audio files that are entered will be transformed into spectograms. They are just used as input for the model. I'll train two CNNs using two separate spectrograms. The brief spectrograms from the center of the audio sample will be used to train the CNN that recognizes pitch. The lengthier spectrograms that start at the beginning of the audio file will be used to train the CNN that recognizes instruments.
