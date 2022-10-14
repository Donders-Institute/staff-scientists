******************************************************************************************
How to align Eyelink eyetracker asc-file with MEG data?
******************************************************************************************

Synchronization between Eyelink data files (at DCCN typically sampled at 1kHz) and MEG data (at DCCN typically sampled at 1.2kHz) can be done by aligning the trigger information from both files, under the assumption that each of the file contains a set of fully overlapping, and truthfully represented triggers. This  `example <https://www.fieldtriptoolbox.org/example/meg_eyelink/>`__ shows how this alignment can be achieved using FieldTrip. 
