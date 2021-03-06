---
event: '2015 Brainhack Americas (MX)'

title:  'Generating music with resting-state fMRI data'

author:

- initials: CF
  surname: Froehlich
  firstname: C
  email: cfrohlich@nki.rfmh.org
  affiliation: aff1
  corref: aff1
- initials: GD
  surname: Dekel
  firstname: Gil
  email: gil.dekel47@myhunter.cuny.edu
  affiliation: aff3
- initials: DSM
  surname: Margulies
  firstname: Daniel S.
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
  orgname: 'Max Planck Research Group for Neuroanatomy & Connectivity, Max Planck Institute for Human Cognitive and Brain Sciences'
  street: Stephanstraße 1A
  postcode: 04103
  city: Leipzig
  state: Leipzig
  country: Germany


url: https://github.com/carolFrohlich/brain-orchestra

coi: None

acknow: The authors would like to thank the organizers and attendees of Brainhack MX.

contrib: CF wrote the software. GD designed the functions for transforming the data to midi. DSM pick the algorithm that chooses ROIs, and CF and RCC wrote the report.
  
bibliography: ./brainhack-report

gigascience-ref: REFXXX
...

#Introduction
Resting-state fMRI (rsfMRI) data generates time courses with unpredictable hills and valleys, which to some degree resemble the notes of a musical scale. 
Taking advantage of these similarities, and using only rsfMRI data as input, we use basic rules of music theory to transform the data into musical form.
Our project is implemented in Python using the [midiutil library](https://code.google.com/p/midiutil/).

#Approach
\textit{Data}: We used open rsfMRI from the ABIDE dataset \cite{di2014autism} preprocessed by the Preprocessed Connectomes Project \cite{pcp}.
We randomly chose 10 individual datasets preprocessed using C-PAC pipeline \cite{Craddock2013c} with 4 different strategies.
To reduce the data dimensionality, we used the CC200 atlas~\cite{cc200} to downsample voxels to 200 regions-of-interest.

\textit{Processing:} The 200 fMRI time courses were analysed to extract pitch, tempo, and volume--- 3 important attributes for generating music. For pitch, we mapped the time course amplitudes to MIDI values in the range of 36 to 84, corresponding to piano keys within a pentatonic scale. The key of the scale was determined by the global mean ROI value (calculated across all timepoints and ROIs) using the equation: \texttt{(global signal \% 49) + 36}. The lowest tone that can be played in a certain key was calculated from \texttt{(key \% 12) + 36}. The set of tones that could be played were then determined from the lowest tone using a scale. For example, the minor-pentatonic scale's set of tones was calculated by adding \texttt{[0, 3, 5, 7, 10]} to its lowest tone, then skipping to the next octave, and then repeating the process until the value 84 was reached. An fMRI time course was mapped to these possible tones by scaling its amplitude to the range between the smallest and largest tones in the set. If a time point mapped to a tone that was not in the set, it was shifted to the closest allowable tone.
An example of allowed set of tones is shown in Figure \ref{keyfig}.

For tempo, we use the first temporal derivative for calculating the length of notes, assuming we have 4 lengths (whole, half, quarter and eighth note). In the time course, if the modulus distance between time point \textit{t}  and \textit{t + 1} was large, we interpreted it as a fast note (eighth). However, if the distance between \textit{t} and \textit{t + 1} was close to zero, we assumed it is a slow note (whole). Using this approach, we mapped all other notes in between. 

We used a naive approach for calculating volume in a way that tackles a problem we had with fast notes: their sound is cut off due to their short duration. A simple way to solve this is to decrease the volume of fast notes. Thus, the faster the note, the lower the volume. While a whole note has volume 100 ([0,100]), an eighth note has volume 50.

Finally, we selected the brain regions that will play. Users complain when two similar brain regions play together. Apparently, the brain produces the same music twice. However, when the regions are distinct, the music is more pleasant. Thus, we used FastICA \cite{scikitlearn} for choosing brain regions with maximally uncorrelated time courses. 

#Results
A framework for generating music from fMRI data, based on music theory, was developed and implemented as a Python tool yielding several audio files. When listening to the results, we noticed that music differed across individual datasets. However, music generated by the same individual (4 preprocessing strategies) remained similar. Our results sound different from music obtained in a similar study using EEG and fMRI data~\cite{lu2012scale}.


# Conclusions
In this experiment, we established a way of generating music with open fMRI data following some basic music theory principles. This resulted in a somewhat naïve but pleasant musical experience. Our results also demonstrate an interesting possibility for providing feedback from fMRI activity for neurofeedback experiments.


\begin{figure}
  \includegraphics[width=0.45\textwidth]{figure}
  \caption{\label{keyfig}
  (a)Correspondence between the original time series of one ROI and the generated pitch.
  (b)The first 10 notes of the same ROI as sheet music.
  (c)All possible piano keys the brain can play, from 36 to 84 (in pink).
    We show in red all the possible tones for a C Minor-pentatonic scale, in the range of [36, 84].
    In that case, the lowest key is 36.
    The keys that can be used are: [36, 39, 41, 42, 43, 46, 48, 51, 53, 54, 55, 58, 60, 63, 65, 66, 67, 70, 72, 75, 77, 78, 79, 82, 84]
      }
\end{figure}
