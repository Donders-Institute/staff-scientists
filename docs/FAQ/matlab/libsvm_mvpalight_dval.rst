***************************************************************************************
Is there a sign problem with the decision values when using libsvm's SVM in MVPA-Light?
***************************************************************************************

What's wrong?
=============

When using MVPA-Light (e.g. mv_classify), in combination with an SVM-classifier as implemented in the libsvm  library, the decision values may get flipped, causing cross-validated accuracies (and other performance metrics)to be miscalculated (if they are calculated from the 'dval', rather than from the decoded classes directly).

Early 2022 the MEG seems to be affected (occasionally) by a high-frequency artifact that seems to be caused by RF-noise entering the MSR, which interacts with the MEG electronics to generate a high frequency artifact in the signals that drifts over time (in terms of peak frequency). The frequency of the artifact is usually well beyond the physiological range, so can be dealt with by lowpass filtering. However, sometimes the peak frequency drifts into the range of phsysiological activity, which makes lowpass filtering useless. 

How to check whether my analyses are affected?
==============================================

The following snippet of MATLAB-code can be used to evaluate whether your setup has this bug.

.. code-block:: matlab

  %%
  
  % set up the environment, only needed if you want to test from scratch,
  % otherwise skip this section
  tdir = tempdir;
  cd(tdir);
  system('git -c http.sslVerify=false clone https://github.com/treder/MVPA-Light'); % the -c is needed to allow the system call to work on our compute cluster
  system('git -c http.sslVerify=false clone https://github.com/cjlin1/libsvm');
  
  addpath(fullfile(tdir, 'libsvm', 'matlab'));
  make;
  
  addpath(fullfile(tdir, 'MVPA-Light', 'startup'));
  startup_MVPA_Light;
  
  %%
  
  % downstream, it is assumed that MVPA-Light and libsvm are on your
  % MATLAB-path, to test your current installation, don't execute the above
  % section
  
  % create some data
  design = [ones(1,100) ones(1,100)+1];
  dat    = randn(size(design)) + (design-1);
  
  cfg = [];
  cfg.classifier = 'libsvm';
  cfg.repeat     = 1;
  cfg.metric     = 'accuracy';
  cfg.preprocess = {'zscore'};
  cfg.feature_dimension = [];
  cfg.k = 5;
  
  rng(42);
  res_svm1 = mv_classify(cfg, dat(:), design(:));
  
  % does another classifier also work?
  cfg.classifier = 'lda';
  
  rng(42);
  res_lda1 = mv_classify(cfg, dat(:), design(:));
  
  % and what if you also want to know about the AUC?
  cfg.metric = {'accuracy' 'auc'}; % specifying auc as well, results in the lower-level code to use the dval for the accuracy estimation, rather than clabel
  
  rng(42);
  res_lda2 = mv_classify(cfg, dat(:), design(:));
  
  cfg.classifier = 'libsvm';
  res_svm2 = mv_classify(cfg, dat(:), design(:));
  
  % evaluate the performance metrics, if res_svm2 has values < 0.5, , then
  % the dval got flipped along the way, leading to a mismatch between the
  % class labels and the assumed side of the data points relative to the
  % decision hyperplane.

Do I need to do something about it?
===================================

If your installation of libsvm+MVPA leads to accuracies (and AUCs) < 0.5 then you need to comment out a line of code in the test_libsvm.m file, which is in <your-distro-of-MVPA-Light>/models. Specifically, the line where the 'dval' is negated (dval = -dval) should be commented out. Also, don't forget to check out the associated link to the libsvm docs, which is probably related to the reason why the sign-flip was implemented in the first place. However, said documentation is a bit ambiguous, and there's no explicit record of whether - with a well-defined combination of versions of the code - the flip does not need to be executed anymore (and that in other words it is a general bug in the current version of MVPA-Light, and thus that it should be fixed. We have reached out to the MVPA-Light developers. Stay tuned.
