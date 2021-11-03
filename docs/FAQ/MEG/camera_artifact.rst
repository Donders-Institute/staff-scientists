**********************************************************
What to do with the 20 Hz + harmonics artifact in the MEG?
**********************************************************

Since its introduction in July, there is an artifact in the MEG signals, caused by the light camera in the MSR. Data that has been collected between early July 2018 and <some-date-in-the-future-once-it-is resolved> is most likely affected.

What's wrong?
=============

The Raspberry pi operated camera - when switched on - produces an artifact at 20 Hz and its harmonics, which produces a series of artifact lines in the powerspectra of the MEG signals. These lines are sharp when there is no (or modest) fluctuations in light picked up by the camera. However, the spectral lines can be appreciably broadened when the projector is projecting stuff on the screen. This suggests that there is some amplitude modulation of the artifact.

How to check how bad it is in my data?
======================================

The easiest way to check whether you are affected is to compute a powerspectrum of the MEG signals with a sufficiently high spectral resolution.

.. code-block:: matlab

  cfg = [];
  cfg.dataset = fullfile(<path-to-data>, <ds-dirname>);
  hdr = ft_read_header(cfg.dataset);
  
  % quick and dirty 5 second chunks
  N = hdr.nSamples*hdr.nTrials;
  trl = [1:3000:(N-5999);6000:3000:N]';
  trl(:,3) = 0;
  
  % let's go for about 10 minutes of data, or less
  cfg.trl = trl(1:min(size(trl,1),12*10),:);
  cfg.channel = {'MEG'};
  cfg.demean  = 'yes';
  cfg.continuous = 'yes';
  data = ft_preprocessing(cfg);
  
  % this may be needed, because there may be some chunks of bad data in this recording, that occlude the artifact
  % this may be removed for your own purposes
  S = [];
  for kk = 1:numel(data.trial)
    S(:,kk) = std(data.trial{kk},[],2);
  end
  S = S./std(S,[],2);
  
  cfg = [];
  cfg.trials = find(mean(S)<2);
  data = ft_selectdata(cfg, data);
    
  cfg = [];
  cfg.method  = 'mtmfft';
  cfg.output  = 'pow';
  cfg.foilim  = [0 200];
  cfg.taper   = 'hanning';
  cfg.channel = 'MEG';
  freq      = ft_freqanalysis(cfg, data);
  
  figure;
  plot(freq.freq,mean(log10(freq.powspctrm))); title('powerspectrum','interpreter','none'); ylim([-30 -27]);

What can be done about it, and do I need to?
============================================

If your data is affected, it depends a bit on the data analysis requirements whether or not you need to do anything about this artifact. Specifically:

1. If you intend to perform your crucial analysis in source space, using      beamformers, you probably don't need to worry at all. The beamformer spatial filtering algorithm will be able to suppress signal sources that originate from outside the scanned dipole-locations-of-interest, so the artifact will probably be effectively suppressed. Note, that if you use a distributed source reconstruction technique, e.g. minimum norm estimation, then you are not off the hook, and you need to think a bit about proper noise regularisation.
2. If you intend to investigate your signals in the time domain (ERF), you might try a series of 'dftfilters' at 20 Hz and its higher harmonics (e.g. [20 40 60]). Since you will lowpassfilter your data most of the time anyhow it is probably not needed to go beyond 20 and 40 Hz for the dftfilter.

If your analysis is in the spectral domain, you may consider using the dftfilter as mentioned above. The dftfilter approach is probably insufficient if your analysis requires an accurate sensor level estimation of (e.g. beta band) spectral parameters, band-limited power, peak frequency, or phase estimates, for frequencies in the vicinity of the artifact frequencies.

.. note::
  One note with respect to dftfiltering: if the artifact's amplitude is not stationary over time, the dftfilter might not be capable of removing the
  artifact well. This is due to the fact that the spectral artifacts get some bandwidth due to the amplitude fluctuations. This could be alleviated with including some extra flanking frequencies in the dftfreq option for ft_preprocessing. These frequencies need to be specified in line with the intrinsic frequency resolution as dictated by the epoch lengths.
  
One way to reduce artifacts that originate from far away from the MEG sensors, is to use the signals picked up by the reference coils, that are located higher up in the MEG dewar. Subtracting a well-chosen linear combination of those reference signals, may remove a large part of the artifact. In CTF-speak, one of these techniques is called higher order gradient balancing, specifically 3d order gradient balancing. This 3d order gradient balancing is implemented in FieldTrip in the ft_denoise_synthetic function. It can be applied to the data as follows:

.. code-block:: matlab

  cfg = [];
  cfg.dataset = fullfile(<path-to-data>, <ds-dirname>);
  hdr = ft_read_header(cfg.dataset);
  
  % quick and dirty 5 second chunks
  N = hdr.nSamples*hdr.nTrials;
  trl = [1:3000:(N-5999);6000:3000:N]';
  trl(:,3) = 0;
  
  % let's go for about 10 minutes of data, or less
  cfg.trl = trl(1:min(size(trl,1),12*10),:);
  cfg.channel = {'MEG' 'MEGREF'}; % note: also read in the MEGREF
  cfg.demean  = 'yes';
  cfg.continuous = 'yes';
  data = ft_preprocessing(cfg);

  % this is needed, because there's some chunk of bad data in this recording
  % this may be removed for your own purposes
  S = [];
  for kk = 1:numel(data.trial)
    S(:,kk) = std(data.trial{kk},[],2);
  end
  S = S./std(S,[],2);
  
  cfg = [];
  cfg.trials = find(mean(S)<2);
  data = ft_selectdata(cfg, data);

  cfg = [];
  cfg.gradient = 'G3BR';
  data_G3BR = ft_denoise_synthetic(cfg, data);

  cfg = [];
  cfg.method  = 'mtmfft';
  cfg.output  = 'pow';
  cfg.foilim  = [0 200];
  cfg.taper   = 'hanning';
  cfg.channel = 'MEG';
  
  freq      = ft_freqanalysis(cfg, data);
  freq_G3BR = ft_freqanalysis(cfg, data_G3BR);
  
  figure;
  subplot(2,2,1); plot(freq.freq,mean(log10(freq.powspctrm))); title('original','interpreter','none'); ylim([-30 -27]);
  subplot(2,2,2); plot(freq_G3BR.freq,mean(log10(freq_G3BR.powspctrm))); title('G3BR','interpreter','none'); ylim([-30 -27]);
