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
Our project used only Python code and the midiutil music library (https://code.google.com/p/midiutil/).

#Approach
\texttt{Data} We used resting-state fMRI data from the ABIDE dataset \cite{di2014autism} that was openly available and already preprocessed by the Preprocessed Connectomes Project \cite{pcp}.
From there, we randomly chose 10 subjects that were preprocessed using C-PAC pipeline \cite{Craddock2013c}.
Because there is no golden standard for preprocessing fMRI (especially for generating music) and C-PAC has 4 preprocessing strategies available, we used data preprocessed with all strategies for comparison purposes, yielding 40 files used in this experiment. 
For reducing the data dimensionality, we chose the CC200 atlas \cite{cc200} , which divides the brain into 200 functional regions.
Thus, for each file, we had a matrix of the size TR x 200 regions. 

\texttt{Processing}
After choosing the data, we processed it in order to extract from the brain 3 important qualities for generating music: pitch, tempo and volume.
For generating the pitches, we pretend we are using a piano that has keys from 1 to 128. Because we know the most pleasant keys are in the range 36 to 84, we only allow the brain to play those keys.
But instead of playing all keys, we make the brain choose only the ones that are in a certain scale.
This is done first by csudo apt-get install texlivehoosing the tone of the music, which is calculated by:
(global_signal % 49) + 36
That always gives us a number between 36 and 84.
Second, we calculate the smallest key we can play with the tone, which is calculated by:
(tone % 12) + 36
Third, we calculate all keys we can play from the smallest key using a scale. In this experiment, we use the pentatonic scale.

For example, if the global signal is 72, the tone is 36, the smallest key is 36, and the keys the brain can play are [todo].
Finally, we scale each time course in the matrix from the smallest to the biggest key we can play on the piano.
Even though all keys will be within the range, some won’t be in the allowed keys set. In this case, we shift them to the closest allowed key, always going down if there is a tie.
In this sense, we build up the pitches in a way most notes sound good together and also maintain the shape of the functional data.  


When calculating the tempo, the simplest method we can do is to play each note for the same time as the TR. In our case, each note would play for 2 seconds. 
In spite of this method being true to the data, it generates predictable and boring music.
For addressing this problem, we use a temporal derivative for calculating the tempo.
We assume we can play the music in 4 tempos:
Whole note, half note, quarter note and eighth note.
In the time course, if the modulus distance between one time point t and t+1 is big (if the data jumped), we interpret it as a fast note, or an eighth note. 
However, if the distance between a time point t and t + 1 is close to zero, we assume it is a slow note, or a whole note.
Using this approach, we map all the other notes in between. 
In this way, we have some variability in the tempo, making the music less boring, but still being true to the data. 
#maybe I have to explain more about it


The third music component we need to generate is the volume.
We use a naive approach for calculating it that actually tackles some problems we have with fast notes.
When playing a fast note (in particular eighth notes) the note sounds cut off because it has played too fast. 
An easy way for solving that is decreasing the volume for fast notes.
Thus, the faster the note, the lower the note’s volume is. While a whole note has volume 100, an eighth note has volume 50.
In conclusion, this easy approach gives some variability to the music and also makes all the notes sounds good.

Finally, we choose the brain regions that will play.
Because we do not want to play all the 200 brain regions at the same time, we use FastICA \cite{scikitlearn} for choosing 3 distinct brain regions. 
Users complain that when two very similar brain regions play together, it seems like the brain is playing the same music twice. But when the regions are distinct, the music is pleasant.



#Results
The music we obtained with this method was self-reported as pleasant for humans. It had most of the characteristics that are considered good by music theory, specifically a nice rhythm.
The implementation gave us some flexibility for experimenting with different instruments, scales, and tempos.
When listening to the results, we noticed that the music across subjects are different, but the music generated by the same subject using the 4 different strategies sounded very similar. 
These results are different than the ones obtained in a similar study using EEG and fMRI data \cite{lu2012scale}.


Some of the drawbacks are that because we just followed some basic music theory formulas, it sounds naive for people with some training, and because we used the default instruments from the midi library, we ended up with very different results depending on the instruments we chose to play. 


# Conclusions
In this experiment, we established a way of generating music with open fMRI data following some basic music theory principles.
This resulted in a somewhat naive music that sounded pleasant for humans, which is very different from the result in a similar experiment \cite{lu2012scale}.



\begin{figure}
  \includegraphics[width=0.5\textwidth]{figure}
  \caption{Lorem ipsum}
\end{figure}