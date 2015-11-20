---
event: '2015 Brainhack Americas (MX)'

title:  'Generating music with resting state fMRI data'

author:

- initials: CF
  surname: Froehlich
  firstname: C
  email: cfrohlich@nki.rfmh.org
  affiliation: aff1, aff2
- initials: GD
  surname: Dekel
  firstname: Gil
  email: gil.dekel85@gmail.com
  affiliation: aff3
- initials: DM
  surname: Margulies
  firstname: Daniel
  email: margulies@cbs.mpg.de
  affiliation: aff4
- initials: RCC
  surname: Craddock
  firstname: R. Cameron
  email: ccraddock@nki.rfmh.org
  affiliation: aff1, aff2


affiliations: 

- id: aff1
  orgname: 'Computational Neuroimaging Lab, Center for Biomedical Imaging and Neuromodulation, Nathan Kline Institute for Psychiatric Research'
  street: 140 Old Orangeburg Rd
  postcode: 10962
  city: Orangeburg
  state: New York
  country: USA
- id: aff2
  orgname: 'Center for the Developing Brain, Child Mind Institute'
  street: 445 Park Ave
  postcode: 10022
  city: New York
  state: New York
  country: USA
- id: aff3
  orgname: 'City University of New York-Hunter College'
  street: 695 Park Ave
  postcode: 10065
  city: New York
  state: New York
  country: USA
- id: aff4
  orgname: 'Max Planck Institute for Human Cognitive and Brain Sciences'
  street: Deutscher Platz 6
  postcode: 04103
  city: Leipzig
  state: Leipzig
  country: Germany


url: https://github.com/carolFrohlich/brain-orchestra

coi: None

acknow: The authors would like to thank the organizers and attendees of Brainhack MX.

contrib: RCC and DJC wrote the software, DJC performed tests, and DJC and RCC wrote the report.
  
bibliography: brainhack-report

gigascience-ref: REFXXX
...

#Introduction
Resting-state fMRI (rsfMRI) data generates interesting time courses with unpredictable hills and valleys. This data in some degree resembles the notes of a music scale. 
Taking advantage of those similarities, we apply the basic rules of music theory for preprocessing rsfMRI and generate pleasant music for humans, using only the rsfMRI data as input.
Our project used only Python code and the midiutil music library \textit{(https://code.google.com/p/midiutil/)}.

#Approach
\textit{Data}: We used resting-state fMRI data from the ABIDE dataset \cite{di2014autism} that was openly available and preprocessed by the Preprocessed Connectomes Project \cite{pcp}.
We randomly chose 10 subjects that were preprocessed using C-PAC pipeline \cite{Craddock2013c}.
Because there is no golden standard for preprocessing fMRI and C-PAC has 4 preprocessing strategies available, we used data preprocessed with all strategies for comparison purposes.
For reducing the data dimensionality, we chose the CC200 atlas \cite{cc200}, which divides the brain into 200 functional regions.

\textit{Processing}
We extracted from the data 3 important attributes for generating music: pitch, tempo and volume.
For the pitches, we pretend using a piano with keys from 21 to 108. Because we know the most pleasant keys are in the range 36 to 84, we only use those keys.
But instead of playing all keys, we choose only the ones that are in a certain scale.
This is done first by calculating the tone of the brain, which is always a number between 36 and 84:
$(global signal \% 49) + 36$. 
Second, we calculate the smallest key we can play with the tone: $(tone \% 12) + 36$
Third, we calculate all keys we can play from the smallest key using a scale.
Finally, we scale each time course from the smallest to the biggest key we can play on the piano.
Some keys will not be in the allowed key set. Then, we shift them to the closest allowed key.
An example of allowed set of keys is shown in Figure \ref{figure}.


The simplest method for creating the tempo is to play each note for the same time as the TR. In our case, 2 seconds. 
In spite of this method being true to the data, it generates boring music.
For addressing this problem, we use a temporal derivative for calculating the tempo, assuming we have 4 tempos (whole, half, quarter and eighth note).
In the time course, if the modulus distance between time point \textit{t}  and \textit{t + 1}  is big, we interpret it as a fast note, or an eighth note. 
However, if the distance between \textit{t} and \textit{t + 1} is close to zero, we assume it is a slow note, or a whole note.
Using this approach, we map all the other notes in between. 


The third music component is the volume.
We use a naive approach for calculating it, in a way that tackles some problems we have with fast notes:
they sound cut off because they play too fast.
An easy way for solving that is decreasing the volume for fast notes.
Thus, the faster the note, the lower the note’s volume is. While a whole note has volume 100 (from 0 to 100), an eighth note has volume 50.

Finally, we choose the brain regions that will play.
Because we do not want to play all the 200 regions at the same time, we use FastICA \cite{scikitlearn} for choosing some distinct brain regions. 
Users complain when two very similar brain regions play together, it seems like the brain is playing the same music twice. But when the regions are distinct, the music is pleasant.


#Results
The music we obtained with this method was self-reported as pleasant. It had most of the characteristics that are considered good by music theory, specifically a nice rhythm.
The implementation gave us some flexibility for experimenting with different instruments, scales, and tempos.
When listening to the results, we noticed that the music across subjects are different, but the music generated by the same subject using the 4 different strategies sounded very similar. 
These results are distinct than the ones obtained in a similar study using EEG and fMRI data \cite{lu2012scale}.


Some of the drawbacks are that because we just followed some basic music theory formulas, it sounds naive for people with some training, and because we used the default instruments from the midi library, we ended up with very different results depending on the instruments we chose to play. 


# Conclusions
In this experiment, we established a way of generating music with open fMRI data following some basic music theory principles.
This resulted in a somewhat naive music that sounded pleasant for humans, which is very different from the result in a similar experiment \cite{lu2012scale}.



\begin{figure}
  \includegraphics[width=0.5\textwidth]{figure}
  \caption{
  (a)Correspondence between the original time series of one ROI and the generated pitch.
  (b)The first 10 notes of the same ROI in a sheet music.
  (c)All possible piano keys the brain can play, from 36 to 84 (in pink).
    We show in red an example of allowed keys when the tone is 36 and we use the minor pentatonic scale.
    In that case, the smallest key is 36.
    The keys that can be pressed from 36 are: [36, 39, 41, 42, 43, 46, 48, 51, 53, 54, 55, 58, 60, 63, 65, 66, 67, 70, 72, 75, 77, 78, 79, 82, 84]
      }
\end{figure}