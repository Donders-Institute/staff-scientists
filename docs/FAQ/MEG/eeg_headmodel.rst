******************************************************************************************
How to deal with anatomical MRIs that were acquired with an EEG cap on the subject's head?
******************************************************************************************

If you want to create EEG volume conduction models for source reconstruction from individual anatomical images, you typically need to create a set of geometrical models that describe the surfaces that separate compartments with different conductivities (e.g., for BEM models, or for nested spheres models), or models that describe the different tissue compartments (for FEM models). The creation of these models is almost an art in itself, and one-shoe-that-fits-almost-all recipes are documented for `BEM<https://www.fieldtriptoolbox.org/tutorial/headmodel_eeg_bem/>`__ and `FEM<https://www.fieldtriptoolbox.org/tutorial/headmodel_eeg_fem/>`__ volume conduction models. 

What's wrong?
=============

Quite often, the standard geometry processing recipe might give suboptimal results for a given anatomical image. For instance, when the MRI has been acquired while the subject was wearing an EEG cap with electrodes. This requires some creativity - e.g. a change in parameters used for the individual processing steps - and sometimes even manual intervention - e.g. manual cleaning of segmentated volumes using image processing tools - by the researcher. Some tips and tricks are provided in this `FAQ<https://www.fieldtriptoolbox.org/faq/why_does_my_eegheadmodel_look_funny/>`__
